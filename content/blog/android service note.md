+++
title = "Android Service Note"
date = 2022-11-16
[taxonomies]
  tags = ["note", "components", "Android"]

+++

- service是一种可以 在后台长时间运行 & 不提供任何界面 的应用组件

- service可以在后台做：处理网络事务、播放音乐🎵、执行文件IO、与ContentProvider进行交互

- Service 分类：

    a.前台（要显示在通知里）：eg 放歌

    b.后台（真的没ui）：eg 压缩存储空间的任务

        - note：after api26，后台服务执行限制🔞，考虑计划作业？
    
    c.In 绑定状态，or not

        - bindService()

        - 组件对service的绑定是N:1的关系

            - 也可以把服务声明成私有的，这样就不给别的app用了

        - 给组件的重要的回调：onStartCommand()来启动服务，onBind()来绑定服务

            - 不论服务在启动/绑定状态下，组件都可以用intent来使用服务

- Service vs 线程

    a. service还是在主线程运行，所以如果是 密集型/组织性操作，应该在服务里新建线程

- 后台任务（跟 后台服务 是一回事吗？YES call it 计划作业 - -）

    a. 分类：即时任务 immediate、延期任务 Deferred、精确任务 Exact
        
        - 考虑：任务是否要在用户&应用进行互动的时候完成？（YES -> 即时）任务是否要在精确的时间点运行？(YES -> 精确）

    b. 即时任务 -》kotlin协程了解一下

        - If 即时&长时间 -〉WorkManager

    c. 延期任务 -》WorkManager了解一下

    d. 精确任务 -》AlarmManager

- About background service & broadcast usage：[androidDeveloper-后台任务指北](https://developer.android.com/guide/background)
