---
title: Flutter渲染原理
---

# setState()分析

## 时序图

![setState](http://ucoon.tech/MyBlogImg/setState.jpg)

函数分析

1. State#setState(VoidCallback fn)

   ```dart
   @protected
   void setState(VoidCallback fn) {
       final Object? result = fn() as dynamic;
       _element!.markNeedsBuild();
     }
   ```

2. StatefulElement#markNeedsBuild()

   ```dart
   void markNeedsBuild() {
       if (dirty)
         return;
       _dirty = true;//标记为脏
       owner!.scheduleBuildFor(this);
   }
   ```

3. BuildOwner#scheduleBuildFor(Element element)

   ```dart
   void scheduleBuildFor(Element element) {
       if (!_scheduledFlushDirtyElements && onBuildScheduled != null) {
         _scheduledFlushDirtyElements = true;
         onBuildScheduled!();
       }
       ///添加到_dirtyElements集合中
       _dirtyElements.add(element);
       element._inDirtyList = true;
   }
   ```

**总结**：

1. setState过程其实只是将当前对应的Element标记为脏，并且添加到_dirtyElements集合中
2. Element持有Widget，存放上下文信息，RenderObjectElement额外持有RenderObject，通过它遍历视图树，支撑UI结构

可以看出`setState`没做任何渲染相关的事，那么页面是如何重新绘制？关键点在于Flutter的渲染机制

## Flutter渲染机制

