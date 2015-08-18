---
layout: post
title: "autorelease方法的源码分析"
date: 2015-08-18 10:51:06 +0800
comments: true
published: true
sharing: true
footer: true
categories: ObjC
---

```
    static inline id autorelease(id obj)
    {
        assert(obj);
        assert(!obj->isTaggedPointer());
        id *dest __unused = autoreleaseFast(obj);
        assert(!dest  ||  *dest == obj);
        return obj;
    }
```

如果指针没有被标记，则进行 autoreleaseFast 处理过程

```
    static inline id *autoreleaseFast(id obj)
    {
        AutoreleasePoolPage *page = hotPage();
        if (page && !page->full()) {
            return page->add(obj);
        } else if (page) {
            return autoreleaseFullPage(obj, page);
        } else {
            return autoreleaseNoPage(obj);
        }
    }
```

```
    static __attribute__((noinline))
    id *autoreleaseFullPage(id obj, AutoreleasePoolPage *page)
    {
        // The hot page is full. 
        // Step to the next non-full page, adding a new page if necessary.
        // Then add the object to that page.
        assert(page == hotPage()  &&  page->full());

        do {
            if (page->child) page = page->child;
            else page = new AutoreleasePoolPage(page);
        } while (page->full());

        setHotPage(page);
        return page->add(obj);
    }
```

```
    static __attribute__((noinline))
    id *autoreleaseNoPage(id obj)
    {
        // No pool in place.
        assert(!hotPage());

        if (obj != POOL_SENTINEL  &&  DebugMissingPools) {
            // We are pushing an object with no pool in place, 
            // and no-pool debugging was requested by environment.
            _objc_inform("MISSING POOLS: Object %p of class %s "
                         "autoreleased with no pool in place - "
                         "just leaking - break on "
                         "objc_autoreleaseNoPool() to debug", 
                         (void*)obj, object_getClassName(obj));
            objc_autoreleaseNoPool(obj);
            return nil;
        }

        // Install the first page.
        AutoreleasePoolPage *page = new AutoreleasePoolPage(nil);
        setHotPage(page);

        // Push an autorelease pool boundary if it wasn't already requested.
        if (obj != POOL_SENTINEL) {
            page->add(POOL_SENTINEL);
        }

        // Push the requested object.
        return page->add(obj);
    }
```

每个 page  的大小是 4096 ，指的是能存储 4096 个对象地址。

如果 `page->full()` 已满，则(通过 `autoreleaseFullPage(obj, page);`)自动从子节点中寻找一个未满的 page 添加新的对象，如果所有子节点都满了，就会创建一个新的page 并让 hotPage 指向这个新的 page，最后添加进新的对象。 

如果没有任何的 page，则创建第一个 page，并添加一个 POOL_SENTINEL 作为边界，然后添加这个新的 obj 对象。


