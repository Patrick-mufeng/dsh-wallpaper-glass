# dsh-wallpaper-glass

DSH（DeepSeek Harness）壁纸引擎插件的**精简分支** —— 只保留日常实际在用的部分：壁纸 + 液态玻璃。

基于 [elysia395/dsh-wallpaper-engine](https://github.com/elysia395/dsh-wallpaper-engine) **v1.4.2**（MIT License）裁剪而来。

> 包名已与上游完全脱离（`dsh-wallpaper-glass` 系列），可以与本机上的上游原版共存而不冲突。

---

## 这个仓库是什么

上游 1.4.2 是一个 **9 包的插件族**（28.6 MB、47 条 HTTP 路由、147 个设置键、6 个设置页签），把壁纸、液态玻璃、字体、吉祥物、头像、点击特效、3D 视差、桌宠联动、主题预设、关于页全部打包在一起。

本仓库按「**只保留在用的部分**」裁到 **4 个包、7 条路由面、4 个设置页签**，体积从 28.6 MB 降到 **4.4 MB**。

保留的：

| 能力 | 说明 |
|---|---|
| **壁纸** | Scene / Web / Video / 图片，扫描本机 Wallpaper Engine，支持自定义上传 |
| **液态玻璃** | 设置窗、对话区、侧边栏、浮层；六面可独立配置 |
| **字体** | 字体集、角色级字号/字重/字族、本机字体扫描 |
| **壁纸库** | 筛选、隐藏/恢复、自动轮播、切换过场 |

裁掉的（详见下方「移除清单」）：聊天吉祥物、会话头像、点击拖尾特效、3D 视差、桌宠联动、「扩展」页签、「关于」页签。

---

## 目录结构

```
packages/
├── dsh-wallpaper-glass/                主包：壁纸扫描/服务 + 设置面板 + 样式层
├── dsh-wallpaper-glass-media-bridge/   壁纸字节与库存路由（27 条，壁纸本体）
├── dsh-wallpaper-glass-ui/             液态玻璃渲染器与 CSS
└── dsh-wallpaper-glass-fonts/          字体集与系统字体扫描
```

| 目录 | npm 包名 |
|---|---|
| `packages/dsh-wallpaper-glass` | `dsh-wallpaper-glass` |
| `packages/dsh-wallpaper-glass-media-bridge` | `dsh-wallpaper-glass-media-bridge` |
| `packages/dsh-wallpaper-glass-ui` | `dsh-wallpaper-glass-ui` |
| `packages/dsh-wallpaper-glass-fonts` | `dsh-wallpaper-glass-fonts` |

> 注意玻璃子包的 npm 名是 `-ui` 而不是 `-glass`：主包已占用 `dsh-wallpaper-glass`，再用 `dsh-wallpaper-glass-glass` 既冗长又易混。

上游的 `about-notice` / `mascot-avatar` / `fx` / `parallax` / `pet-link` 五个包**不在本仓库**。

---

## 安装

本仓库是**改动后的产物包**（不是上游的 `src/` 源码树），安装方式与上游一致：

```sh
# 桌面端
dsh plugin --profile desktop add <本仓库的本地路径或打包后的 tgz>

# Web 端
dsh plugin --profile web add <同上>
```

前置：**Node.js >= 22.12.0**、**DSH 0.2 内核**、**pnpm**。安装前完全退出 DSH。

> ⚠️ 主包 `package.json` 的 `dependencies` 指向同版本的 3 个子包（`1.4.2`）。
> 若从本地安装，请确保四个包目录在同一层级，或先 `pnpm link` 好再装。

---

## 移除清单（与上游 1.4.2 的差异）

### 一、整包移除（5 个 npm 包 + 5 条 bundle 行）

| 移除项 | 原因 | 省下 |
|---|---|---|
| `-mascot-avatar` | 聊天吉祥物（拉绳挂件、立绘、拖拽）与会话头像 | 25 KB + 2 条路由 |
| `-fx` | 点击涟漪 / 拖尾特效 | 50 KB |
| `-parallax` | 3D 视差层 | 102 KB |
| `-pet-link` | 桌宠联动（依赖外部 `dshpet-dshwe` 运行时） | **20.3 MB 素材** + 5 条路由 |
| `-about-notice` | 「关于」页签的 Star 数与群二维码 | 23 KB + 2 条路由 |

### 二、设置页签（6 → 4）

移除「扩展」「关于」两个页签：

- **「扩展」**原有四张卡：自定义会话头像、点击拖尾、3D 效果、桌宠联动 —— 四个来源包全部移除，页签已无内容可画。
- **「关于」**是纯资讯页（简介 / Star 数 / 群二维码 / 致谢），不含任何设置；其数据源 `about-notice` 已移除，同时**去掉了对 GitHub 外网 API 的请求**。

现在：**壁纸库 / 外观 / 播放 / 系统**。

### 三、素材剔除（3.03 MB）

| 目录 | 内容 | 为何剔除 |
|---|---|---|
| `assets/rope/` | 10 张吉祥物立绘 | 吉祥物已移除，无消费者 |
| `assets/readme/` | 41 张 README 插图 | 非运行必需 |
| `lib/about/` + `assets/about/` | QQ/抖音群二维码、更新公告图 | 「关于」页已移除 |

> 连带把 `lib/index.js` 里的 `registerRopeArtRoutes` 注册点与 import 一并下线（路由服务这些立绘，图已不存在）。

### 四、代码内的适配改动

| 文件 | 改动 |
|---|---|
| `lib/index.js` | 移除 `about-notice` / `mascot-avatar` 的静态 import（包已不存在，否则模块求值期 `ERR_MODULE_NOT_FOUND`）；`componentImpls()` 分发表只留 `media-bridge`；移除 `registerRopeArtRoutes` |
| `lib/client.js` | `PICKER_TABS` 6 → 4 项；`PICKER_TAB_LEGACY` 把 `extensions` / `about` 迁移到 `system`；删除 `renderExtensionsTab()` 与 `renderAboutTab()`；`renderMascotTab()` 返回 `null`；`RopeDock` 不再挂载（portal 改为只渲染 `UpdateNotice`）；取消 avatar 扩展模块注册 |
| `cordis.patch.yml` | 9 行 → 4 行 |
| `package.json` | `dependencies` 8 → 3 个子包；`files` 白名单剔除已删素材 |

### 五、顺带的布局修复

上游 `client.js` 里有几个类**完全没有 CSS 规则**，导致设置页排版退化。本版补齐：

- `.we-picker`（根容器）：原先零规则 → flex 纵向流 + 14px 统一节奏
- `.we-picker__card-shell`（外层卡片）：原先零规则 → 内边距 + 圆角 + 底色 + 描边
- `.we-picker__card-head` / `__card-name` / `__card-desc`：原先零规则 → 标题行排版 + 数量徽标胶囊
- `.we-picker__rate` 的 `padding: 0` 导致按钮文字贴边（它与 `.we-picker__btn` 的 12px 冲突且排在后面）→ 改为 `0 10px` + `min-width: 76px`
- 4 枚页签的尺寸放开一档（28→32px 高、12→13px 字），用 `.we-picker` 前缀限定以免影响右侧栏

完整过程见 [`docs/`](docs/)。

---

## 恢复某个功能

以恢复「桌宠联动」为例：

1. `dsh plugin add dsh-plugin-wallpaper-engine-pet-link@1.4.2`（从上游装，包名保持上游原样）
2. `packages/dsh-wallpaper-glass/package.json` 的 `dependencies` 加回该包
3. `packages/dsh-wallpaper-glass/cordis.patch.yml` 加回那一行：
   ```yaml
   - id: wallpaper-engine-pet-link
     name: 'dsh-plugin-wallpaper-engine-pet-link'
   ```
4. 客户端不改（页签已删，但桌宠联动原本是挂在吉祥物上的 —— 需先恢复吉祥物）

恢复「吉祥物」则要额外：还原 `renderMascotTab()` 实现、把 portal 的 `root.render` 换回 `React.createElement(RopeDock, null)`、恢复 `assets/rope/` 与 `registerRopeArtRoutes`。

> ⚠️ **行与依赖必须同时加/删**。主包 `apply()` 的 `assertSubpackagesResolvable` 会按 patch 里的净新增行逐个 `resolve`，解析不到就**大声抛错**（宿主对行级失败是静默的，上游故意做成硬失败）。

---

## 已知限制

- **本仓库是构建产物**，不是源码树。`lib/client.js` 是 65 个前端模块内联成的单文件（约 1.9 MB / 28,900 行），无法按模块阅读或修改。要改前端逻辑请回到上游源码。
- **无法合并上游更新**。上游发新版本时，需要手动重新裁剪。
- 上游的 `test/` 与 `docs/` 未随 npm 包发布，本仓库没有守卫测试。

---

## 许可与来源

**代码**以 [MIT License](LICENSE) 发布，原始版权归 **elysia395** 所有（本分支的全部改动同样以 MIT 发布）。

**素材**的授权可能不同 —— 上游 `SOURCES.md` 明确说明「壁纸、角色与字体素材的授权可能不同，再分发或商用前请核对素材说明」。本仓库已剔除角色立绘、群二维码等素材，仅保留运行必需的图标与出厂预设。

上游项目：https://github.com/elysia395/dsh-wallpaper-engine

---

## 免责

这是**个人使用的裁剪版**，非上游官方发行版。上游不接受本分支的 issue；上游相关的问题请到[上游仓库](https://github.com/elysia395/dsh-wallpaper-engine/issues)反馈。
