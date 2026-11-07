# dsh-wallpaper-glass

DSH（DeepSeek Harness）壁纸引擎 —— 只保留壁纸与液态玻璃的精简分支。

基于 [elysia395/dsh-wallpaper-engine](https://github.com/elysia395/dsh-wallpaper-engine) **v1.4.2**（MIT License）裁剪而来。

> 完整说明、移除清单与本版改动记录见**仓库根目录的 [README.md](../../README.md)**。
> 本文件只说明包内约定。

---

## 本包是什么

主包（宿主半边 + 浏览器半边合体）：

- **宿主半边** `lib/index.js` —— 扫描本机 Wallpaper Engine（Steam app 431960），
  枚举 Scene / Video / Web / 图片壁纸，经同源 HTTP 路由提供库存与媒体字节
- **浏览器半边** `lib/client.js` —— 设置面板、壁纸层、液态玻璃行为层、样式表

壁纸字节与库存路由由同系列的 `dsh-wallpaper-glass-media-bridge` 承载。

## 本发行版与上游的差异

已整块移除（对应 npm 包不再是依赖）：

| 移除项 | 说明 |
|---|---|
| 聊天吉祥物 / 会话头像 | 含拉绳挂件与立绘 |
| 点击涟漪 / 拖尾特效 | — |
| 3D 视差 | — |
| 桌宠联动 | — |
| 「关于」页签 | 纯资讯页，无设置项 |
| 「扩展」页签 | 四个来源包全部移除后已无内容 |

设置页现为四页签：**壁纸库 / 外观 / 播放 / 系统**。

同时补了上游缺失的若干 CSS（设置页骨架、间距层级、按钮内边距、玻璃兼容区块字号），
详见仓库根 README。

## 目录

```
lib/
├── index.js            宿主半边入口（Cordis plugin）
├── client.js           浏览器半边产物（构建后单文件）
├── host-boot.js        宿主引导与组件装载
├── kernel/             宿主内核（发现 / 配置 / 数据目录 / 上传库 …）
├── routes/             主包自有的少数路由族
├── media/              媒体探测与转码内核
├── webwallgl/          Scene 实时渲染引擎
├── styles-*.js         样式域（生产物内联进 client.js）
├── glass-presets/      出厂玻璃预设
└── types/              类型声明
```

## 关于本产物

本包是 **构建产物**，不是源码树。`lib/client.js` 由约 65 个前端模块内联成单文件
（约 1.9 MB），无法按模块阅读或修改。要改前端逻辑请回到上游源码仓库。

## 许可

代码以 [MIT License](../../LICENSE) 发布，原始版权归 **elysia395** 所有。
素材授权可能不同，再分发或商用前请核对上游素材说明。

上游项目：https://github.com/elysia395/dsh-wallpaper-engine
