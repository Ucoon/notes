

# 创建game loop（游戏循环）

game loop是一款游戏的本质，即为一组反复运行的代码。

有一个很常见的叫法：FPS，它代表每秒的帧数，这意味着，若你的游戏是60fps，那么game loop将在每秒循环60次。

一个基本的game loop由两部分组成，`update`和`render`

![game loop](http://ucoon.gitee.io/myblogimg/game_loop.png)

绝大多数的游戏基于这两种函数构建：

- render（渲染）：用于绘制游戏的当前状态
- update（更新）：接收自上次update以来的时间增量（以秒为单位），并允许切换至下个状态

# 结构Structure

只需创建一个`assets`，包含两个子目录：`audio`和`images`

## 目录结构

文件的结构应该为：

```dart
└── assets
    ├── audio
    │   └── explosion.mp3
    └── images
        ├── enemy.png
        └── player.png
```

## 在项目中引入

在文件`pubspec.yaml`中引入你的资源文件：

```dart
flutter:
  assets:
    - assets/audio/explosion.mp3
    - assets/images/player.png
    - assets/images/enemy.png
```

参考文档：

1. Getting Started：https://flame-engine.org/docs/#/
2. ReadMe 1：https://github.com/flame-engine/flame/blob/main/tutorials/1_basic_square/README.md
3. ReadMe 2：https://github.com/flame-engine/flame/blob/main/tutorials/2_sprite_animations_gestures/README.md