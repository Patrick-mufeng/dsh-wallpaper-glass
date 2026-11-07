# 完整裁撤清单（可执行版）

配套文档：`01-插件分析与精简清单.md`
状态：**仅分析，未改动任何文件**
依据：你的三条决定 —— ①先看清单再决定 ②吉祥物彻底裁掉（含 mascot-avatar 包）③玻璃六面独立配置完全不动

---

## ⚠️ 零、先看这一节：三个必须先知道的陷阱

我上一轮给你的「方案②」里有一处**判断错误**，这里更正，并且新发现两个会让界面直接坏掉的坑。**在你拍板前必须知道这些。**

### 陷阱 1（我上轮说错了）：media-bridge 不能停，它才是壁纸本体

我上轮写「裁掉媒体桥整包」——**这是错的，会直接导致所有壁纸失效**。

真相：`/inventory`、`/media`、`/preview`、`/upload`、`/remove`、`/props`、`/slides`、`/scene-*` 全部注册在 **media-bridge** 里（`lib/routes/media-bytes.js`、`inventory.js`、`upload.js` …）。

实测证据 —— 当前运行中的服务正好印证：
```
GET /wallpaper-engine/inventory → 200
components: {"about-notice":"started","mascot-avatar":"started","media-bridge":"started"}
wallpapers: video/【哲风壁纸】卡通-水面-治愈 , image/【哲风壁纸】会议室-卡通插画
```

主包自己的 `lib/index.js` 只保留了 8 条路由（`/settings` `/glass-presets` `/diag` `/diag-log` `/client-diag` `/cache-dir` `/cache-dir/browse` `/rope-art`），**一条壁纸字节都不发**。

> **结论：media-bridge 是必需行。** 想裁它内部的重链路（转码/抽帧/Now Playing/遮挡），只能删 `media-bridge/lib/index.js` 里对应的 `registerXxxRoutes(...)` 调用行——这是**真正的代码手术**，不是改 YAML 能办到的。我上轮把它归到"停一行就完事"是错误的，特此更正。

### 陷阱 2（新发现）：停用 mascot-avatar 会让「扩展」页签**整页空白**

`client.js:13696` 的 `renderExtensionsTab()` 第一行就是：
```js
if (!weComponentVisibleOf(ctx.sel?.inventory?.components, 'mascot-avatar')) return null;
```

而页签导航表 `TAB_SECTION_OWNER = { library: "library" }`（`client.js:1194xxx`）**只对 library 做了 gate**，「extensions」页签**永远在导航栏里**。

后果：mascot-avatar 一停 → 扩展页签内容返回 `null` → 用户点进去看到**一块完全空白的面板**，没有报错、没有空态提示。这正是插件自己在注释里专门防的那个故障形态（"点进去什么都没有"），但它这次没防住自己。

> **必须连带处理**：停 mascot-avatar 的同时，要把 `extensions` 从 `PICKER_TABS`（`client.js:24352`）里删掉那一行，否则留一个死页签。

### 陷阱 3（新发现）：RopeDock 是「更新公告」的唯一挂载点

`client.js:27851`：
```js
// Hidden: render nothing (rope + drawer), but keep the one-time update notice
if (hidden) {
  return React.createElement(React.Fragment, null, React.createElement(UpdateNotice, null));
}
```

你现在的 `ropeShown: false` 只是让吉祥物不画，**RopeDock 组件仍然挂在 `body` 上**（`ROPE_DOCK_ID`），它同时还负责渲染「有新版本」的更新公告弹窗。

> **如果直接删掉 RopeDock 组件，更新公告会一起消失。** 吉祥物要裁干净，必须把 `UpdateNotice` 另行挂到一个宿主上（或接受没有更新提示）。

---

## 一、你的三条决定 → 连带影响推演

