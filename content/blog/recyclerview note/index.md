+++
title = "RecyclerView Note"
date = 2024-07-24
[taxonomies]
  tags = ["note", "Android", "view"]

[extra]
  toc = true
+++


- *References*
    
    [Android源码分析之RecyclerView源码分析（一）—— 绘制流程](https://juejin.cn/post/6844904104637022221)
    
    [Android源码分析之RecyclerView源码分析（二）—— 缓存机制](https://juejin.cn/post/6844904104641167368)
    
- When to get the View and try to get it from cache? → `Recycler.getViewForPosition()`
    
    [recycler view note.pdf](RecyclerView%20note%20d996114cee2b466c963a6ed104577a5e/recycler_view_note.pdf)

    ![recycler view note.pdf](./RecyclerView%20note%20d996114cee2b466c963a6ed104577a5e/recycler_view_note_img.png)
    
- How to try getting View from cache?
    
    There are some simple caches in eg `LinearLayoutManager.mScrapList`. 
    
    The core cache of a RecyclerView is Recycler. See `Recycler.getViewForPosition`. Here is the priority in it ( `Recycler.tryGetViewHolderForPositionByDeadline` ) :
    
    ```java
    // 0, by position, then id
    // from scrap - mChangedScrap
    getChangedScrapViewForPosition();
    // 1, by position
    // from scrap - mAttachedScrap
    // then hidden - mHiddenViews
    // then cache - mCachedViews
    getScrapOrHiddenOrCachedHolderForPosition();
    // 2, by id 
    // from scrap - mAttachedScrap
    // then cache - mCachedViews
    getScrapOrCachedViewForId();
    // 3
    (ViewCacheExtension) mViewCacheExtension.getViewForPositionAndType();
    // 4
    getRecycledViewPool().getRecycledView(type);
    // 5, create one
    mAdapter.createViewHolder();
    ```
    
    After getting the ViewHolder instance, check for binding and use `tryBindViewHolderByDeadline`  to bind. Then it is ok to return the ViewHolder.