+++
title = "Compiler Note（从Xcode的角度）"
date = 2022-11-16
[taxonomies]
  tags = ["note", "compiler"]

+++

- 编译器做什么？
    a. 将高级源码（eg objective-c）转换成 低级源码
    b.代码错误检查

- Xcode使用clang作为编译器
    - Clang？
        - 分析&转换 Objective-C源码 到一个低阶表达（类似于 汇编码）——‘LLVM中间表示’（LLVM intermediate representation） 的工具
            - LLVM IR？
                - 低阶的、独立于操作系统的（所以可以跨平台）
                    - 跨平台eg 一套ios app的代码可以运行在两个不同架构上：intel、ARM，因为 LLVM可以将IR代码翻译成某平台的原生字节码
                - 可以接受指令，然后将其转换成目标平台的原生字节码；这个过程既可以实时地完成，也可以在编译期间完成
                - 具有三级架构：
                    - 一级：编程语言们 eg C、Objective-C、C++、Haskell
                    - 二级：可共享的优化器（优化LLVM IR的）
                    - 三级：目标平台 eg intel、ARM、PowerPC
                - 参考：[开源应用的架构 - LLVM](http://www.aosabook.org/en/llvm.html)

- 看看clang在编译一个hello.m的时候会做什么：
```txt
Makefile
% clang -ccc-print-phases hello.m

0: input, "hello.m", objective-c
1: preprocessor, {0}, objective-c-cpp-output
2: compiler, {1}, assembler
3: assembler, {2}, object
4: linker, {3}, image
5: bind-arch, "x86_64", {4}, image

```
<br/>


    a.预处理
        i. 处理宏（宏扩展）：将源码中的宏表达替换为其定义值
        ii. 有时做一些内联的工作
            - 但有时会出错：考虑用 static inline函数来替代一些宏表达
    b.token化
        i. 什么是token？
            - Eg
            ```objective-c
            Objective-C
            // token化前：（宏扩展前的源码）
            int main() {
            NSLog(@"hello, %@", @"world");
            return 0;
            }


            Groovy
            // token化后（location对应的是宏扩展前的位置）
            int 'int'        [StartOfLine]  Loc=<hello.m:4:1>
            identifier 'main'        [LeadingSpace] Loc=<hello.m:4:5>
            l_paren '('             Loc=<hello.m:4:9>
            r_paren ')'             Loc=<hello.m:4:10>
            l_brace '{'      [LeadingSpace] Loc=<hello.m:4:12>
            identifier 'NSLog'       [StartOfLine] [LeadingSpace]   Loc=<hello.m:5:3>
            l_paren '('             Loc=<hello.m:5:8>
            at '@'          Loc=<hello.m:5:9>
            string_literal '"hello, %@"'            Loc=<hello.m:5:10>
            comma ','               Loc=<hello.m:5:21>
            at '@'   [LeadingSpace] Loc=<hello.m:5:23>
            string_literal '"world"'                Loc=<hello.m:5:24>
            r_paren ')'             Loc=<hello.m:5:31>
            semi ';'                Loc=<hello.m:5:32>
            return 'return'  [StartOfLine] [LeadingSpace]   Loc=<hello.m:6:3>
            numeric_constant '0'     [LeadingSpace] Loc=<hello.m:6:10>
            semi ';'                Loc=<hello.m:6:11>
            r_brace '}'      [StartOfLine]  Loc=<hello.m:7:1>
            eof ''          Loc=<hello.m:7:2>


            ```

    c.解析
        i.token将被解析成为一棵抽象的语法树（AbstractSyntaxTree）
            - Eg
            ```objective-c
            Objective-C
            // 高阶语言的源码
            #import <Foundation/Foundation.h>
            @interface World
            - (void)hello;@end
            @implementation World
            - (void)hello {
            NSLog(@"hello, world");
            }@end
            int main() {
            World* world = [World new];
            [world hello];
            }


            Groovy
            // 打印出来的语法树
            @interface World- (void) hello;
            @end
            @implementation World
            - (void) hello (CompoundStmt 0x10372ded0 <hello.m:8:15, line:10:1>
            (CallExpr 0x10372dea0 <line:9:3, col:24> 'void'
                (ImplicitCastExpr 0x10372de88 <col:3> 'void (*)(NSString *, ...)' <FunctionToPointerDecay>
                (DeclRefExpr 0x10372ddd8 <col:3> 'void (NSString *, ...)' Function 0x1023510d0 'NSLog' 'void (NSString *, ...)'))
                (ObjCStringLiteral 0x10372de38 <col:9, col:10> 'NSString *'
                (StringLiteral 0x10372de00 <col:10> 'char [13]' lvalue "hello, world"))))


            @end
            int main() (CompoundStmt 0x10372e118 <hello.m:13:12, line:16:1>
            (DeclStmt 0x10372e090 <line:14:4, col:30>
                0x10372dfe0 "World *world =
                (ImplicitCastExpr 0x10372e078 <col:19, col:29> 'World *' <BitCast>
                    (ObjCMessageExpr 0x10372e048 <col:19, col:29> 'id':'id' selector=new class='World'))")
            (ObjCMessageExpr 0x10372e0e8 <line:15:4, col:16> 'void' selector=hello
                (ImplicitCastExpr 0x10372e0d0 <col:5> 'World *' <LValueToRValue>
                (DeclRefExpr 0x10372e0a8 <col:5> 'World *' lvalue Var 0x10372dfe0 'world' 'World *'))))


            ```
            - clang根据AST中的<line, col>来给出代码错误的位置（如果有错的话）
    d. 静态检查
        - 有了 抽象语法树AST 之后，可以对TA进行错误检查，eg 类型检查
        i. 类型检查
            - ' Check if the program send the correct messages to the correct objects & calls the correct functions on the correct values.'
            - 动态类型、静态类型？前者在运行时检查，后者在编译时检查
        ii. 一些其他检查
            - 对于clang，lib/StaticAnalyzer/Checkers/ 下有TA使用的checker.cpp。eg ObjCUnuserdIVarsChecker.cpp, ObjCSelfInitChecker.cpp, etc (you can tell the usage by their names)
    e. 代码生成：前面都正确执行完之后，可以生成LLVM代码了（存放在 xxx.ll 文件里）
        i.代码优化：给clang命令加上-O3参数
        ii.To compare with clang：可以参考[gcc做了哪些代码优化](https://ridiculousfish.com/blog/posts/will-it-optimize.html)

- 关心编译器干嘛？
    改造开源编译器、做自己的编译器插件、自己写分析器