| 你的决定 | 直接效果 | 必须的连带动作 | 风险 |
|---|---|---|---|
| ② 吉祥物彻底裁掉（含 mascot-avatar 包） | 「系统」页签吉祥物节消失（`renderMascotTab` → null，但 advanced 六节还在，页签不死）；拉绳挂件消失 | ⚠️ 必须同时删 `extensions` 页签（陷阱 2）<br>⚠️ 必须给 `UpdateNotice` 另找挂载点（陷阱 3）<br>⚠️ `assets/rope/` 1.74 MB + `/rope-art` 路由可一并删 | 中：牵动 3 处非预期位置 |
| ③ 玻璃六面独立配置完全不动 | 外观页签 36 控件原样保留 | 无 | 零 |

---

## 二、逐项裁撤清单（含精确位置 / 体积 / 风险 / 验证）

### A 档 · 只改 YAML，停整包行（0 行代码，随时可逆）

| # | 目标 | 精确改动 | 省下 | 连带影响 | 风险 |
|---|---|---|---|---|---|
| A1 | **pet-link** | 从 `desktop\cordis.patch.yml`（或 bundle 层）置 `disabled: true`；同时可把 `dsh-plugin-wallpaper-engine-pet-link` 从 `dependencies` 移出 | **263 KB JS + 20.3 MB 素材**（webm 动画 13.5 MB / 表情包 6.4 MB / 图文 0.34 MB）+ **5 条路由**（`/pet-link/official` ×4、`/pet-link/pet/state`） | 「扩展」第 4 张卡消失（你本来也没装 dsh-pet） | **低** |
| A2 | **fx** | 同上 | 50 KB JS + **10 个控件** + 1 条内置（无独立路由） | 「扩展」第 2 张卡消失；`fxEnabled` 已 false | **低** |
| A3 | **parallax** | 同上 | 102 KB JS + **11 个控件** | 「扩展」第 3 张卡消失；`parallaxEnabled` 已 false | **低** |
| A4 | **about-notice** | 同上 | 23 KB JS + **2 条路由**（`/star-count` 含**外网 GitHub API 调用**、`/about-qr`） | 「关于」页签少 Star 行 + 2 张二维码；简介/致谢/仓库链接仍在（`renderAboutTab` 整体被 gate → 注意：**会整页空白**，同陷阱 2 形态！） | **中** ⚠️ |
| A5 | **mascot-avatar** | 同上 | 25 KB JS + **2 条路由**（`/avatar` `/mascot`）+ `assets/rope/` **1.74 MB** | 见第四节完整推演 | **中** |

> ⚠️ **A4 也要注意**：`renderAboutTab` 同样被 gate（`client.js:13740`），停 about-notice → 「关于」页签整页空白，而它也在导航栏里（`TAB_SECTION_OWNER` 没管它）。**A4 的处置和 A5 一样，要连带删页签。**

**A 档合计**：停 A1+A2+A3+A4+A5 → 省 **463 KB JS + 22.1 MB 素材 + 9 条路由**。

---

### B 档 · 需改代码，裁功能但保主体（按性价比排序）

