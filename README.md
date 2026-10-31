# Open Source Canvas · 中文说明

> **让画布成为你的 AI 创作工作室。** 在一张无限画布中连接提示词、参考图、模型和输出，使用 ComfyUI 生成图片与视频。

**本仓库是 [raph651/open-source-canvas](https://github.com/raph651/open-source-canvas) 的 Fork。衷心感谢原作者 [@raph651](https://github.com/raph651) 的开源工作与设计。** 本仓库以中文使用说明为主，便于学习、部署和后续探索自托管 ComfyUI 工作流。项目原作者、上游代码及贡献归属不因 Fork 而改变。

- **原作者仓库：** [raph651/open-source-canvas](https://github.com/raph651/open-source-canvas)
- **本 Fork：** [xiaoqianran/open-source-canvas](https://github.com/xiaoqianran/open-source-canvas)
- **上游在线演示：** [raph651.github.io/open-source-canvas](https://raph651.github.io/open-source-canvas/)（仅演示画布，无法执行真实 ComfyUI 推理）
- **开源协议：** [MIT](LICENSE)；模型权重另有各自的授权限制

<p align="center">
  <a href="docs/media/demo-1080p-v3.mp4"><img src="docs/media/demo-sharp-v3.gif" alt="Open Source Canvas 上游项目演示" width="720"></a>
</p>

## 项目简介

Open Source Canvas 是一个基于 **React、Vite、React Flow、Zustand** 构建的开源图片与视频创作画布。画布本身不运行模型，而是通过 **ComfyUI 的 HTTP / WebSocket API** 提交任务、监听进度并获取输出。

主要功能包括：

- 无限画布：缩放、拖拽、多选、分组、撤销 / 重做、复制 / 粘贴；
- 提示词、图片、模型、输出、备注等节点，支持节点间连线；
- 图片到图片、图片到视频的串联创作；
- 提示词、参考图、分辨率、画幅、时长、种子等生成参数；
- 生成历史、版本比较、浏览器本地自动保存及 JSON 导入 / 导出；
- 可选的 WebSocket 协作房间、评论与浅色 / 深色主题。

![上游夜间工作室](docs/media/studio-night.jpg)

## 当前支持范围

| 类型 | 当前状态 |
| --- | --- |
| SD-Turbo 图片生成 | 上游已接入的本地工作流 |
| LTX-Video 2B 视频生成 | 上游已接入的本地工作流 |
| Wan 2.2 TI2V 5B、Pika H3 Flex | 上游模型目录可识别，**不等于已接通任意自定义工作流** |
| LTX-2.5、Seedream 等 ComfyUI Partner API 模型 | 上游展示模型与示例；部分生成尚需实现对应 wrapper |
| 自建 Qwen Image 2.1 / MiniMax H3 / Modal 多端点 | **后续适配方向；当前 Fork 不声称已原生支持** |

> 模型是否能运行取决于你的 ComfyUI 版本、已安装节点、权重文件和具体工作流。前端显示模型名称，不代表推理已可用。

## 快速开始

### 环境要求

- Node.js **20 或更新版本**（建议 22）及 npm；
- 一套可用的 ComfyUI 服务；推荐使用较新的 ComfyUI 版本；
- 对应模型所需的 GPU、内存与磁盘空间。

### 1. 获取本仓库

```bash
git clone https://github.com/xiaoqianran/open-source-canvas.git
cd open-source-canvas
npm install
```

### 2. 启动前端

开发模式：

```bash
npm run dev
```

构建并启动带 ComfyUI 代理的服务（推荐连接远程 ComfyUI 时使用）：

```bash
npm run build
node server/app.mjs
```

默认访问：**http://127.0.0.1:4173**。

### 3. 连接 ComfyUI

本地 ComfyUI 默认地址通常是 `http://127.0.0.1:8188`。若使用浏览器直接访问 ComfyUI API 的开发模式，可能需要在 ComfyUI 启动时添加 `--enable-cors-header`。

连接**远程 / Modal ComfyUI** 时，推荐通过本项目的服务端代理：

```bash
# 替换为你自己部署的 ComfyUI Web Endpoint
export COMFY_URL="https://your-comfyui-endpoint.modal.run"
npm run build
node server/app.mjs
```

Windows PowerShell：

```powershell
$env:COMFY_URL = "https://your-comfyui-endpoint.modal.run"
npm run build
node server/app.mjs
```

**注意：** `COMFY_URL` 填入可访问 `/prompt`、`/history` 等 API 的 **ComfyUI Web Endpoint**，不要填 Modal Dashboard 的管理页面链接。打开画布后，在顶部 **ComfyUI** 状态入口检查连接状态。

服务端常用环境变量：

| 变量 | 默认值 | 用途 |
| --- | --- | --- |
| `COMFY_URL` | `http://127.0.0.1:8188` | ComfyUI 后端地址 |
| `PORT` | `4173` | 画布服务端口 |
| `HOST` | `127.0.0.1` | 监听地址 |
| `COMFY_API_KEY` | 未设置 | 代理所连接网关要求的 API Key |
| `CF_ACCESS_CLIENT_ID` / `CF_ACCESS_CLIENT_SECRET` | 未设置 | Cloudflare Access 服务令牌 |

### 4. 开始创作

1. 打开示例画布或创建新画布。
2. 添加 **Prompt** 和 **Model** 节点并连线。
3. 设置提示词、模型及生成参数，点击 **Generate**。
4. 对支持的工作流，可将上游图片连接到视频模型，作为参考图或首帧。

## 接入自定义 ComfyUI 工作流

如果你希望使用自建 Qwen Image、MiniMax H3 或 Pika 工作流，需要在此基础上增加对应模型适配，而非只修改 Endpoint：

1. 准备能够运行的 **ComfyUI API 格式 JSON** 工作流；
2. 在 `src/lib/comfyWorkflows.ts` 实现工作流构建与参数映射；
3. 在 `src/lib/modelWrappers.ts` 配置模型的生成调用，并更新相应模型目录；
4. 使用 ComfyUI 的 `/prompt`、`/ws`、`/history/{prompt_id}` 和 `/view` 完成任务与结果闭环；
5. 多 ComfyUI Endpoint 间传图时，通过上传 / 对象存储中转，不能直接复用另一容器中的本地路径。

参考上游文档：[添加模型](docs/ADDING_MODELS.md) · [ComfyUI 运行机制](docs/COMFYUI_RUNTIME.md)。

## 项目结构

```text
src/
  components/         画布编辑器、节点及界面
  lib/                ComfyUI 客户端、工作流构建、模型路由
  store/              状态与协作
server/
  app.mjs             静态资源服务与 ComfyUI API 代理
  room.mjs            可选协作服务
workflows/            ComfyUI API 格式工作流
public/               示例媒体与静态资源
scripts/              本地安装及演示构建脚本
docs/                 工作流、模型接入与技术说明
deploy/               部署参考
CHANGELOG.md          更新记录
LICENSE               MIT 协议
```

## 常用命令

| 命令 | 作用 |
| --- | --- |
| `npm run dev` | 启动开发服务 |
| `npm run build` | 类型检查并构建前端 |
| `node server/app.mjs` | 启动生产式静态服务及 ComfyUI 代理 |
| `npm run preview` | 预览前端构建结果 |
| `npm run room` | 启动演示用的协作服务 |
| `npm run build:pages` | 构建 GitHub Pages 静态演示版（不连接真实推理） |

## 常见问题

**连接显示 Offline：** 确认 ComfyUI 已启动、Endpoint 可访问，并检查 `COMFY_URL` 与代理是否正确配置。开发模式的浏览器直连还可能遇到 CORS 限制。

**提示 `/prompt 400` 或找不到模型：** 核对工作流 JSON、节点定义以及 ComfyUI 实际可见的模型文件名。连接成功不代表任意工作流都能运行。

**视频生成显存不足：** 降低帧数 / 分辨率或选择更大显存的推理设备。

**多人共享：** 请为 ComfyUI Endpoint 和画布代理配置实际鉴权及资源限额。公开的 GPU 推理接口可能被他人调用并产生费用；`room.mjs` 是演示协作服务，不应直接当成带鉴权的生产系统。

## 致谢与开源声明

本项目基于 **[@raph651](https://github.com/raph651) 创作的 [Open Source Canvas](https://github.com/raph651/open-source-canvas)**。感谢原作者将无限画布、节点交互、ComfyUI 集成和图片 / 视频生成流程开源，让更多开发者可以学习、复用并继续改进。

我们尊重并保留上游项目的作者信息、许可证和版权声明。**本 Fork 与原作者项目并非官方合作或官方发行版本。** 若本仓库后续加入自定义适配，相关改动将与上游已有功能清晰区分。

项目代码遵循仓库中的 **[MIT License](LICENSE)**。**模型权重、LoRA、素材和第三方 API 不因本项目采用 MIT 而自动获得 MIT 授权**，请分别阅读对应模型卡和服务条款；尤其在商用或跨地区部署时核对许可要求。

欢迎访问并支持[上游原作者仓库](https://github.com/raph651/open-source-canvas)，向原作者反馈问题与贡献代码。
