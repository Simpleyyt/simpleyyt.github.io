---
title: '我也复刻了一个 Manus，带高仿 WebUI 和沙盒'
layout: post
tags:
  - llm
  - agent
  - manus
category: AI
---

最近在学习 LLM Agent，但终觉“纸上学来终觉浅，绝知此事要躬行”，所以想写个小项目试试手。  
在这个人均写一个 Manus 的时代，在这个半开卷、且 Manus 提词已经泄漏的情况下，我整理了一下自己的目标：

- 可单主机云上部署，方便直接部署在家里的 Server 上。
- 集成浏览器、Shell、Python、Node 等工具及 Ubuntu 沙盒环境，每一个任务分配一个沙盒。
- Web UI 与提示词直接借鉴官方 Manus。

结合 Cursor，理论上可以快速写出一个 Manus 示例，听说 OpenManus 4 小时就写出来了。

项目地址：<https://github.com/Simpleyyt/ai-manus>

<!--more-->

# Demo 演示

## Code Use

> Prompt：写一个复杂的 Python 示例

<video controls playsinline width="100%" src="https://github.com/user-attachments/assets/5cb2240b-0984-4db0-8818-a24f81624b04"></video>

若无法直接播放，可访问：<https://github.com/user-attachments/assets/5cb2240b-0984-4db0-8818-a24f81624b04>

## Browser Use

> Prompt：任务：LLM 最新论文

<video controls playsinline width="100%" src="https://github.com/user-attachments/assets/8f7788a4-fbda-49f5-b836-949a607c64ac"></video>

若无法直接播放，可访问：<https://github.com/user-attachments/assets/8f7788a4-fbda-49f5-b836-949a607c64ac>

效果可以说是相当凑合，但是目前主要是学习目的，提示词与 Agent 流程都还有继续优化空间，懒得再调了，就交给广大网友了。

# 整体设计

> 原文此处包含系统整体架构图（本次同步未包含原始图片文件）。

整体系统由三个模块组成：Web、Server 与 Sandbox，用户使用流程如下。

当用户发起对话时：

1. Web 向 Server 发送创建 Agent 请求，Server 通过 `/var/run/docker.sock` 创建出 Sandbox，并返回会话 ID。
2. Sandbox 是一个 Ubuntu Docker 环境，里面会启动 Chrome 浏览器及 File/Shell 等工具 API 服务。
3. Web 往会话 ID 中发送用户消息，Server 收到后将消息发送给 PlanAct Agent 处理。
4. PlanAct Agent 在处理过程中调用相关工具完成任务。
5. Agent 处理过程中的所有事件通过 SSE 发回 Web。

当用户浏览工具时：

- 浏览器：
  1. Sandbox 的无头浏览器通过 xvfb 与 x11vnc 启动 VNC 服务，并通过 websockify 将 VNC 转成 WebSocket。
  2. Web 的 NoVNC 组件通过 Server 的 WebSocket Forward 转发到 Sandbox，实现浏览器查看。
- 其它工具：原理类似。

# AI Agent 设计模式

AI Agent 是什么？相信大家都听烂了。我在这里说得肯定不如别人好，简单来说就是：

**AI Agent = LLM + Planning + Memory + Tools**

> 原文此处包含 AI Agent 核心特征示意图（本次同步未包含原始图片文件）。

## FunctionCall or ReAct or LangChain?

先说 Tool Use 部分，目前常见方式是：

1. 高阶模型自身 FunctionCall；
2. ReAct Prompt 框架；
3. LangChain Agent 框架（本质上是前两者的高度封装）。

最简单是 LangChain，但对学习项目来说，我更想弄清楚底层发生了什么，所以先不走这条路。

