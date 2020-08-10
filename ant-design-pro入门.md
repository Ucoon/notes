---
title: ant design pro入门
---

# ant design pro 简介

```ant design pro```是基于```ant design```这个框架搭建的中后台管理控制台的脚手架，而```ant design```是蚂蚁金服基于```react```打造的一个服务于企业级产品的UI框架。

```ant design pro```官方文档：[传送门](https://pro.ant.design/docs/getting-started-cn)

# Get Started

**1. 根据文档安装**

**2.目录结构**

```react
├── config                   # umi 配置，包含路由，构建等配置
├── mock                     # 本地模拟数据
├── public
│   └── favicon.png          # Favicon
├── src
│   ├── assets               # 本地静态资源
│   ├── components           # 业务通用组件
│   ├── e2e                  # 集成测试用例
│   ├── layouts              # 通用布局
│   ├── models               # 全局 dva model
│   ├── pages                # 业务页面入口和常用模板
│   ├── services             # 后台接口服务
│   ├── utils                # 工具库
│   ├── locales              # 国际化资源
│   ├── global.less          # 全局样式
│   └── global.ts            # 全局 JS
├── tests                    # 测试工具
├── README.md
└── package.json
```

**3. 运行脚手架**

**4. 如何新建一个页面**

1. 在```src/pages```目录下新建模块与页面
2. 在```config```目录下```config.ts```文件中配置菜单路由

**5.与服务端进行交互**

1. UI组件交互操作
2. 调用model的effect
3. 调用service的请求函数
4. 使用封装的request.js发送请求
5. 获取服务端返回
6. 调用reducer改变state
7. 更新model