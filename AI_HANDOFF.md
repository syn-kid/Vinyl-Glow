# AI 交接本

## 当前状态
- 项目名称：Vinyl Glow
- 项目版本：v0.1.0
- 当前分支：main（已 git init，尚未首次提交）
- 运行环境：Node v24.15.0 / npm 11.12.1

## 项目定位
Windows Electron 桌面透明彩唱片机音乐播放器。
核心体验：搜索、播放、歌单、歌词、3D 歌单架、切换唱片动效、透明窗口。

## 技术栈
- React 18 + TypeScript
- electron-vite（构建）
- react-three-fiber + drei（3D）
- Tailwind CSS（样式）
- howler.js（音频，可选）
- electron-builder（打包 Windows）

## 音乐数据源
- 在线模式（不做本地扫描）：网易云 + QQ 音乐双源
- 架构：业务逻辑（搜索/URL/歌词/登录）走 IPC 直调 NeteaseCloudMusicApi；封面/音频走本地 HTTP server 代理（`/api/cover` 绕防盗链 + `/api/audio` 透传 Range）
- 登录：网易云扫码（NCM API 二维码）+ QQ 扫码（webview 加载网页登录页）
- 方案文档：`.trae/documents/网易云QQ双源音乐接入方案.md`
- 进度：Step 1-3 完成（骨架/搜索/播放 URL），Step 4 起未做（歌词/登录/QQ）

## 用户偏好（摘要）
- 中文沟通，语气直接，少废话。
- 精致 / 亮色 / 高级 / 流畅，拒绝廉价渐变、过度透明、错位、闪烁、卡顿。
- 视觉质量 = 质感 + 丝滑度 + 帧数稳定，三者同时成立；不能为性能牺牲质感。
- 视觉方向：白底 + 玻璃质感 + 舞台光影 + 音乐可视化，iOS 原生感极强。
- 视觉雷区：默认白框、太素、没设计感——白底不等于裸白，必须有材质和层次。

## 红线（摘要）
- 不重写核心文件大块逻辑，先定位再小改。
- 未明确要求不上传/推送/发布。
- 只用免费/授权音乐源，不内置破解。

## 工作区地图
```
Vinyl Glow/
├── electron/              # 主进程
│   ├── main/              # 主进程入口、窗口管理（透明窗口、frameless）
│   └── preload/           # 预加载脚本，IPC 桥
├── src/                   # 渲染进程（React）
│   ├── components/        # 通用组件
│   ├── views/             # 页面级视图
│   ├── three/             # 3D 场景：歌单架、唱片、动效
│   ├── player/            # 播放器逻辑（howler、音频控制）
│   ├── services/          # 音乐源接入（本地扫描、在线搜索）
│   ├── stores/            # 状态管理
│   ├── hooks/             # 自定义 hooks
│   ├── styles/            # 全局样式、Tailwind 入口
│   └── App.tsx
├── resources/             # 图标、静态资源、安装包资源
├── docs/                  # 文档和记忆（PROJECT_MEMORY.md）
├── AGENTS.md
├── AI_HANDOFF.md
├── package.json
├── electron.vite.config.ts
├── tailwind.config.ts
└── tsconfig.json
```