FunctionCall 对模型能力要求高，ReAct 的设计又较繁琐，思来想去，还是先用 FunctionCall，先用好模型把系统跑通，后面再继续研究 ReAct。  
关于 ReAct，可参考：[ReAct 框架 | Prompt Engineering Guide](https://www.promptingguide.ai/zh/techniques/react)。

> 原文此处包含 ReAct 示例图（本次同步未包含原始图片文件）。

因此，该项目对 LLM 的要求如下：

1. 兼容 OpenAI 接口
2. 支持 FunctionCall
3. 支持 JSON 格式输出（因为抛弃了 LangChain 又想省事）

## Plan-and-Act Agent 设计模式

整体使用 Plan-and-Act 的 Agent 设计模式，相关论文：  
[Plan-and-Act: Improving Planning of Agents for Long-Horizon Tasks](https://arxiv.org/html/2503.09572v1)

> 原文此处包含 Plan-and-Act 流程图（本次同步未包含原始图片文件）。

即将系统分成 Planner 和 Executor，Planner 将任务进行规划拆分，Executor 负责任务分步执行，再将执行结果返回给 Planner 做重新规划。

项目中的状态流转如下：

> 原文此处包含状态流转图（本次同步未包含原始图片文件）。

系统支持被打断，所有打断消息都会流向 Planner Agent，Planner 会根据打断消息重新规划。

# Sandbox 设计

为了实现每个任务使用独立 Docker 沙盒，Server 通过 `/var/run/docker.sock` 在宿主机上创建与销毁沙盒。

## Sandbox 进程与生命周期管理

整个 Sandbox 通过 supervisord 进行进程管理，并通过 supervisord 实现 Sandbox TTL。  
当前 Agent 还没有主动销毁机制，所以需要 Sandbox 自动过期自销毁，并支持续时等接口。

## File & Shell 工具

文件操作与 Shell 命令执行没有太大难度。Cursor 在这块很强，我把 Manus 的工具描述丢给 Cursor 后，很快就用 FastAPI 生成了一整套代码，稳定性还不错，基本没怎么改。

## Browser 工具

目前很多复刻 Manus 的项目都在用 browser-use（<https://github.com/browser-use/browser-use>）。  
为了学习和研究，我还是决定用 Playwright + Chrome 自己做一版。由于暂时没有接入视觉模型能力，所以先以文字模型为基础操作浏览器。

为了让 Sandbox 更纯粹，Sandbox 只启动 Chrome，并暴露 CDP 与 VNC 供 Server 使用。

### 启动 Browser

坑点一：启动参数

Chrome 启动参数很多，遇到问题再现找参数比较耗时。直接参考 browser-use 的参数配置：  
<https://github.com/browser-use/browser-use/blob/main/browser_use/browser/chrome.py>

坑点二：CDP 监听地址不支持 `0.0.0.0`

新版本 Chrome 似乎已不支持 `--remote-debugging-address`（参考：<https://issues.chromium.org/issues/327558594>）。  
可通过端口转发绕过：

```bash
# 假设 CDP 监听在 127.0.0.1:8222
socat TCP-LISTEN:9222,bind=0.0.0.0,fork,reuseaddr TCP:127.0.0.1:8222
```

坑点三：CDP 地址不能通过域名访问

即 HTTP Header 的 Host 字段只能是 IP 或 localhost。  
在反向代理中替换相关字段即可。

### VNC 访问

由于 Docker 镜像内没有 X Server 等图形环境，所以通过虚拟 X11 显示服务器 `Xvfb` 给 Chrome 绘制窗口，并通过 `x11vnc` 提供 VNC Server：

```bash
# 启动 Xvfb 在 Display :1
Xvfb :1 -screen 0 1280x1029x24

# Chrome 浏览器指定 Display
google-chrome \
    --display=:1 \
    ...

# 启动 VNC 服务
x11vnc -display :1 -nopw -listen 0.0.0.0 -xkb -forever -rfbport 5900
```

由于 VNC 的四层端口不便于反向代理转发，所以再用 `websockify` 将 VNC 转为七层 `WebSocket`：

```bash
# 暴露 5901 端口 WebSocket 服务
websockify 0.0.0.0:5901 localhost:5900
```

便于后续 `NoVNC` 连接。

### AI 网页元素操作与信息提取

一开始尝试把整个 HTML 丢给大模型，结果很快爆 Token。  
调研后发现主流做法基本是两步：

1. 可交互元素提取
2. 网页信息提取

先提取可见、可交互元素，让模型识别哪些可输入、可点击。  
一般会整理成 `index <tag>text</tag>` 形式，例如：

```bash
1 <input>手机号</input>
2 <input>密码</input>
3 <button>确认</button>
...
```

并给原标签打上 ID。下面是 Cursor 生成、且已验证可用的一段示例代码：

```typescript
const interactiveElements = [];
const viewportHeight = window.innerHeight;
const viewportWidth = window.innerWidth;

const elements = document.querySelectorAll(
  'button, a, input, textarea, select, [role="button"], [tabindex]:not([tabindex="-1"])'
);

let validElementIndex = 0;

for (let i = 0; i < elements.length; i++) {
  const element = elements[i];
  const rect = element.getBoundingClientRect();

  if (rect.width === 0 || rect.height === 0) continue;
  if (rect.bottom < 0 || rect.top > viewportHeight || rect.right < 0 || rect.left > viewportWidth) continue;

  const style = window.getComputedStyle(element);
  if (style.display === 'none' || style.visibility === 'hidden' || style.opacity === '0') continue;

  let tagName = element.tagName.toLowerCase();
  let text = '';

  if (element.value && ['input', 'textarea', 'select'].includes(tagName)) {
    text = element.value;

    if (tagName === 'input') {
      let labelText = '';
      if (element.id) {
        const label = document.querySelector(`label[for="${element.id}"]`);
        if (label) labelText = label.innerText.trim();
      }

      if (!labelText) {
        const parentLabel = element.closest('label');
        if (parentLabel) labelText = parentLabel.innerText.trim().replace(element.value, '').trim();
      }

      if (labelText) text = `[Label: ${labelText}] ${text}`;
      if (element.placeholder) text = `${text} [Placeholder: ${element.placeholder}]`;
    }
  } else if (element.innerText) {
    text = element.innerText.trim().replace(/\s+/g, ' ');
  } else if (element.alt) {
    text = element.alt;
  } else if (element.title) {
    text = element.title;
  } else if (element.placeholder) {
    text = `[Placeholder: ${element.placeholder}]`;
  } else if (element.type) {
    text = `[${element.type}]`;

    if (tagName === 'input') {
      let labelText = '';
      if (element.id) {
        const label = document.querySelector(`label[for="${element.id}"]`);
        if (label) labelText = label.innerText.trim();
      }

      if (!labelText) {
        const parentLabel = element.closest('label');
        if (parentLabel) labelText = parentLabel.innerText.trim();
      }

      if (labelText) text = `[Label: ${labelText}] ${text}`;
      if (element.placeholder) text = `${text} [Placeholder: ${element.placeholder}]`;
    }
  } else {
    text = '[No text]';
  }

  if (text.length > 100) text = text.substring(0, 97) + '...';

  element.setAttribute('data-manus-id', `manus-element-${validElementIndex}`);
  const selector = `[data-manus-id="manus-element-${validElementIndex}"]`;

  interactiveElements.push({
    index: validElementIndex,
    tag: tagName,
    text: text,
    selector: selector,
  });

  validElementIndex++;
}

return interactiveElements;
```

这样模型就可以按 ID 操作元素。

还需要做网页信息提取。主流方式是先去掉不可见元素，再转 Markdown 后交给大模型抽取，从而节省 Token，例如：

```python
# Convert to Markdown
markdown_content = markdownify(visible_content)

max_content_length = min(50000, len(markdown_content))
response = await self.llm.ask(
    [
        {
            "role": "system",
            "content": "You are a professional web page information extraction assistant. Please extract all information from the current page content and convert it to Markdown format.",
        },
        {
            "role": "user",
            "content": markdown_content[:max_content_length],
        },
    ]
)
```

至此，大模型就可以与网页交互并阅读网页信息了。

# Web UI 设计

Web UI 本来是我的软肋，但正好是 Cursor 的强项。结合对正版 Manus 的借鉴，页面虽然简单，但也能做得七七八八。

# 如何部署？

## 环境要求

本项目主要依赖 Docker 进行开发与部署，建议安装较新版本 Docker：

- Docker 20.10+
- Docker Compose

模型能力要求也比较高：

- 兼容 OpenAI 接口
- 支持 FunctionCall
- 支持 JSON Format 输出

推荐 Deepseek 与 ChatGPT。

## 部署

推荐使用 Docker Compose 部署：

```yaml
services:
  frontend:
    image: hub.byted.org/byvirt/manus-frontend
    ports:
      - "5173:80"
    depends_on:
      - backend
    restart: unless-stopped
    networks:
      - manus-network
    environment:
      - BACKEND_URL=http://backend:8000

  backend:
    image: hub.byted.org/byvirt/manus-backend
    depends_on:
      - sandbox
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      - manus-network
    environment:
      - API_BASE=https://api.openai.com/v1
      - API_KEY=sk-xxxx
      - MODEL_NAME=gpt-4o
      - TEMPERATURE=0.7
      - MAX_TOKENS=2000
      #- GOOGLE_SEARCH_API_KEY=
      #- GOOGLE_SEARCH_ENGINE_ID=
      - LOG_LEVEL=INFO
      - SANDBOX_IMAGE=hub.byted.org/byvirt/manus-sandbox
      - SANDBOX_NAME_PREFIX=sandbox
      - SANDBOX_TTL_MINUTES=30
      - SANDBOX_NETWORK=manus-network

  sandbox:
    image: hub.byted.org/byvirt/manus-sandbox
    command: /bin/sh -c "exit 0"  # prevent sandbox from starting, ensure image is pulled
    restart: "no"
    networks:
      - manus-network

networks:
  manus-network:
    name: manus-network
    driver: bridge
```

# 如何开发？

## 环境准备

环境要求同部署章节。

下载项目：

```bash
git clone https://github.com/Simpleyyt/ai-manus.git
cd ai-manus
```

复制配置文件：

```bash
cp .env.example .env
```

修改配置文件：

```bash
# Model provider configuration
API_KEY=
API_BASE=https://api.openai.com/v1

# Model configuration
MODEL_NAME=gpt-4o
TEMPERATURE=0.7
MAX_TOKENS=2000

# Optional: Google search configuration
#GOOGLE_SEARCH_API_KEY=
#GOOGLE_SEARCH_ENGINE_ID=

# Sandbox configuration
SANDBOX_IMAGE=simpleyyt/sandbox
SANDBOX_NAME_PREFIX=sandbox
SANDBOX_TTL_MINUTES=30
SANDBOX_NETWORK=manus-network

# Log configuration
LOG_LEVEL=INFO
```

## 开发

> 开发模式下只会全局启动一个沙盒。

运行调试：

```bash
# 相当于 docker compose -f docker-compose-development.yaml up
./dev.sh up
```

Web、Sandbox、Server 都会以 reload 模式运行，代码改动会自动 reload。暴露端口如下：

- **5173**：Web 前端端口
- **8000**：Server API 服务端口
- **8080**：Sandbox API 服务端口
- **5900**：Sandbox VNC 端口
- **9222**：Sandbox Chrome 浏览器 CDP 端口

当依赖变化（如 `requirements.txt` 或 `package.json`）时，可以清理并重建：

```bash
./dev.sh down -v
./dev.sh build
./dev.sh up
```

## 发布

```bash
export IMAGE_REGISTRY=xxxx
export IMAGE_TAG=latest

# 构建镜像
./run build

# 推送到镜像仓库
./run push
```

# 写在最后

本项目主要用于学习与研究，欢迎一起交流和改进，也算为代码工程师向“提词工程师”跃迁做一点准备。