| # | 目标 | 精确位置 | 省下 | 风险 | 说明 |
|---|---|---|---|---|---|
| B1 | **Scene / Web 实时渲染链** | `webwallgl/`（277 KB，含 renderer.js 291 行）<br>`scene-manifest.js` 325 行 / `pkg-read.js` 422 行 / `media/scene-probe.js` 146 行<br>内联 5 个 `live-layer*.js` ≈ 2,900 行<br>路由 `/scene-live` `/scene-files` `/scene-frame` `/scene-frame-cache` `/scene-video` `/scene-audio` `/scene-payload-progress` `/live-frame` | **~1 MB JS + 8 条路由** | **低**（本机没装 WE，`installDir` 为空 —— 实测无感） | ⚠️ 8 条路由**在 media-bridge 里**，得删注册行不是停行 |
| B2 | **转码 / 抽帧** | `media/transcode.js` 616 行、`mp4-probe.js`、`mp4-vfs.js` 256 行、`faststart.js`、`video-preview.js`<br>路由 `/transcoded` `/transcode-progress` `/media-info` `/video-preview` | ~900 行 + 4 条路由 + **ffmpeg 依赖** | **低** | 剪掉后「播放」页签的倍速仍可用，只是帧率上限滑条失去意义（需一并隐藏） |
| B3 | **Now Playing / 系统音频 / 遮挡** | `media-bridge/lib/supervisor.js` 766 行、`backend.js`、`legacy.js` 470 行、`provision.js` 14 KB、`audio-tap.swift`<br>`occlusion/` 全目录 + `occlusion/x11.js` 334 行<br>路由 `/now-playing` `/now-playing/artwork` `/audio-spectrum` `/media-status` `/media-control` `/occlusion` | **~1,400 行 + 6 条路由 + 子进程 + x11 依赖** | **中**（`mediaIntegration` 是 `kind:'const'` 恒开，删了要同步改 schema） | 这是"系统音频频谱 / 封面 / 遮挡暂停"的整条链 |
| B4 | **主题预设整机快照** | `routes/presets.js` 967 行 + 内联 `preset-store.js` 527 行 + `lib/glass-presets/*` **239 KB**（含 156 KB 的 factory-author.json）+ `/glass-presets` 路由 | **~1,500 行 + 239 KB + 1 条路由** | **中** | 出厂 7 套预设一起没了；你目前也没用预设 |
| B5 | **字体集系统** | `fonts` 整包 108 KB（`fontsets.js` 416 行、`system-fonts.js` 536 行、`client.fonts.js` 657 行）+ 内联 `fontset-store.js` 560 行 + `/fontsets` `/system-fonts` | **~2,100 行 + 2 条路由 + PowerShell 扫描** | **中** | 你 `fontCustom:true` + 字体集 `preset-font` **在用**；砍了只能保留单一全局字体 |
| B6 | **诊断族** | `routes/diag.js`、`diag-trace.js` 118 行、`/diag` `/diag-log` `/client-diag` + 内联 `live-layer-diag.js` | ~250 行 + 3 条路由 | **低** | 出问题少一个排查入口 |
| B7 | **适配器矩阵** | 内联 `adapter.js` 146 行 + `media-origin.js` 139 行 + `/media-origin` | ~300 行 | **中** | 影响网页壁纸载荷来源判定，牵涉安全栅栏，**不建议动** |
| B8 | **缓存目录浏览器** | `routes/cache-dir.js` 140 行 + `/cache-dir/browse` + 内联面板 ~130 行 | ~270 行 + 1 条路由 | **低** | 保留 `/cache-dir` 手动填路径即可 |
| B9 | **"关于"页签静态内容** | `renderAboutTab` 全函数 + `about-assets.js` + `lib/about/*` **268 KB**（QQ/抖音二维码、公告图） | ~200 行 + 268 KB | **低** | 纯资讯页 |

---

### C 档 · 保留但可收纳（你说完全不动，列出仅供知情）

| # | 内容 | 规模 | 你的决定 |
|---|---|---|---|
| C1 | **玻璃六面独立配置** | 36 控件 / ~40 键 | ✅ **完全不动** |
| C2 | 侧边栏圆角 | 20 按钮 / 10 键 | 未定 |
| C3 | 全局 UI 配色 5 元素 | 8 键（你只用了 `accent`） | 未定 |
| C4 | 主题随壁纸（试验性） | 1 键 + `theme-follow.js` 483 行 | 未定 |
| C5 | 「播放」页签 + 侧栏快捷面板双档渲染 | 架构重复，改一处要改两处 | 未定 |

---

## 三、裁撤后的规模推演