## 已完成工作日志
- 2026-07-02：重写三个文档（AGENTS.md / AI_HANDOFF.md / docs/PROJECT_MEMORY.md），从旧的"项目管理系统"模板替换为 Vinyl Glow 音乐播放器项目内容。确认技术栈、音乐源、审美、红线。项目尚未 git init，尚未搭建脚手架。
- 2026-07-02：补充视觉方向到 AI_HANDOFF.md 和 PROJECT_MEMORY.md——白底+玻璃+舞台+音乐可视化+iOS 原生感；新增视觉雷区（默认白框/太素/没设计感）。
- 2026-07-02：搭建 electron-vite 脚手架。装 React18+TS+R3F+drei+Tailwind+howler+electron-builder 依赖。配置透明窗口（frame:false + backgroundMaterial:acrylic）+ 玻璃面板初始空壳。踩沙箱坑：electron Cache 和 userData 写入被拦，已用 ELECTRON_CACHE 环境变量 + app.setPath('userData') 解决。typecheck + lint 通过，npm run dev 验证窗口正常启动（4 个 electron 进程）。
- 2026-07-02：lottie 启动页。调用 lottie-main skill 生成 src/lottie/splash.json（720x360，90帧@60fps，native text ty:5，VINYL GLOW 浮现动画，settle-soft easing `.00,.65,.51,.99`，frame 0→30 入场 30→90 hold，无光晕/流光/模糊/路径形变/字距变化）。装 lottie-react + lottie-web。写 src/components/Splash.tsx（等 document.fonts.load('600 56px Inter') 后渲染避免 fallback，2.4s 后淡出 300ms 切主界面）。App.tsx 集成 showSplash 闸门。index.css 加 @font-face Inter（truetype-variations，weight 100-900）。字体放 src/assets/fonts/ 让 Vite 处理 dev/prod 路径。修复 splash.json fonts.list fName 与文字层 f 不匹配（Inter-SemiBold→Inter）。typecheck + lint 通过。验证方式待用户定。
- 2026-07-03：修复启动页灰屏。现象：npm run dev 后 Electron 灰屏静止无动画。诊断：curl 渲染器发现 `/`、`/src/main.tsx` 全 404，仅 `/@vite/client` 200 → electron-vite 默认 renderer root 指向不存在的 `src/renderer`，vite dev server 找不到 index.html 和源码 → Electron 加载空页 → body 透明露出 acrylic 灰底。修复：electron.vite.config.ts renderer 段加 `root: resolve(__dirname)`。验证：dev 起来后 `/`、`/src/main.tsx`、`/src/lottie/splash.json`、`/src/components/Splash.tsx` 全 200，4 个 electron 进程稳定运行，typecheck exit 0。注：之前"玻璃面板"其实也没渲染出来，本次才真正让渲染器服务起来。
- 2026-07-03：修复启动页无动画。现象：白屏后直接切主界面，看不到 VINYL GLOW 浮现动画。根因：lottie-react 底层 lottie-web 对 ty:5 native text 渲染不可靠（lottie-main skill player-contract 明确"不要用 lottie-web 验证"，skill 围绕 Skottie 设计），文字层不可见 → Splash 期间白屏 → 2.4s 后切主界面。修复：Splash.tsx 改用 DOM `<h1>VINYL GLOW</h1>` + CSS `@keyframes vinyl-glow-reveal` 逐参数复刻 splash.json（opacity 0→1 + translateY 14px→0 + scale 0.97→1，settle-soft cubic-bezier(0,.65,.51,.99)，500ms），index.css 加 keyframes。保留 font-loading 闸门避免 fallback 闪烁。splash.json 保留作设计规格。typecheck + lint exit 0，HMR 推送后 4 electron 进程稳定。结论：本项目文字动效走 CSS/DOM，lottie-web 仅用于纯 shape 动效。
- 2026-07-03：启动页开发中。当前已实现：白底 + VINYL GLOW 纯浮现动画（CSS/DOM），2.4s 后淡出切主界面。未完成：动效细节调整（位移/时长/easing 微调）、视觉质感提升、可能的其他启动页元素。
- 2026-07-05：换启动页 logo 字体为 Pinyon Script（手写签名字体，Google Fonts OFL）。下载 src/assets/fonts/PinyonScript-Regular.ttf（151KB，单权重 Regular 400）。index.css 加 @font-face。Splash.tsx 改动：fontFamily 'Pinyon Script' / fontWeight 400 / fontSize 56px→72px / 去掉负字距（签名字体有连笔，负字距破坏连笔） / 文字 VINYL GLOW → Vinyl Glow（手写签名字体配首字母大写）。document.fonts.load 同步改成 '400 72px "Pinyon Script"'。lint + typecheck exit 0。
- 2026-07-05：用户确认保留 Pinyon Script，要求字号"再大一点点"。Splash.tsx fontSize 72px→84px，document.fonts.load 同步改成 '400 84px "Pinyon Script"'。按记忆协议把 Pinyon Script 字体选择追加到 docs/PROJECT_MEMORY.md（含关键参数 + 视觉禁忌：禁负字距/禁全大写/禁换其他手写字体）。lint + typecheck exit 0。
- 2026-07-05：frameless 窗口自定义标题栏（右上角三按钮：最小化-最大化/还原-关闭，Windows 习惯顺序）。涉及文件：electron/main/index.ts（IPC：ipcMain.handle('window:toggle-maximize') + ipcMain.on('window:minimize'/'window:close')，注册在 app.whenReady 内，模块级 mainWindow 变量 + 闭包引用 + 'closed' 时置 null）、electron/preload/index.ts + index.d.ts（暴露 windowControls: toggleMaximize/minimize/close）、src/global.d.ts（渲染进程全局声明 WindowControlsAPI，electron/preload/index.d.ts 没被 src ts 识别）、src/components/TitleBar.tsx（顶部 36px 拖拽条 -webkit-app-region: drag + 右上角 absolute top-2 right-2 三按钮，SVG 内联图标，每个 .titlebar-btn 显式 no-drag 关键修复）、src/App.tsx（集成 <TitleBar />，仅主界面，启动页不显示）、src/styles/index.css（.titlebar-btn 玻璃质感圆形按钮 28px + backdrop-blur(12px) + hover scale 1.06，.titlebar-btn-close hover 转 #FF5F57 macOS 红）。三大根因修复：① IPC handler 之前在某次回退中丢失 → 重新加并移到 whenReady；② -webkit-app-region: drag 不继承到子元素，按钮必须显式 no-drag 否则点击被拖拽吞掉；③ 按钮 top-0 right-0 被 Win11 frameless 圆角裁切 → top-2 right-2 往内缩 8px。main/preload 改动需重启 electron（HMR 不生效）。lint + typecheck exit 0，4 electron 进程稳定。未提交。
- 2026-07-05：全屏交互优化（maximized 状态生效）。TitleBar.tsx 重构：加 controlsVisible state + hideTimer ref + showControls/scheduleHideControls callback + useEffect 监听 maximized 变化。布局改为拖拽条（z-40）+ 右上角 200×100 检测区（z-50，.titlebar-hover-zone）双层结构，按钮在检测区内（子元素，mouseenter/mouseleave 不冒泡，鼠标从检测区移到按钮上不会触发 mouseleave 避免误隐藏）。检测区 expanded（maximized）pointer-events auto 接收鼠标 / compact（窗口模式）pointer-events none 鼠标穿透不挡拖拽。控件 .titlebar-buttons-hidden（opacity 0 + pointer-events none，0.3s 淡出，鼠标穿透到检测区）/ .titlebar-buttons-visible（opacity 1 + pointer-events auto，0.3s 淡入）。逻辑：进入最大化默认隐藏 → 鼠标进 200×100 立即显示 → 鼠标离开 500ms 后隐藏 → 退出最大化清定时器始终显示。200×100 固定像素保证分辨率一致。仅渲染层改动，HMR 直接推无需重启 electron。lint + typecheck exit 0，4 electron 进程稳定。未提交。
- 2026-07-05：接入主播放界面（用户在外部写好的 Allagi 风格代码，整合进本项目）。用户依次发来 15 份代码：package.json/index.html/vite.config.ts/tsconfig.json/index.css/main.tsx/App.tsx/songs.ts/usePlayer.ts/Logo/SpotifyButton/TrackInfo/CoverArt/PlayerControls/Timeline。决策：Tailwind 升 v4 + Manrope 本地化 + 8 张占位封面 + React 19。改造内容：① 删 postcss.config.js + tailwind.config.ts（v4 不需要）；② electron.vite.config.ts renderer 段加 `tailwindcss()` 插件（@tailwindcss/vite）；③ src/styles/index.css 重写为 `@import 'tailwindcss'` + `@theme` 色板（ink #2530b8 / mint #58c28b / 白底 / surface 灰白）+ Manrope @font-face + 保留 Pinyon Script @font-face + 保留启动页 .vinyl-glow-reveal keyframes + 保留 TitleBar .titlebar-* 样式；④ 下载 Manrope-Variable.ttf（165KB）到 src/assets/fonts/；⑤ 新建 src/data/songs.ts（8 首 SoundHelix 示例曲）+ src/hooks/usePlayer.ts（Howl + html5 + 自动下一首）；⑥ 新建 6 个组件 src/components/{Logo,SpotifyButton,TrackInfo,CoverArt,PlayerControls,Timeline}.tsx（用户原代码，import 路径适配本项目）；⑦ 新建 src/views/MainView.tsx（用户 App.tsx 内容，作为主界面）；⑧ 改 src/App.tsx：保留 Splash 闸门 + TitleBar + MainView 三段结构（启动页不动）；⑨ 生成 8 张 600×600 jpg 占位封面到 public/covers/（径向渐变 + 同心圆纹路 + 中心白圆黑点，色调对应 TINTS 数组）；⑩ index.html title 改回 Vinyl Glow。版本兼容坑：@vitejs/plugin-react 6.0.3 peer dep 是 vite ^8，但 electron-vite 5 peer dep 只支持 vite 5/6/7 → 降 @vitejs/plugin-react 到 5.2.0（peer dep 含 vite ^6，React 19 过渡版）。React 19 把 JSX 命名空间从全局移到 React.JSX → src/global.d.ts 加全局 JSX 声明桥接（不动业务文件，Splash/TitleBar/App 的 `: JSX.Element` 注解原样保留）。最终依赖：react ^19.2.7 / vite ^6.4.3 / electron-vite ^5.0.0 / @vitejs/plugin-react ^5.2.0 / tailwindcss ^4.3.2 / gsap ^3.15.0 / @gsap/react ^2.1.2。lint + typecheck exit 0，dev 启动成功（vite v6.4.3，main/preload/renderer 全部 built，dev server 5174，curl / 和 /src/main.tsx 都 200）。未提交。
- 2026-07-05：底部 Timeline 缩略图扩到 13 个 + 波纹式 magnify 改造。需求：底部选择栏扩到 13 个；鼠标悬浮波纹式跳起，鼠标下方图标最大、向周围依次递减；不管图案多大，所有图标底部始终对齐静止图标的底部水平线。改造内容：① songs.ts 加 5 首（Reverie/Nightfall/Daydream/Echoes/Horizon，SoundHelix 9-13）；② MainView.tsx TINTS 从 8 色扩到 13 色（新增 #0d9488/#db2777/#7c3aed/#ea580c/#0f766e）；③ Timeline.tsx 改造——transformOrigin 保持 `center center`（不用 bottom，因 center bottom 在 gsap.quickTo + scale 配合下行为不稳定），改用 `scale` + `y` 双 quickTo 协同，`y = -BASE * (scale - 1) / 2` 精确抵消放大导致的底部下移，所有图标底部始终对齐同一水平线；容器高度从 `BASE + LIFT + 20`（134px）改成 `BASE * (1 + MAX_SCALE) + 30`（186px）确保放大后顶部不被 `overflow-x-auto` 强制 Y 裁切；④ 生成 5 张新占位图 cover-9.jpg ~ cover-13.jpg（同款径向渐变 + 同心圆纹路 + 中心白圆黑点，色调对应新增 5 个 TINTS）。lint + typecheck exit 0。HMR 推送但用户反馈后 5 个无动效。
- 2026-07-05：修复后 5 个缩略图无 magnify 动效。根因：Timeline.tsx 第 23 行 `useGSAP` 没 `dependencies`，只在 mount 时跑一次。HMR 把 songs 从 8 扩到 13 后组件 re-render 重建了 13 个 button ref，但 `useGSAP` 不重新执行 → `setters.current` 还是旧的 8 个 quickTo → `applyMagnify` 里 `setters.current[8..12]` 是 undefined → `if (!s) return` 跳过后 5 个。原代码能跑只是因为 mount 时刚好 8 ref 对 8 setter，扩容后断了。修复（双重保险）：① `useGSAP` 加 `dependencies: [songs.length]`，songs 长度变化时自动 cleanup 旧 quickTo + 重建新 setters；② `applyMagnify` 加防御性兜底，万一 useGSAP 时机错过（HMR/StrictMode 双调用），临时补建 setter，绝不跳过任何 thumb。lint + typecheck exit 0，HMR 推送 3 次。教训：动手前先读完所有相关源代码理解机制，别凭直觉改。
- 2026-07-06：Timeline 鱼眼菜单动效优化（对标 allagi.music）。完成内容：① 锚点居中修复——track 加 `position: relative`，让 button.offsetLeft 相对 track，修复 restCenters 计算偏移导致"鼠标悬浮的不是最大"；② 开放式不裁切——track 改 `overflow: visible` + `width: fit-content`，挤开后边缘 thumb 延伸到 track 外；③ 切歌无 scale 变化——移除 `ACTIVE_SCALE` 持久态，切歌时只切换黑边框（白色 span + 黑 ring）opacity，无"缩小一下"动画；④ 去紫色光晕——active button 的 boxShadow 去紫色改 `none`；⑤ 性能优化——缓存 restCenters（mount/resize 算一次）+ RAF 节流（每帧最多一次计算）+ 直接写 `style.transform`（translate3d 硬件加速）+ transition 0.4s→0.32s，快速滑动鼠标时更丝滑；⑥ 连续锚点修复抖动——之前用离散 anchorIdx（鼠标下方 dist 最小的 thumb），鼠标越过两 thumb 中点时锚点瞬间切换导致 x 偏移跳变、CSS transition 不断重启 → 抖动，改用连续锚点（找到 cx 两侧的 thumb l 和 r，用 t 插值计算 x 偏移），锚点在两 thumb 之间连续移动无跳变。所有改动 lint + typecheck 通过。
- 2026-07-06：CoverArt 彩胶唱片改为银色光盘（对标 allagi.music）。发现 allagi.music 的银色光盘实际是 PNG 图片（非 CSS 渐变），通过 curl 获取 HTML 源码和 CSS 文件，定位到 `<img class="cd" src="CD.png">` 结构（position:absolute, left:23%, width:93.5%, z-index:1, pointer-events:none）。下载原版 CD.png（486KB, 655px）到 public/cd-disc.png，将 CoverArt.tsx 的 CSS 渐变方案替换为 `<img src="/cd-disc.png" className="vinyl-disc">`，保留 GSAP 旋转动效（.vinyl-disc class + transformOrigin 50% 50%）。移除 tint 参数依赖（用下划线前缀忽略）。lint + typecheck 通过。
- 2026-07-07 ~ 2026-07-09：在线音乐源接入（网易云 + QQ 双源，参照 Mineradio-main 项目）。用户拍板方案 A：照搬 Mineradio 的网易云 + QQ 双源（含逆向 API），放行 AGENTS.md「不接入盗版 API」红线；只做在线不做本地；扫码登录；网易云+QQ 一起接。完成 Step 1-3：
  - Step 1 主进程骨架：装 NeteaseCloudMusicApi ^4.32.0；新建 electron/main/music/{types,cookie,proxy,netease,index}.ts；proxy.ts 本地 HTTP server（findOpenPort(3000) 随机端口 + 只绑 127.0.0.1），/api/cover 带 Referer: https://music.163.com/ 绕防盗链 + CORS 全开 + Cache-Control 24h，/api/audio 透传 Range（howler html5 模式拖进度条必需）；cookie.ts 落 userData（本项目被 setPath 重定向到 .electron-data/）；index.ts registerMusicIpc 注册 music:proxy-base + music:search + music:song-url；preload + 双 d.ts（electron/preload/index.d.ts + src/global.d.ts）暴露 window.music。踩坑：npm install ERESOLVE peer 冲突（@react-three/drei 要 react^18，项目用 react 19）→ 用 --legacy-peer-deps（项目本来就有此冲突）。
  - Step 2 网易云搜索：netease.ts searchNetease = cloudsearch + 缺封面用 song_detail 批量回填（照搬 Mineradio server.js 1641-1667）；src/services/music.ts 封装 search/getSongUrl/wrapAudioUrl；新建 SearchBar/SearchResults/SearchView 组件；App.tsx 接 searchView + handleSelectSong。验证：proxy 监听 3000，curl 外部图片返回 200/4286 bytes。
  - Step 3 网易云播放：netease.ts getNeteaseSongUrl 按 ['exhigh','standard','lossless'] 三 level 候选尝试，优先返回完整 URL，退而求其次返回试听片段（freeTrialInfo 兜底）；踩坑 const enum SoundQualityType 在 isolatedModules 下不接受字符串字面量 → 用 `level: level as never` 绕过（运行时仍是字符串 'exhigh' 等，NCM 函数接受）；usePlayer.ts 重写为参数化 list + 按需 getSongUrl + wrapAudioUrl 包装代理 URL；App.tsx queue 状态管理。typecheck + lint exit 0，dev 编译成功。
