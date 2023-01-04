---
title: Flutter性能分析
---

# 概念

## 如何了解页面流畅度

FPS(Frames Per Second)，在一定程度上反映了页面流畅度。当每秒的画面达到60帧，整个过程是流畅的。

- 流畅：FPS大于55，即一帧耗时低于18ms
- 良好：FPS在30-55之间，即一帧耗时在18-33ms之间
- 轻微卡顿：FPS在15-30之间，即一帧耗时在33-67ms之间
- 卡顿：FPS低于15，即一帧耗时大于66.7ms

# 工具

- [PerfDog](https://perfdog.qq.com/)