| 方案 | 停 A 档 | 加做 B 档 | 剩余 JS | 剩余路由 | 剩余页签 | 剩余设置键 |
|---|---|---|---|---|---|---|
| **现状** | — | — | 3.8 MB / 62,968 行 | **47** | **6** | **147** |
| **A 档全停** | A1–A5 | — | 3.35 MB / 56,000 行 | 38 | 4（删 extensions + about） | 147（键仍在 schema） |
| **A + B1/B2** | A1–A5 | 实时渲染 + 转码 | ~2.4 MB / ~45,000 行 | 26 | 4 | ~140 |
| **A + B1–B4** | A1–A5 | + Now Playing/遮挡 + 预设 | ~1.7 MB / ~38,000 行 | 19 | 4 | ~140 |
| **A + B1–B9** | A1–A5 | 全部 | ~1.3 MB / ~32,000 行 | 14 | 3（壁纸/外观/播放） | ~135 |

> 注意：**设置键数量降不下来**，除非改 `settings-schema.js` 的 7 张表（KINDS / DEFAULTS / SETTINGS_NAMESPACES / RESET_MODULES / RESET_TABS / PRESET_ASSET_KEYS / PRESET_EXCLUDED_KEYS）+ 过守卫。这是文档 01 里说的那条技术红线。**建议键表先不动**——它不占界面、不耗性能，只是配置文件里多几行。

---

## 四、A5「吉祥物彻底裁掉」的完整执行推演

你说要连 mascot-avatar 包一起裁。这是唯一一个**必须动代码、且牵动 4 个非预期位置**的项，单独列清楚：

### 4.1 受影响的界面

| 位置 | 现状 | 裁后 | 处理 |
|---|---|---|---|
| 「系统」页签 · 聊天吉祥物节 | `renderMascotTab` 渲染 | 返回 `null`（gate 命中） | ✅ 自动，页签不死（advanced 六节还在） |
| 「扩展」页签 | `renderExtensionsTab` | 返回 `null` → **整页空白** | ⚠️ 必须删页签（陷阱 2） |
| 会话头像装饰层 | `syncAvatarLayer` | 整层不挂 | ✅ 自动（gate 已就位，`client.js:11508`） |
| 拉绳吉祥物 RopeDock | 挂在 `body`，`ropeShown:false` 时不画 | 组件应删除 | ⚠️ 但它是 `UpdateNotice` 的挂载点（陷阱 3） |
| 宿主路由 | `/avatar` `/mascot` | 2 条消失 | ✅ 自动（行停即撤） |
| 立绘素材 | `assets/rope/` 10 张 1.74 MB + `/rope-art` 路由 | 可删 | 手动 |

### 4.2 需要改的代码（4 处）

```
① client.js:24352  PICKER_TABS
   删掉 { id: "extensions", get label() { return weT("扩展"); } },
   ← 不删 = 留一个点进去全白的死页签

② client.js:27851  RopeDock 的 early return
   现在：if (hidden) { return <UpdateNotice/>; }
   问题：整个 RopeDock 组件要删，UpdateNotice 就没人渲染了
   改：把 UpdateNotice 挂到另一个常驻宿主（如 WallpaperPickerSection 外层
       或一个独立 portal），再把 RopeDock 整块删除（约 1,700 行：27296–27900 +
       ropeArtOf / ropeClamp / readRopePos 等辅助 + ROPE_IMG_* 10 个常量）

③ settings-schema.js
   ropeShown / ropeForm / ropeScale / mascotImage / mascotImageBox 5 键
   → 进 PRESET_EXCLUDED_KEYS 或直接删（删要动 7 张表，建议留着不用）

④ package.json + cordis.patch.yml
   移除 dsh-plugin-wallpaper-engine-mascot-avatar 依赖与 wallpaper-engine-mascot-avatar 行
```

### 4.3 你的现有数据会怎样

```
~/.dsh-wallpaper-engine/config.json
  mascotImage:     "mascot-mw66qqp5siz4.png"   ← 你导入过的自定义立绘
  mascotImageBox:  "52x57"
  ropeForm:        "deepseek"
  ropeScale:       1.15
  ropeShown:       false                        ← 你已经关了
```
裁撤后这 4 个键变成**死配置**（连同 `~/.dsh-wallpaper-engine/mascot/` 里那张 PNG 一起成为孤儿文件）。文件不会被删，只是没人读。**建议**：裁之前先把那张 PNG 备份出来，万一以后想恢复。