- 2026-07-09：底部歌曲菜单栏改造（移除顶部搜索按钮 + 加号入口 + queue 动态渲染）。需求：①移除顶部搜索控件；②底部菜单栏选中图标实时同步当前播放封面；③空状态显示空；④最多 12 首 + 末尾加号 = 13 位；⑤加号参与现有动效；⑥点加号弹搜索页。改动 5 文件：
  - [Timeline.tsx](src/components/Timeline.tsx)：从静态 `import { songs }` 改为 props 接收 `songs`(queue) 动态渲染；末尾加号按钮同尺寸 56×56 纳入 `thumbs` 数组参与现有 magnify 动效（余弦衰减 + 挤开算法零改动）；加号样式 ink 蓝虚线边框 + 玻璃背景，hover 转实色显眼；底部状态行显示 `N/12`，空状态提示「点击 + 添加歌曲到播放列表」。
  - [App.tsx](src/App.tsx)：queue 初始 `[]`（空状态启动）；handleSelectSong 从「替换为单曲」改为「追加到末尾并播放」（用 queueMicrotask 确保 setQueue 后再 select）；满 12 首不再追加；onOpenSearch 传给 MainView→Timeline 的 onAdd。
  - [MainView.tsx](src/views/MainView.tsx)：删除 Logo 旁搜索按钮；Timeline 传 songs={player.list}/onAdd；空状态占位（Vinyl Glow / 未在播放 + CoverArt 渐变占位）。
  - [usePlayer.ts](src/hooks/usePlayer.ts)：toggle/next/prev/select 在空 list 时直接 return 避免 howler 报错；返回 list 供 MainView 传 Timeline。
  - [CoverArt.tsx](src/components/CoverArt.tsx)：cover 为空时显示渐变 + 唱片纹路占位，避免 broken img。
  设计取舍：加号位置实现为「跟随队列末尾」（第 N+1 位，满 12 首时位于第 13 位），空状态时加号居中作空状态入口（比「加号永远最右、左边一大片空白」更协调）；加号动效完全复用歌曲图标 magnify 逻辑（同一 thumbs 数组、同一套余弦衰减 + 挤开算法），零改动。typecheck + lint exit 0，HMR 推送成功。未提交。

- 2026-07-10：实时可拖动歌词动效框（苹果液态玻璃 + 水性形变）。新建 [LyricFrame.tsx](src/components/LyricFrame.tsx)，GSAP elastic.out(1, 0.55) scale in 入场动效，液态玻璃质感（backdrop-filter blur(28px) saturate(180%) brightness(1.08) + 多层 inset/outset 阴影 + 顶部高光层）。初始长度=13组件总长 896px（THUMB_SIZE 56 × 13 + THUMB_GAP 14 × 12），圆角 20px（介于 Timeline 12px 和 CoverArt 24px 之间），初始高度 56px。拖动交互：pointer events + setPointerCapture，仅左右拖动垂直固定；超过 MORPH_THRESHOLD(32px) 触发水性形变（内容 skewX + scaleX + 偏移被挤向拖动方向）+ 液态边缘（borderRadius 动态变大）+ 动态尺寸（水平长度=中心到最近边界 windowCenterX，垂直高度=Timeline 顶部到 CoverArt 底部）。弹性回弹 elastic.out(1, 0.5)。性能：gsap.quickTo 复用 tween + will-change + transform 优先（width/height 不可避免但仅影响单元素）。Timeline.tsx track 加 data-timeline-track 属性供 LyricFrame 查询位置。MainView.tsx 在 PlayerControls 和 footer 之间插入 LyricFrame，active=queue 非空时激活。typecheck + lint exit 0。未提交。

