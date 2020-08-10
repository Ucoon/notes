---
title: LiveData Tips
---

**1.概览**

`LiveData`是一种可观察的数据存储器类，与常规的可观察类不同，`LiveData` 具有生命周期感知能力，意指它遵循其他应用组件（如 `Activity`、`Fragment`或 `Service`）的生命周期。这种感知能力可确保 `LiveData `**仅更新处于活跃生命周期状态**的应用组件观察者。

什么是观察者的活跃状态：

观察者（由 [`Observer`](https://developer.android.com/reference/androidx/lifecycle/Observer) 类表示）的生命周期处于 `STARTED` 或 `RESUMED` 状态

```java
Lifecycle.State STARTED:
	after onStart call;
	right before onPause call;
Lifecycle.State RESUMED: after onResume call;
```

LiveData 只会将更新通知给活跃的观察者。为观察 [`LiveData`](https://developer.android.com/reference/androidx/lifecycle/LiveData) 对象而注册的非活跃观察者不会收到更改通知。

**2.优势**

1. 确保界面符合数据状态
2. 不会发生内存泄漏
3. 不会因Activity停止导致崩溃
4. 不再需要手动处理生命周期
5. 数据始终保持最新状态
6. 适当的配置更改
7. 共享资源