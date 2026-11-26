# KOBE AIRWAYS · 坠机航班

一个可以直接在浏览器里玩的 3D 叙事小游戏，单文件构建，无外部依赖。

## 在线游玩

https://cubernie.github.io/groupwork-1/

打开即玩，无需安装。首次进入会加载 3.6 MB 左右的内嵌资源。

## 操作

| 操作 | 按键 |
| --- | --- |
| 移动 | `W` `A` `S` `D` |
| 环视 | 移动鼠标（点击画面锁定指针） |
| 交互 / 跳跃 | `Space`、`E`、`F` |
| 奔跑 | `Shift` |
| 释放鼠标 | `Esc` |

游戏内的解谜键位（例如 `X` 慢放时间、`K` 连线、`Q` 等）会在对应剧情里以键帽提示显示，跟着屏幕提示走即可。

## 兼容性

- ✅ 桌面浏览器（Chrome / Edge / Firefox / Safari）
- ⚠️ 手机浏览器可以打开，但**没有触屏操作**，需要键盘和鼠标

## 技术说明

- `index.html`：完整游戏，three.js r160 与全部模块以字符串内嵌，通过 Blob URL + import map 加载
- `kobe-airways.html`：旧链接的跳转页
- `.nojekyll`：关闭 GitHub Pages 的 Jekyll 处理，避免大文件被改写

## 本地运行

直接双击 `index.html` 即可；若浏览器限制本地模块加载，可在目录下起一个静态服务器：

```bash
python -m http.server 8000
```

然后访问 http://localhost:8000/ 。

## CI/CD 探索记录

一次真实部署的完整观察，数据来自 GitHub REST API（`actions/runs`、`jobs`、`deployments`、`pages/builds`）。

### 1. 被检查的部署与源提交

| 项目 | 值 |
| --- | --- |
| 工作流 | `pages build and deployment`（`dynamic/pages/pages-build-deployment`） |
| Run ID | `45782213805` |
| 事件类型 | `dynamic`（Pages 内置触发器，非 `push` 工作流） |
| 源提交 | `3e9d8a0c5e9f5cda72be8b26201111a729a1f90b`（`Serve game at site root and add Pages config`） |
| 部署记录 | `deployment 8080003673`，环境 `github-pages`，`ref=main` |
| 结论 | `success`，Pages build `built`，耗时 22.0 s |

### 2. 触发 → 步骤 → 结果

| 时间（UTC） | 环节 | 耗时 |
| --- | --- | --- |
| 08:19:39 | Pages 构建记录创建（源提交 `3e9d8a0`） | — |
| 08:19:40 | 工作流 run 被创建 | — |
| 08:19:46 → 08:19:53 | **job `build`**：`Set up job` → `Checkout` → `Upload artifact` → `Post Checkout` → `Complete job` | 7 s |
| 08:19:55 → 08:20:00 | **job `report-build-status`**：回写构建状态 | 5 s |
| 08:19:57 → 08:20:05 | **job `deploy`**：`Deploy to GitHub Pages` | 8 s |
| 08:20:06 | run 结束，全部 `success` | 总计约 26 s |

三个 job 并行编排：`build` 产出 artifact 后 `deploy` 才开始，`report-build-status` 独立回写状态。

**发布内容**：`.nojekyll` 使其跳过 Jekyll，仓库根目录的 4 个文件（`index.html` 3.87 MB、`kobe-airways.html`、`README.md`、`.gitignore`）原样作为静态文件发布。

**部署结果**：`https://cubernie.github.io/groupwork-1/` 返回 200，实际下载 3,874,078 字节，与本地 `index.html` 完全一致；`<title>` 与 `id="mods"` 标记均在，说明 3.9 MB 单文件完整发布。

#### 第二次部署（本文档提交，用于独立观察）

| 项目 | 值 |
| --- | --- |
| Run ID | `45783166640` |
| 源提交 | `1d90760`（`Document CI/CD exploration`） |
| 部署记录 | `deployment 8080135118`，环境 `github-pages` |
| `build` job | 08:26:29 → 08:26:35（6 s），同样是 `Set up job` / `Checkout` / `Upload artifact` / `Post Checkout` / `Complete job` |
| `report-build-status` | 08:26:37 → 08:26:41（4 s） |
| `deploy` job | 08:26:39 → 08:26:49（10 s） |
| Pages build | `built`，19.2 s |
| 总耗时 | 08:26:26 → 08:26:50，约 24 s |

两次部署的步骤集合完全一致，说明这条流水线是确定性的：同样的输入永远走同样的五个步骤，**两次都没有任何内容检查**。线上 `README.md` 已包含本节，说明本次发布同样生效。

### 3. 哪些自动，哪些手动

| 自动 | 手动 |
| --- | --- |
| push 到 `main` 即触发构建 | 首次开启 Pages（本仓库是调 `POST /repos/{owner}/{repo}/pages` 完成的，没有走 Settings 界面） |
| checkout、打包 artifact、上传 | 指定源分支与目录（`main` + `/`） |
| 部署到 Pages CDN、回写 build status | 是否强制 HTTPS |
| HTTPS 证书签发与续期 | 提交与 push 本身（人工动作） |

### 4. 跑了什么检查

**没有跑任何自定义检查。** `build` job 只有 checkout 和 upload artifact，没有 lint、单元测试、HTML 校验、链接检查或体积守卫。这个工作流是平台托管的模板，步骤无法在仓库里编辑，所以它天然不含项目级质量门禁。

唯一的质量信号来自平台层：artifact 上传成功、`deploy` 返回成功、HTTPS 可用。这些只能证明「文件传上去了」，不能证明「页面能正常玩」。

### 5. 一个有用的缺失检查

**入口冒烟检查（pre-deploy smoke test）**：部署前确认仓库根目录存在 `index.html`，且内容包含 `KOBE AIRWAYS` 与 `id="mods"`，否则让 job 失败。

理由：本仓库把 3.87 MB 的全部逻辑塞进一个文件，重命名、编码损坏、截断或误把跳转页当成首页，任何一种都会让站点「部署成功但打不开」。这个检查只要几秒，却能在发布前拦住最可能发生的故障——而当前的流水线对此完全无感。

实现方式：加一个 `.github/workflows/` 下的前置 workflow（或用 `workflow_run` 在 Pages 构建后校验线上响应），失败即阻断或告警。

## 许可

仓库根目录附 `LICENSE`（MIT License，1,067 字节），版权声明为 `Copyright (c) 2026 Fly_Butter`，覆盖本仓库自写的文档与构建内容

游戏本体（`index.html` 内的代码、剧情与美术资源）版权归原作者所有，仅用于课程小组作业展示，不在这份 MIT 许可的覆盖范围内