- 2026-07-10（续）：液态玻璃重写（参考 archisvaze/liquid-glass）。需求：①降低磨砂增强液态流动；②水物理质感（表面张力/波纹/光反射）；③背景替换为模糊专辑图填充不变形。方案从「主色渐变」改为「专辑图背景 + SVG 液态折射」。[LyricFrame.tsx](src/components/LyricFrame.tsx) 重写：①专辑图双层 imgA/imgB object-cover + scale(1.06) 填充（避免 displacement 边缘露白），交叉淡化切歌；②SVG feTurbulence(baseFrequency 0.008, numOctaves 2, seed 92) + feGaussianBlur(stdDeviation 2) + feDisplacementMap(scale 77) 液态折射层（backdrop-filter blur(3px) saturate(160%) + filter url(#liquid-glass-distortion)）；③表面张力高光层（inset 0 0 20px -5px rgba(255,255,255,0.55) + 顶部 inset 1.5px 0.85 + 底部 inset 1px 0.25）；④顶部光反射带（白色渐变 0.30→0）；⑤外投影黑色双层。active=player.list.length>0，无歌隐藏。删除 useAlbumColors.ts（主色提取不再需要）。typecheck + lint exit 0。未提交。

- 2026-07-11：LyricFrame 液态玻璃组件定稿（移除拖动 + 修复首张图空白）。需求演进：从「可拖动 + 三位置吸附」简化为「固定居中玻璃框」，从「主色渐变」改为「专辑图背景 + SVG 液态折射」。[LyricFrame.tsx](src/components/LyricFrame.tsx) 最终状态：①active=player.list.length>0，无歌完全隐藏（minHeight:0 + autoAlpha:0）；②入场动效 gsap.fromTo scale 0.85→1 + autoAlpha 0→1 power3.out 0.8s；③专辑图双层 imgA/imgB object-cover + scale(1.06) 交叉淡化切歌，img.onLoad 加载完成后才淡入（修复首张图未加载完就显示空白的问题）；④SVG 液态 filter（feTurbulence + feDisplacementMap scale 77）折射扭曲；⑤三层玻璃：backdrop blur(3px) saturate(160%) + filter url() 折射层 / tint rgba(255,255,255,0.10) + inset 高光（表面张力 0.55 + 顶部 0.85 + 底部 0.25）/ 顶部光反射带渐变；⑥固定居中不可拖动（已移除所有拖动/吸附逻辑 + snapPos 指示器）。[MainView.tsx](src/views/MainView.tsx) 传 active/cover/title。typecheck + lint exit 0。未提交。

- 2026-07-11：PlayerControls 液态玻璃化（已撤回）。尝试给播放控件套用与 LyricFrame 同等参数的液态玻璃质感（三层结构 + SVG filter），并把 SVG filter 提升到 App.tsx 全局共用。用户要求撤回，已恢复 [PlayerControls.tsx](src/components/PlayerControls.tsx) 为原 bg-surface 样式、[App.tsx](src/App.tsx) 删除全局 SVG filter、[LyricFrame.tsx](src/components/LyricFrame.tsx) 恢复内嵌 SVG filter。typecheck exit 0。未提交。

- 2026-07-15：歌词显示功能完整接入（参考 beautiful-lyrics 项目方案）。完成网易云 Step 4 + LyricFrame 卡拉OK逐字显示。改动 8 文件 + 新建 1 文件：
  - 主进程 [netease.ts](electron/main/music/netease.ts)：加 `getNeteaseLyric(id)` 调 `lyric` 接口，返回 `{lyric, tlyric, yrc}`（行级LRC + 翻译 + 逐字YRC，三者皆可空）。
  - 主进程 [index.ts](electron/main/music/index.ts)：注册 `music:lyric` IPC handler，QQ 留 Step 6 占位。
  - preload [index.ts](electron/preload/index.ts) + [index.d.ts](electron/preload/index.d.ts) + [src/global.d.ts](src/global.d.ts)：三处同步暴露 `window.music.getLyric(provider, id)`。
  - 渲染端 [services/music.ts](src/services/music.ts)：加 `LyricResult` 类型 + `getLyric()` 封装。
  - 新建 [src/utils/lyricParser.ts](src/utils/lyricParser.ts)：歌词解析器。`parseLRC` 解析行级 `[mm:ss.xx]text`，用下一行起始推算 duration；`parseYRC` 解析网易云逐字 `[行起始ms,行时长ms]字(起始ms,时长ms)...`；`parseLyrics` 入口按 yrc > lyric 优先级返回 `ParsedLyrics{lines, hasSyllables, translations}`；`findActiveLineIndex` 二分查找当前行；`computeLineProgress` 计算填充进度。
  - [usePlayer.ts](src/hooks/usePlayer.ts)：加 `currentTime` state（RAF 循环 60fps 更新，Howl 无 tick 事件）+ `seek(time)` 方法 + `lyrics` state（切歌时自动 `getLyric` + `parseLyrics`，本地 SoundHelix 示例曲无歌词时为 null）。注册 `play`/`pause`/`end` 事件管理 RAF 生命周期。
  - [LyricFrame.tsx](src/components/LyricFrame.tsx)：保持 56px 玻璃框不变，内容区从单行歌名升级为歌词显示。Props 加 `lyrics?` + `currentTime?`。`useMemo` 找当前行 + 翻译；行切换 GSAP `elastic.out(1, 0.55)` 弹性入场（autoAlpha 0→1 + y 6→0 + scale 0.92→1，0.55s）。渲染分三态：① 无歌词/时间在首行前 → 降级显示歌名（原行为）；② yrc 逐字模式 → 每个音节独立 `<span>` 用 `background-clip:text` + `linear-gradient(90deg, rgba(255,255,255,0.95) X%, rgba(255,255,255,0.40) X+8%)` 卡拉OK填充，X 由 `computeSyllableProgress` 实时计算；③ lrc 行级模式 → 整行渐变填充，X 由 `computeLineProgress` 计算。颜色参考 beautiful-lyrics（已唱 0.95 / 未唱 0.40 + 8% 羽化带）。字号 15px font-semibold + drop-shadow 增强可读性。
  - [MainView.tsx](src/views/MainView.tsx)：传 `lyrics={player.lyrics}` + `currentTime={player.currentTime}` 到 LyricFrame。
  - 关键设计取舍：① 保持 56px 单行不展开为完整面板（用户拍板"保持当前玻璃框"）；② 不做行级滚动/前后行模糊（56px 容不下多行，仅当前行 + 弹性入场）；③ 不实现 beautiful-lyrics 的 Spring 物理引擎（项目用 GSAP，弹性用 `elastic.out` 近似，工程量更低且效果接近）；④ 不做用户滚动暂停机制（单行无滚动需求）。
  - typecheck + lint exit 0。未提交。main/preload 改动需重启 electron（HMR 不生效）。待用户实测：搜索→点歌→歌词加载→逐字填充→切歌切词动效。

- 2026-07-15（续）：LyricFrame 重构为完整多行歌词面板（仿 beautiful-lyrics 视频动效）。用户反馈"没显示歌词 + 要仿照视频里的动效"，原 56px 单行方案容不下视频里的多行 + 模糊 + 滚动动效，重做为 380px 多行面板。改动 3 文件：
  - [LyricFrame.tsx](src/components/LyricFrame.tsx)：从 56px 单行扩到 **380px 多行面板**，宽 896px 保持（对齐 Timeline 13 thumb），圆角 24px。核心动效参考 beautiful-lyrics 源码参数：
    - **多行渲染**：所有歌词行 map 渲染，行高 1.45、行间距 6px、当前行字号 22px/字重 700、其他行 18px/字重 500。
    - **当前行定位**：`useEffect` 监听 activeIndex 变化，`scrollRef.scrollTo({ behavior: 'smooth' })` 滚到视口中心偏上 0.8 行高处（参考 MoveToActiveLyrics）。
    - **距离模糊**：每行按距 activeIndex 的行数算 `text-shadow: 0 0 Xpx rgba(255,255,255,0.5)`（非 filter:blur），1.25px/行封顶 5px（4 行外满模糊）。参考 beautiful-lyrics `DistanceToMaximumBlur=4, BlurScale=1.25`。
    - **透明度三态**：当前行 1.0 / 已播 0.497 / 未播 0.51，过渡 `0.25s cubic-bezier(0.61,1,0.88,1)`。
    - **当前行缩放**：scale 1.05，过渡 `0.1s cubic-bezier(0.37,0,0.63,1)`。
    - **卡拉OK逐字填充**：yrc 模式每音节独立 `<span>` + `background-clip:text` + `linear-gradient(90deg, rgba(255,255,255,0.95) X%, rgba(255,255,255,0.40) X+20%)`，X 由 `computeSyllableProgress` 实时算；lrc 模式整行渐变。羽化带 20%。
    - **上下 mask 淡出**：`mask-image: linear-gradient(180deg, transparent 24px, black 64px, black calc(100%-80px), transparent calc(100%-24px))`，顶部 24-64px / 底部 80-24px 渐隐。
    - **滚动条隐藏**：`scrollbar-width:none` + `ms-overflow-style:none`。
    - **液态玻璃保持**：专辑图双层背景 + SVG feTurbulence 折射 + 表面张力高光 + 顶部光反射带，blur 从 3px 提到 8px（多行文本可读性）。
  - [CoverArt.tsx](src/components/CoverArt.tsx)：加 `size?: number` prop（默认 300），唱片右移量按比例算 `Math.round(size*0.187)`。让 MainView 能缩小封面给歌词面板让位。
  - [MainView.tsx](src/views/MainView.tsx)：CoverArt 传 `size={180}`（从 300 缩到 180），main `gap-7` → `gap-3`。让 380px LyricFrame 在 1200x800 窗口能塞下（TitleBar 36 + main ~280 + LyricFrame 380 + Timeline 134 + padding ≈ 830px 临界）。
  - 关键设计取舍：① 不做 beautiful-lyrics 的 Spring 物理引擎（项目用 GSAP，CSS transition + cubic-bezier 近似）；② 不做 4 层旋转模糊封面 WebGL 背景（保留现有液态玻璃 + 专辑图，工程量更低）；③ 不做间奏三圆点（先核心动效，间奏后续补）；④ 不做用户滚动暂停 750ms 机制（后续补）；⑤ 不做逐字母拆分强调动效（音节级已够）。
  - 没显示歌词的根因：SoundHelix 示例曲（songs.ts 13 首）无 provider/sourceId，usePlayer 的歌词 useEffect 直接 setLyrics(null) 降级显示歌名。**用户需用搜索功能点网易云歌曲才有歌词**。
  - typecheck + lint exit 0。未提交。main/preload 改动需重启 electron。待用户实测：搜索网易云歌曲→歌词加载→多行滚动→当前行卡拉OK填充→切行平滑滚动→前后行距离模糊。

- 2026-07-15（续4）：音频实例重叠 bug 修复 + 歌单模块完全重构为 mineradio 纵向风格。改动 3 文件：
  - **修复音频实例重叠**：[usePlayer.ts](src/hooks/usePlayer.ts) toggle 重写。根因：旧方案用 `sound.once('play', () => sound.seek(pausePosRef.current))` 回调 seek，时序不可控（play 事件触发时 audio 已从 0 开始播放 0.x 秒，seek 后产生"从头声音 + 跳到暂停位置"的听觉重叠）。新方案：① 用 `sound.playing()` 判断真实状态（不依赖 React state 异步）；② **先 seek 再 play**（seek 在 html5 模式下同步设置 audio.currentTime，确保 play 从正确位置开始）；③ useEffect 创建新 sound 前 unload 旧的（双重保险，`howlRef.current = null` 置空引用），cleanup 时 unload + 置 null。彻底避免两个音频实例并存。
  - **歌单模块完全重构**：[ExpandedView.tsx](src/components/ExpandedView.tsx) 删除 `renderHorizontalTimeline`（与未展开态 Timeline 有关联的横向歌单），新建 `renderVerticalQueue` 纵向歌单，**与未展开态 Timeline 完全独立**（代码、常量、交互逻辑零关联）。参考 `C:\Users\13412\有点意思\gg\Mineradio-main\public\index.html` 的 `.queue-item` 设计：
    - 布局：纵向 flex 列表 + gap 6px + overflow-y-auto + 3px 细滚动条（ink 蓝 35%）
    - 列表项：flex 横向（缩略图 42×42 圆角 7px + 歌名 12px 白88% + 歌手 10.5px 白42%），padding 8px 圆角 10px
    - 默认态：背景 `rgba(255,255,255,0.025)` + 边框 `rgba(255,255,255,0.05)`
    - 当前播放 `.now`：背景 `rgba(37,48,184,0.14)` + 边框 `rgba(37,48,184,0.42)` + boxShadow + 左侧 ink→mint 渐变指示条 + translateX(2px) 微右移
    - 入场动画：GSAP fromTo x:-8 y:6 autoAlpha 0→1 stagger 0.012 duration 0.22 delay 0.25（参考 mineradio animateListItems）
    - 自动滚动：当前歌曲变化时 `scrollTo({ behavior: 'smooth' })` 滚到视口中心（参考 mineradio smoothScrollToItem）
    - 添加按钮：队列未满时纵向排列在末尾，ink 蓝虚线边框
  - **布局调整**：左侧 35%→32%，右侧 65%→68%（扩大歌词区）。左侧重新分区：上区（CoverArt 220→160 + 歌曲信息 + PlayerControls）+ 歌单标题栏（"歌单 / 队列" + N/12 计数）+ 下区（纵向歌单 flex-1 占据剩余空间，min-h-0 允许收缩）。
  - **歌词展示优化**：字号增大（active 24→26 / idle 18→19），行高 1.5→1.55，行间距 8→10，提升可读性。
  - **滚动条样式**：[src/styles/index.css](src/styles/index.css) 加 `.queue-scroll` 类（3px 宽 + ink 蓝 35% thumb + hover 55%，Chromium ::-webkit-scrollbar + Firefox scrollbar-color 双兼容）。
  - typecheck + lint exit 0。未提交。renderer 改动 HMR 生效。待用户实测：① 暂停后再次播放无重叠（只有一个音频实例）；② 展开态左侧纵向歌单（mineradio 风格 + 当前歌曲高亮 + 自动滚动）；③ 歌词区扩大 + 字号增大；④ 点击歌单项切歌。

- 2026-07-15（续3）：4项修复（专辑图尺寸恢复 + 左侧透明毛玻璃增强 + 展开态横向歌单 + 暂停续播bug）。改动 3 文件：
  - 修复1：默认态专辑图尺寸恢复至历史版本（180 → 300）。[MainView.tsx](src/views/MainView.tsx) CoverArt `size={180}` → `size={300}`，解决展开态收起后专辑图过小问题。
  - 修复2：展开态左侧 35% 区域改为透明毛玻璃 + 增强清晰度（参考 archisvaze/liquid-glass 项目）。[ExpandedView.tsx](src/components/ExpandedView.tsx) 左侧卡片背景 `rgba(255,255,255,0.10)` → `rgba(255,255,255,0.04)` 更透明；新增液态玻璃折射层（`backdropFilter: blur(16px) saturate(220%)` + `filter: url(#liquid-glass-distortion-expanded)` SVG feTurbulence + feDisplacementMap scale 77，saturate 220% 增强清晰度不过度模糊）；新增表面张力高光层（多层 inset boxShadow 0 0 32px rgba(255,255,255,0.50) + 顶部 1.5px 0.60 + 底部 1px 0.20）。
  - 修复3：展开态左侧下方歌单从竖列改为横向丝滑滚动选择（与未展开态 Timeline 一致）。[ExpandedView.tsx](src/components/ExpandedView.tsx) 删除 `renderVerticalTimeline`，新增 `renderHorizontalTimeline`：缩略图从 32px(h-8 w-8) → 56px（THUMB_SIZE，与未展开态 Timeline BASE 一致）；`overflow-x-auto` 横向滚动 + `scrollbar-width:none` 隐藏滚动条；当前歌曲 `ACTIVE_SCALE_THUMB = 1.18` 放大 + 白色激活边框；hover `group-hover:scale-[1.12]` 放大；过渡 `cubic-bezier(0.34, 1.3, 0.64, 1)` 丝滑弹性；添加按钮同尺寸同动效参与横向排列。
  - 修复4：歌曲暂停后再次播放从头开始的 bug。[usePlayer.ts](src/hooks/usePlayer.ts) 加 `pausePosRef = useRef(0)` 缓存暂停位置；`toggle` 中 pause 时 `pausePosRef.current = sound.seek()` 保存、play 时 `sound.once('play', () => sound.seek(pausePosRef.current))` 续播；切歌 useEffect 开头 `pausePosRef.current = 0` 重置避免新歌 seek 到旧歌位置。根因：Howler html5 模式下 pause/play 通常自动续播，但部分代理 URL / Electron 场景会丢失位置。
  - typecheck + lint exit 0。未提交。renderer 改动 HMR 生效。待用户实测：① 默认态专辑图大小；② 展开态左侧透明毛玻璃质感 + 清晰度；③ 展开态横向歌单动效（hover/active 放大 + 滚动）；④ 暂停后再次播放续播位置准确。

- 2026-07-15（续2）：歌词框改为"长条单行 + 点击展开双区域沉浸视图"交互方案。用户反馈 380px 常驻面板挤压原画面，改为默认长条单行、点击展开为双区域沉浸视图。改动 2 文件 + 新建 1 文件：
  - [LyricFrame.tsx](src/components/LyricFrame.tsx)：从 380px 多行面板**改回 56px 长条单行**（宽 480px，圆角 18px）。Props 加 `onExpand` / `onWheelPrev` / `onWheelNext`。单行显示当前歌词 + 卡拉OK渐变填充（yrc 聚合音节进度 / lrc 整行进度）。交互：① `onClick` 触发展开；② `onWheel` 滚轮切歌（节流 380ms，上滚 prev / 下滚 next）；③ hover 反馈 `scale(1.02) + brightness(1.1)` + 右下角展开图标淡入。液态玻璃质感保留（专辑图双层背景 + SVG 折射 + 表面张力高光 + 顶部光反射带）。
  - 新建 [ExpandedView.tsx](src/components/ExpandedView.tsx)：双区域沉浸视图。**左侧 35%**（大曲率圆角 32px 玻璃卡片）：CoverArt 220px（旋转持续）+ 歌曲信息 + PlayerControls（形状不变直接复用）+ 竖列 Timeline（歌曲列表点击切换 + 添加按钮，最大高 200px 滚动）。**右侧 65%**（大曲率圆角 32px）：模糊专辑图背景（blur 28px + scale 1.18）+ 暗化遮罩 + 液态玻璃层 + 多行歌词面板（24/18px 字号、距离模糊 1.5px/行封顶 6px、透明度三态 1.0/0.5/0.55、当前行 scale 1.04、上下 mask 淡出、scroll-behavior smooth 滚动定位、卡拉OK逐字/整行填充）。点击右侧 → 收起。GSAP 入场：整体淡入 + 左右从两侧滑入（0.6s power3.out）。
  - [MainView.tsx](src/views/MainView.tsx)：加 `isExpanded` state。collapsed 渲染原画面（Logo + Button + CoverArt 180 + TrackInfo + Controls + 长条 LyricFrame + 横向 Timeline）；expanded 渲染 ExpandedView（队列非空才允许展开）。LyricFrame 传 `onExpand/onWheelPrev/onWheelNext`。
  - 关键设计取舍：① CoverArt 在 expanded 时重新 mount（旋转从头开始，但 isPlaying 立即 play，视觉上连续可接受；如需角度绝对连续需用 ref 传递角度，工程量大暂不做）；② 竖列 Timeline 是简化版（无 magnify 波纹动效，仅点击切换 + 当前高亮 + 添加按钮）；③ 滚轮切歌仅节流不做惯性滚动（丝滑度够用）；④ 收起提示用 group-hover 淡入（不用常驻避免干扰歌词阅读）。
  - typecheck + lint exit 0。未提交。renderer 改动 HMR 生效（main/preload 未改，无需重启 electron）。待用户实测：搜索网易云歌曲→长条单行歌词→点击展开→双区域沉浸视图→多行滚动卡拉OK→点击右侧收起→滚轮切歌。

- 2026-07-15（续5）：mineradio 深度模仿 + ControlPanel iOS 浅灰毛玻璃控制面板（参考 beautiful-lyrics / liquid-glass / mineradio 三项目）。完成 5 项需求，改动 2 文件：
  - **需求1：mineradio 滚动歌单复刻 + 左侧留白减 20%**。[ExpandedView.tsx](src/components/ExpandedView.tsx) 严格复刻 `Mineradio-main/public/index.html` 的 `bindSmoothWheelScroll` + `smoothScrollToItem`：
    - 滚轮接管：`useEffect` 在 queueListRef 上绑定 `wheel` 事件（passive:false），`e.preventDefault()` + GSAP tween `scrollTop` duration 0.24 power2.out；deltaMode 1=*18 / 2=*clientHeight 标准化；ctrlKey 放行（页面缩放）；横向滚动放行
    - 滚轮状态互斥：`wheelTargetTop` ref + `wheelTween` ref，wheel tween 和 auto tween 互相 kill 后同步 targetTop 避免 scrollTop 跳变
    - 自动滚动定位：当前歌曲变化时 `smoothScrollToItem` align 0.42（居中偏上 8%）+ duration 0.30 power2.out + clamp 到 [0, scrollHeight-clientHeight]
    - 左侧留白减 20%：`p-5` → `p-4`（20px → 16px），优化空间利用率
  - **需求2：专辑照片统一增大 30%**。[ExpandedView.tsx](src/components/ExpandedView.tsx)：
    - CoverArt `size={160}` → `size={208}`（+30%，展开态可转动专辑图）
    - `QUEUE_THUMB` 42 → 55（+30%，纵向歌单缩略图）
    - `QUEUE_GAP` 6 → 8（缩略图放大后间距同步放大保持视觉呼吸）
    - 歌单容器加 `pb-12`（48px 底部 padding，避免最后一项被齿轮按钮遮挡）
  - **需求3：ControlPanel iOS 浅灰毛玻璃控制面板**。新建 [ControlPanel.tsx](src/components/ControlPanel.tsx)：
    - 4 参数：bgBlur(0-40px 默认16) / bgSaturate(100-280% 默认220) / lyricScale(80-140% 默认100) / lyricIntensity(0-200% 默认100)
    - CSS 变量驱动：`--vg-bg-blur` / `--vg-bg-saturate` / `--vg-lyric-scale` / `--vg-lyric-intensity`，参数变化直接写 `document.documentElement.style`，**不触发 React re-render，不影响音频播放**，响应延迟 <1ms（满足 <100ms 要求）
    - localStorage 持久化（key: `vg-control-panel`），模块级初始化避免首帧闪烁
    - 齿轮按钮：absolute bottom-3 left-4 z-30，34×34 圆角 10px，浅灰毛玻璃 rgba(255,255,255,0.18) + blur(20px) saturate(1.8) + inset 高光，hover scale 1.05 + active scale 0.95
    - iOS 面板：absolute bottom-12 left-4 z-30 width 280px，rgba(255,255,255,0.18) + blur(28px) saturate(1.8) + inset 高光 + 12px 40px 阴影
    - GSAP 展开/收起：fromTo autoAlpha 0→1 + y 12→0 + scale 0.96→1 duration 0.35 power3.out（开）/ to autoAlpha 0 + y 8 + scale 0.96 duration 0.25 power2.in（收）
    - 原生滑块 `<input type="range">` + accent-color #2530b8（ink 蓝），标签 64px + 滑块 flex-1 + 数值 42px font-mono tabular-nums
    - 重置按钮：恢复 DEFAULTS
  - **需求4：参考资源整合**。
    - beautiful-lyrics 歌词动效：已在前续会话接入（卡拉OK background-clip:text + 距离模糊 text-shadow + 透明度三态 + 上下 mask 淡出 + 弹性入场）；本次把 text-shadow blur 和 active scale 接入 `--vg-lyric-intensity` 实时调节
    - liquid-glass 毛玻璃参数调节：iOS 风格面板 + 原生滑块 + CSS 变量驱动 + localStorage 持久化，参考 `archisvaze/liquid-glass` 的浅灰变体 rgba(255,255,255,0.55) + blur(40px) + saturate(1.8)
    - mineradio 滚动歌单：`bindSmoothWheelScroll` + `smoothScrollToItem` 严格复刻（wheel 接管 + GSAP tween + align 0.42 + duration 0.30）
  - **需求5：性能与体验**。
    - 实时预览：<1ms（CSS 变量直写 documentElement.style，不触发 React re-render）
    - 平滑过渡：GSAP power3.out 0.35s 展开 / power2.in 0.25s 收起
    - 不影响播放：参数调节只改 CSS 变量，不触碰 howler.js 音频实例
    - 防止 lyricIntensity=0 让歌词消失：active scale 公式 `1 + (ACTIVE_SCALE - 1) * intensity`（intensity=0 时 scale=1.0 无放大，100% 时 1.04，200% 时 1.08 双倍增强）
  - **集成 ControlPanel 到 ExpandedView**：左侧卡片底部放置（依赖左侧卡片 relative），齿轮按钮 absolute bottom-3 left-4，面板 absolute bottom-12 left-4 width 280px，不挤压歌单空间
  - **应用 CSS 变量到毛玻璃层和歌词**：
    - 左侧液态玻璃折射层 `backdropFilter: blur(var(--vg-bg-blur, 16px)) saturate(var(--vg-bg-saturate, 220%))`
    - 歌词字号 `calc(${ACTIVE_FONT_SIZE}px * var(--vg-lyric-scale, 1))`
    - 歌词 text-shadow `0 0 calc(${blurPx}px * var(--vg-lyric-intensity, 1)) rgba(255,255,255, calc(0.5 * var(--vg-lyric-intensity, 1)))`
    - 歌词 active scale `calc(1 + 0.04 * var(--vg-lyric-intensity, 1))`
  - typecheck + lint exit 0。未提交。renderer 改动 HMR 生效（main/preload 未改，无需重启 electron）。待用户实测：① 展开态左侧齿轮按钮→点击弹出 iOS 毛玻璃面板→拖动 4 个滑块→毛玻璃模糊度/清晰度/歌词大小/律动强度实时变化；② 滚轮在歌单上滚动→GSAP 平滑滚动（非原生卡顿）；③ 切歌→歌单自动滚动定位（align 0.42 居中偏上）；④ CoverArt 208px + 缩略图 55px 视觉突出；⑤ 左侧 padding 16px 空间利用率提升。

- 2026-07-15（续6）：UI 交互与视觉效果优化（按钮组件 + 专辑滚动模块 + 展开界面视觉）。完成 3 项需求，改动 4 文件：
  - **需求1：按钮组件实现**。[PlayerControls.tsx](src/components/PlayerControls.tsx) 加 `variant?: 'light' | 'glass'` prop：
    - light（默认）= 原浅灰胶囊 `bg-surface`，适配白底主界面
    - glass = iOS 深色毛玻璃 `rgba(255,255,255,0.10)` + `blur(20px) saturate(1.8)` + inset 高光 + 12px 阴影，适配展开态深色背景
    - 按钮颜色：glass 时白色文字（70% 默认 / 100% hover），light 时深色文字（55% / 100%）
    - hover/active 反馈：GSAP scale 1.14 power3.out 0.25s + 颜色过渡 0.2s
    - 展开态 ExpandedView 传 `variant="glass"`
  - **需求2：专辑滚动模块交互**。[CoverArt.tsx](src/components/CoverArt.tsx) 重写播放状态动画：
    - **播放放大**：isPlaying 时 `scale: 1.04`（PLAY_SCALE，轻微放大）duration 0.9s power3.out，完成后启动小律动 breathe
    - **暂停缩小**：!isPlaying 时 `scale: 0.94`（PAUSE_SCALE，最小尺寸限制，不过度缩小）duration 0.9s power3.out
    - **小律动**：breathe `scale: 1.048`（PLAY_SCALE + BREATHE_SCALE - 1 = 1.048）yoyo sine.inOut 2.2s repeat -1，幅度仅 ±0.008，避免过度干扰视觉焦点
    - **时序安全**：放大/缩小 tween 加 `overwrite: true`，避免快速切换播放/暂停时 tween 冲突；breathe 在 useEffect 开头 kill 旧的
    - **光盘右侧露出**：drop-shadow 从 `0 18px 40px`（下方阴影）改为 `16px 0 28px`（右侧阴影），避免下方阴影被误判为"下侧露出"；光盘容器 `right: -discOffset, top: 0, height: coverSize`，与封面同高垂直居中，确保光盘只从右侧露出
  - **需求3：展开界面视觉优化**。[ExpandedView.tsx](src/components/ExpandedView.tsx) 歌单改为纯图片紧凑网格布局：
    - **2 列网格**：`display: grid; gridTemplateColumns: repeat(2, 1fr)`，平衡图片大小和列表长度（12 首歌 = 6 行，可滚动）
    - **纯图片无留白**：每个 item = `aspect-square w-full`，`padding: 0`，图片 `object-cover` 填满 item，消除模块内留白空间
    - **增大尺寸**：从固定 55px 改为自适应宽度（约 160-170px），图片显著增大
    - **当前播放高亮**：ink 蓝边框 2px + scale 1.03 + boxShadow + 右上角 pulse 指示点（ink 蓝 + mint 绿双层呼吸动画）
    - **hover 反馈**：暗色渐变蒙层 + 歌名/歌手 overlay（替代常驻文字，紧凑且美观）
    - **添加按钮**：与歌单项同尺寸正方形，ink 蓝虚线边框 + hover scale 1.10
  - **CSS 改动**：[src/styles/index.css](src/styles/index.css) 加 `@keyframes pulse`（scale 1→1.4 + opacity 0.85→0，1.6s ease-in-out infinite），用于当前播放指示点
  - typecheck + lint exit 0。未提交。renderer 改动 HMR 生效（main/preload 未改，无需重启 electron）。待用户实测：① 展开态播放控件 iOS 玻璃质感（深色毛玻璃 + 白色文字 + hover 反馈）；② CoverArt 播放时放大 1.04 + 小律动，暂停时缩小 0.94；③ 光盘只从右侧露出（无下方阴影）；④ 歌单 2 列纯图片网格（无留白 + hover 蒙层显示歌名 + 当前播放 ink 蓝边框 + pulse 指示点）。

- 2026-07-15（续7）：按用户要求撤回续6的部分改动。保留"展开态歌单纯图片网格化"（ExpandedView 2 列网格），撤回其余 2 项：
  - **撤回 PlayerControls glass variant**：[PlayerControls.tsx](src/components/PlayerControls.tsx) 移除 `variant?: 'light'|'glass'` prop 及 iOS 深色毛玻璃分支，恢复为单一浅灰胶囊版本（`var(--color-surface)` + 深色文字）。ExpandedView 调用去掉 `variant="glass"`。
  - **撤回 CoverArt 播放状态动画**：[CoverArt.tsx](src/components/CoverArt.tsx) 移除 PLAY_SCALE/PAUSE_SCALE/BREATHE_SCALE 常量、breathe ref、播放放大/暂停缩小/小律动 tween，恢复为仅 spin.play/pause；光盘 drop-shadow 从右侧 `16px 0 28px` 恢复为下方 `0 16px 28px`。
  - **保留**：ExpandedView 歌单 2 列纯图片网格（`gridTemplateColumns: repeat(2,1fr)` + aspect-square + pulse 指示点）；[index.css](src/styles/index.css) `@keyframes pulse` 作为歌单指示点配套保留。
  - typecheck + lint exit 0。未提交。renderer 改动 HMR 生效。

- 2026-07-15（续8）：播放呼吸律动动效 + 歌单图片修复 + 简约风格优化。改动 2 文件：
  - **CoverArt 播放呼吸律动**：[CoverArt.tsx](src/components/CoverArt.tsx) 添加 `BREATHE_SCALE=1.012` 常量 + `breathe` ref，播放时启动 GSAP yoyo 呼吸动画（scale 1→1.012，duration 2.8s，sine.inOut），暂停时恢复 scale 1（0.6s power3.out），`overwrite:true` 防 tween 冲突。
  - **歌单专辑图片修复**：[ExpandedView.tsx](src/components/ExpandedView.tsx) 歌单项 button 去掉 `overflow-hidden`，图片 borderRadius 从 `QUEUE_RADIUS-2` 改为 `QUEUE_RADIUS`（与 button 一致），边框移到 img 上，确保图片完整四边曲率不被剪切遮挡。
  - **简约风格优化**：歌单当前播放边框从 ink 蓝 `rgba(37,48,184,0.85)` 改为白色半透明 `rgba(255,255,255,0.60)`；指示点底色从 ink 蓝改为白色半透明；空状态按钮去掉 `border-dashed`；添加按钮从 ink 蓝虚线 `rgba(37,48,184,0.35)` + 蓝图标改为白色细线 `rgba(255,255,255,0.12)` + 白图标 `rgba(255,255,255,0.50)`，hover 变为 `white/70`。
  - typecheck + lint exit 0。未提交。renderer 改动 HMR 生效。

- 2026-07-15（续9）：呼吸律动幅度优化 + 歌词字号增大 + 高亮增强 + 歌单区域扩展 + 圆润曲率。改动 2 文件：
  - **CoverArt 呼吸律动幅度优化**：[CoverArt.tsx](src/components/CoverArt.tsx) 恢复 `size` prop（默认 300）+ `coverSize`/`discOffset` 变量驱动尺寸；呼吸 scale 从 1.012 调到 **1.022**（视觉幅度更明显但仍克制），duration 从 1.6s 调到 **2.4s**（更舒缓），加 `overwrite:true` 防 tween 冲突。
  - **歌词字号增大**：[ExpandedView.tsx](src/components/ExpandedView.tsx) ACTIVE_FONT_SIZE 26→**30**，IDLE_FONT_SIZE 19→**22**，提升可读性。
  - **高亮增强**：ACTIVE_SCALE 1.04→**1.06**；SUNG_COLOR 0.95→**1.0**（已唱全亮）；UNSUNG_COLOR 0.40→**0.38**（未唱更暗，对比更强）；OPACITY_SUNG 0.5→**0.45**，OPACITY_IDLE 0.55→**0.50**（已唱/未唱区分更明显）；active 行 textShadow 改为 **8px 白色 glow + 2px 6px 深色阴影**双层（关键信息更醒目），非 active 行用淡 glow + 1px 3px 深色阴影。
  - **歌单区域扩展**：左侧 flex 32%→**34%**，右侧 68%→**66%**，给歌单更多空间。
  - **歌单圆润曲率**：QUEUE_RADIUS 12→**18**（更大曲率更圆润），QUEUE_GAP 6→**8**（适度间隔增强呼吸感）。
  - typecheck + lint exit 0。未提交。renderer 改动 HMR 生效。

- 2026-09-08：歌词板块左侧 AI 对话整合。需求：进入歌词板块后移除左侧唱片机区块，原位替换为 AI 对话框。改动：src/components/ExpandedView.tsx 删除 CoverArt/歌曲信息/PlayerControls 区块，接入已有 AIChatBox（songTitle/songArtist/queue 传当前歌曲上下文，左侧保留歌单与 ControlPanel，AI 对话区与歌单区 flex-1 平分）；src/styles/index.css 补 @keyframes ai-bounce（思考三点弹跳）。过渡沿用现有 GSAP transform/opacity 入场（GPU 友好，≥60fps 无新增布局动画），AIChatBox memo 化避免歌词逐字更新重渲染。AI 回复为本地 mock，无新依赖/无外部 API。lint + typecheck exit 0。未提交。
 
 - 2026-09-08：调小歌词扩展页各容器圆角。涉及 src/components/ExpandedView.tsx（CARD_RADIUS 28→22、QUEUE_RADIUS 10→7）与 src/components/AIChatBox.tsx（CHAT_RADIUS 16→12、BUBBLE_RADIUS 14→10、输入框 rounded-2xl→rounded-xl）。lint + typecheck exit 0。未提交。
 ---
 
 ## 2026-07-15 今日总结

今日完成 9 轮迭代（续1-续9），主要围绕展开态歌词视图的视觉与交互打磨：

**核心功能实现**：
1. **歌词系统**（续1-续2）：网易云歌词接入 + 卡拉OK逐字填充 + LRC/YRC 解析 + LyricFrame 长条单行点击展开双区域沉浸视图
2. **音频 bug 修复**（续3-续4）：暂停续播从头开始 + 暂停后播放出现两个音频实例重叠（先 seek 再 play + unload 旧实例）
3. **mineradio 深度模仿**（续5）：滚轮接管平滑滚动 + smoothScrollToItem 自动定位 + iOS 风格毛玻璃 ControlPanel（CSS 变量实时调节模糊度/清晰度/歌词大小/律动强度）
4. **UI 视觉优化**（续6-续9）：按钮 glass variant（续6，续7 撤回）+ 歌单纯图片网格 + 简约风格（去蓝色虚线）+ 呼吸律动幅度调优 + 歌词字号增大 + 高亮增强 + 歌单区域扩展 + 圆润曲率

**今日改动文件**（按修改顺序）：
- [src/components/CoverArt.tsx](src/components/CoverArt.tsx) — 呼吸律动动效（scale 1.022 + 2.4s sine.inOut yoyo）
- [src/components/PlayerControls.tsx](src/components/PlayerControls.tsx) — 加 variant prop（续6）→ 撤回（续7）
- [src/components/ExpandedView.tsx](src/components/ExpandedView.tsx) — 双区域沉浸视图 + 歌单网格 + 歌词字号/高亮 + 布局扩展
- [src/components/LyricFrame.tsx](src/components/LyricFrame.tsx) — 长条单行歌词框 + 卡拉OK渐变填充
- [src/components/ControlPanel.tsx](src/components/ControlPanel.tsx) — iOS 风格毛玻璃控制面板（CSS 变量驱动）
- [src/styles/index.css](src/styles/index.css) — `@keyframes pulse` 当前播放指示点动画
- [src/hooks/usePlayer.ts](src/hooks/usePlayer.ts) — 暂停续播 + 音频实例管理修复
- [src/utils/lyricParser.ts](src/utils/lyricParser.ts) — LRC/YRC 歌词解析器
- [electron/main/music/netease.ts](electron/main/music/netease.ts) — 网易云歌词 IPC

**当前状态**：lint + typecheck exit 0，7-27 进度条工作已撤回，当前仅存 7-16 SearchPopup 未提交改动。

- 2026-07-15（提交）：git commit `28ecc21` "feat: 在线音乐源接入 + 歌词系统 + 播放界面 + 视觉打磨"（51 files, 6104 insertions, 1259 deletions），working tree clean。

- 2026-07-16：搜索模块弹窗化重构 + iOS 液态玻璃统一 + 拖动修复 + CoverArt 去呼吸 + 歌词抖动（方案A Web Audio 失败后撤回）。改动 7 文件：
  - [src/components/SearchPopup.tsx](src/components/SearchPopup.tsx) — 新建：弹窗式搜索窗口，替代页面跳转。iOS 26 液态玻璃 5 层结构（SVG feTurbulence + feDisplacementMap scale 77 + backdrop blur 6px saturate 160% + 表面张力 inset 高光 + 色调 rgba(255,255,255,0.06) + 顶部光反射带）。功能：实时搜索建议（300ms debounce）、热门搜索 12 词条、搜索历史（localStorage 最多 10 条）、多选批量添加、背景穿透毛玻璃、iOS 弹性出入动画（power3.out 0.4s / power2.in 0.25s）、自由拖动（仅左键 + 200ms 长按阈值 + 文档级 pointermove/up 防丢失 + RAF 节流 + ink 蓝发光边框拖动反馈）、关闭机制（外部点击/ESC/关闭按钮）
  - [src/App.tsx](src/App.tsx) — 移除 SearchView 页面跳转逻辑（`view` state），改为 `searchOpen` + SearchPopup 弹窗 overlay + `handleBatchAddSongs` 批量添加
  - [src/components/ControlPanel.tsx](src/components/ControlPanel.tsx) — 齿轮按钮 + 面板卡片同步为 iOS 液态玻璃 5 层结构（复用 `#liquid-glass-popup` 滤镜）。全部文字改为灰色系。齿轮未触发时灰色、触发后 ink 蓝
  - [src/components/CoverArt.tsx](src/components/CoverArt.tsx) — 取消播放呼吸动画（`breathe` ref + `scale:1.003` yoyo），保留唱片光盘 `vinyl-disc` 旋转
  - [src/hooks/usePlayer.ts](src/hooks/usePlayer.ts) — 先尝试接入 Web Audio API AnalyserNode 做歌词实时低频抖动驱动，但 `createMediaElementSource` 劫持 audio 输出导致 Electron 中无声。经 3 次迭代修复未果，最终完全移除所有 Web Audio 代码，恢复 howler 原生纯净播放器
  - [src/components/LyricFrame.tsx](src/components/LyricFrame.tsx) — 先加入 GSAP quickTo + 独立 RAF 双频正弦抖动，后随方案 A 撤回一并移除
  - [src/styles/index.css](src/styles/index.css) — 新增 `.popup-scroll` 白色半透明滚动条样式

  - 本日 lint + typecheck 始终 exit 0。未提交。

- 2026-07-27：展开态歌单滚动 + 剪切修复（ExpandedView）+ 进度条组件方案 A 开发 → 撤回。
  - 改动涉及文件：[ExpandedView.tsx](src/components/ExpandedView.tsx)、[usePlayer.ts](src/hooks/usePlayer.ts)、[MainView.tsx](src/views/MainView.tsx)、新建 [ProgressBar.tsx](src/components/ProgressBar.tsx)
  - V1-V5 共 5 轮迭代，最终因手动 seek 后进度条不跟随播放继续推进的 bug 未解决 + 用户对视觉质感不满意，全量撤回至 `28ecc21` 状态。
  - **保留经验**：ExpandedView 歌单区需补 `h-full` 才能滚动、激活项 scale(1.03) 需四边 padding 防裁切、GSAP transform 对齐用 `yPercent:-50` 而非 margin、`overwrite:true` 会杀 quickTo tween、进度条与 quickTo 争抢 x 属性会导致冻结。

## 未完成 / 待确认事项

### 待提交
- [ ] 7-16 SearchPopup 批次待提交（搜索弹窗 + iOS 液态玻璃统一 + CoverArt 去呼吸 + 全部调和微调）
- [ ] 7-15 及之前所有改动已在 `28ecc21` 提交
- 注意：项目根目录的 `插件/` 未跟踪文件夹与本项目无关（2026-08-30 用户明确），提交时必须排除，也不要修改它

### 在线音乐源接入进度（参照 Mineradio-main，方案文档 .trae/documents/网易云QQ双源音乐接入方案.md）
- [x] Step 1：主进程骨架 + 依赖 ✅ 2026-07-07
- [x] Step 2：网易云搜索跑通 ✅ 2026-07-07
- [x] Step 3：网易云播放跑通 ✅ 2026-07-09（const enum 坑用 `as never` 绕过；待用户实际测试搜索→点歌→播放链路）
- [x] Step 4：网易云歌词 ✅ 2026-07-15（getNeteaseLyric + music:lyric IPC + 渲染层 lyricParser + LyricFrame 卡拉OK逐字填充；待用户实测）
- [ ] Step 5：网易云扫码登录（qr_create + qr_check + cookie 写入 + 登录 UI）
- [ ] Step 6：QQ 搜索 + 播放 + 歌词（高风险，fetch musicu.fcg vkey 接口）
- [ ] Step 7：QQ 登录 webview
- [ ] Step 8：UI 美化（搜索页/登录页统一玻璃质感）

### 底部菜单栏改造进度
- [x] 移除顶部搜索按钮 + 加号入口 + queue 动态渲染 + 加号参与动效 + 空状态占位 ✅ 2026-07-09
- [ ] 待用户实际验证：空状态加号居中、点加号弹搜索页、搜索选歌追加并播放、加号与歌曲图标一同放大、满 12 首提示
- [ ] 待确认：加号位置当前为「跟随队列末尾」（空状态居中、满载第 13 位），如需严格固定第 13 位需改 track 为 13 等宽槽位

### 搜索模块改造进度
- [x] 页面跳转 → 弹窗模式 ✅ 2026-07-16
- [x] iOS 液态玻璃 5 层结构 ✅ 2026-07-16
- [x] 实时搜索建议 + 热门搜索 + 搜索历史 ✅ 2026-07-16
- [x] 多选批量添加 ✅ 2026-07-16
- [x] 可拖动（200ms 长按 + 文档级事件 + 视觉反馈）✅ 2026-07-16
- [x] 关闭机制（外部点击 / ESC / 关闭按钮）✅ 2026-07-16
- [x] SearchView 已死代码（可后续清理）

### ExpandedView 优化（7-27 已撤回，经验保留）
- [ ] 歌单区滚轮浏览（需补 `h-full` 让 `overflow-y-auto` 生效）
- [ ] 激活项 scale(1.03) 四边 padding 防裁切（ACTIVE_PAD=3px）
- [ ] 触摸设备滑动（`touchAction: pan-y` + `-webkit-overflow-scrolling: touch`）
- [ ] 滚动条拖动同步（`scroll` 事件同步 `wheelTargetTop`）
- [ ] 进度条组件（待重新设计，避免 quickTo + overwrite 冲突）

### 歌词动效框进度（LyricFrame）
- [x] 液态玻璃质感（SVG feTurbulence + feDisplacementMap 折射 + 表面张力 inset 高光 + 顶部光反射带）✅ 2026-07-10
- [x] 专辑封面图背景填充（object-cover + scale 1.06 + 双层交叉淡化切歌）✅ 2026-07-10
- [x] 无歌隐藏 / 有歌自动淡入弹出（active=player.list.length>0）✅ 2026-07-11
- [x] 首张图加载完成后才淡入（img.onLoad 修复空白问题）✅ 2026-07-11
- [x] 移除拖动功能，固定居中 ✅ 2026-07-11
- [x] 歌词数据接入（网易云 Step 4：getNeteaseLyric + IPC + lyricParser 解析 LRC/YRC）✅ 2026-07-15
- [x] 卡拉OK逐字填充（yrc 模式每音节独立 background-clip:text 渐变；lrc 模式整行渐变）✅ 2026-07-15
- [x] 行切换弹性入场（GSAP elastic.out(1, 0.55) autoAlpha + y + scale）✅ 2026-07-15
- [x] usePlayer 加 currentTime（RAF 60fps）+ seek + 自动加载歌词 ✅ 2026-07-15
- [ ] 待用户实际验证：搜索→点歌→歌词加载→逐字填充→切歌切词动效
- [ ] PlayerControls 液态玻璃化：已尝试但被用户撤回，播放控件保持原 bg-surface 样式（如需重新做需用户确认）
- [x] 长条单行 + 点击展开双区域沉浸视图 ✅ 2026-07-15（LyricFrame 改回 56px 长条单行 + onClick 展开 + hover 反馈 + 滚轮切歌；新建 ExpandedView 双区域布局：左 35% 控制区 + 右 65% 歌词区，iOS 大曲率圆角 32px，竖列 Timeline；待用户实测）

### 毛玻璃统一进度
- [x] SearchPopup → iOS 液态玻璃 5 层 ✅ 2026-07-16
- [x] ControlPanel 齿轮按钮 + 面板 → iOS 液态玻璃 ✅ 2026-07-16
- [x] 文字灰度化 ✅ 2026-07-16
- [x] 齿轮蓝色触发态 ✅ 2026-07-16

### 专辑唱片模块
- [x] 取消呼吸抖动动画 ✅ 2026-07-16
- [x] 保留唱片旋转 ✅ 2026-07-16

### 歌词框动效
- [x] 方案 A（Web Audio FFT → GSAP quickTo 抖动）已实施，但因 `createMediaElementSource` 导致 Electron 无声，经 3 次修复未果，最终撤回 ✅ 2026-07-16
- [ ] 歌词抖动特效待后续重新评估（需找不劫持 audio 输出的方案，或使用 howler Web Audio 模式）

### 核心体验未实现（项目定位要求）
- [x] 搜索功能 ✅ 2026-07-07（网易云搜索；QQ 待 Step 6）
- [ ] 歌单功能（创建/编辑/收藏歌单，歌单管理 UI）
- [x] 歌词功能 ✅ 2026-07-15（网易云歌词）
- [ ] 3D 歌单架（react-three-fiber 场景，唱片架浏览/选择）
- [ ] 切换唱片动效（切歌时唱片滑入滑出/翻转/淡入淡出动效）
- [x] 在线音乐源 API 选型与确认 ✅ 2026-07-07 用户拍板网易云 + QQ 双源（含逆向，放行红线）
- [x] 本地音乐扫描 → 改为纯在线（不做本地） ✅ 2026-07-07 用户决定

### 已完成需后续优化
- [x] git init + 初始提交 ✅ 2026-07-05（899e8f6）
- [x] 第二次提交 `28ecc21` ✅ 2026-07-15
- [x] electron-vite 项目脚手架 ✅ 2026-07-02
- [x] 透明窗口 + frameless + acrylic ✅ 2026-07-02
- [x] 自定义标题栏 + 全屏交互 ✅ 2026-07-05
- [x] 主播放界面（Allagi 风格）✅ 2026-07-05
- [x] Timeline magnify 鱼眼动效 ✅ 2026-07-05
- [x] 启动页 Pinyon Script 字体 ✅ 2026-07-05

### 待调优
- [ ] 启动页动效细节微调
- [ ] 占位封面替换为真封面
- [ ] 主播放界面视觉/交互细节
- [ ] 播放器功能补全（进度条、时长显示、音量、播放模式）
- [ ] ExpandedView 歌单滚动浏览（需补 `h-full` 等 7-27 经验）

## 每次任务完成后的固定动作
1. 更新本文件「已完成工作日志」。
2. 有新问题就更新「未完成事项」。
3. 跑一次语法检查（npm run lint && npm run typecheck）。
4. 最后说明 git status。