### 4.4 风险评级

| 项 | 评级 | 理由 |
|---|---|---|
| 4.2 ① 删页签 | **低** | 一行，删完导航栏少一个 tab |
| 4.2 ② UpdateNotice 搬家 | **中** | 唯一有"搬错了就丢功能"风险的步骤；需要重启后实测验证更新公告仍弹 |
| 4.2 ③ 键表 | **中** | 动 schema 要走 7 张表 + 守卫；建议**先留着不删** |
| 4.2 ④ 依赖移除 | **低** | 但必须同时从 `dependencies` 和 patch 行两处删，只删一处会触发主包启动自检**大声抛错**（`assertSubpackagesResolvable`，故意设计） |

---

## 五、执行顺序建议（如果要动手）

```
第 0 步  备份：~/.dsh-wallpaper-engine/config.json + mascot/<那张png> + 整个 profile 的 cordis.patch.yml
第 1 步  只做 A1+A2+A3（pet-link / fx / parallax）→ 重启 → 看「扩展」页签是否只剩头像卡
         （这三项互不牵连，且不影响任何页签存亡 —— 最安全的第一步）
第 2 步  做 A4+A5 + 代码 4.2①②（about + mascot + 删两个页签 + UpdateNotice 搬家）
         → 重启 → 验证：导航栏 4 个页签、关于/扩展已消失、更新公告仍弹、壁纸正常
第 3 步  若要：B1（实时渲染）+ B2（转码）→ 重启 → 验证视频壁纸仍播放
第 4 步  其余按需
```

每一步之后可跑的验证：
- `GET /wallpaper-engine/inventory` → `components` 里应只剩 `media-bridge: started`
- `GET /wallpaper-engine/diag-log` → 看 `components` 探针的 missing/failed 面
- 目视：设置页页签数、壁纸是否仍显示、玻璃是否仍在

---

## 六、等你拍板的点

1. **A 档做到哪一步？** 我建议 A1+A2+A3 先做（零风险），A4+A5 单独一批（要动代码）。
2. **B 档做不做？** 若做，B1 性价比最高（本机没 WE，纯省 ~1 MB 且无感）。
3. **设置键表动不动？** 我建议**不动**（不占界面不耗性能，动它有 7 张表 + 守卫的连带成本）。
4. **「关于」页签要保留吗？** 若保留，A4 就不能停 about-notice（否则整页空白）；折中办法是保留行、只删页签。
5. **那张自定义立绘 PNG 要不要先备份？**

---

## 附：本次新增核实的证据

```
client.js:13696   renderExtensionsTab → gate on 'mascot-avatar' → return null
client.js:13740   renderAboutTab      → gate on 'about-notice'  → return null
client.js:24352   PICKER_TABS 六项（library/appearance/playback/system/extensions/about）
client.js:1194xxx TAB_SECTION_OWNER = { library: "library" }（只 gate library）
client.js:27851   RopeDock: if (hidden) return <UpdateNotice/>
client.js:11508   syncAvatarLayer gate on 'mascot-avatar'
client.js:27296   function RopeDock()（约 1,700 行整块）
client.js:27195   ROPE_IMG_* → BASE + "/rope-art/<name>.png"（10 个）

media-bridge/lib/routes/media-bytes.js:37   for (seg of ['media','preview'])
media-bridge/lib/routes/inventory.js:106    path: `${BASE}/inventory`
media-bridge/lib/index.js:9                 「/inventory、/media、/scene-* 都消费内核媒体」

运行中实测：
GET /wallpaper-engine/inventory → 200
components {"about-notice":"started","mascot-avatar":"started","media-bridge":"started"}
wallpapers 2 张（video + image），installDir 为空
```
