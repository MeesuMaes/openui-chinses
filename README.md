# OpenUI

<p align="center">
  <img src="./assets/openui.png" width="150" alt="OpenUI" />
</p>

构建 UI 组件可能会很繁琐。OpenUI 的目标是让这个过程变得有趣、快速且灵活。我们还在 [W&B](https://wandb.com) 使用它来测试和原型化下一代工具，以便在 LLM 之上构建强大的应用程序。

## 概述

![Demo](./assets/demo.gif)

OpenUI 让你可以通过想象力描述 UI，然后实时查看渲染效果。你可以要求更改并将 HTML 转换为 React、Svelte、Web Components 等。它就像 [v0](https://v0.dev)，但开源且没有那么精致 :stuck_out_tongue_closed_eyes:。

## 在线演示

[尝试演示](https://openui.fly.dev)

## 本地运行

OpenUI 支持 [OpenAI](https://platform.openai.com/api-keys)、[Groq](https://console.groq.com/keys) 和任何 [LiteLLM](https://docs.litellm.ai/docs/) 支持的模型，例如 [Gemini](https://aistudio.google.com/app/apikey) 或 [Anthropic (Claude)](https://console.anthropic.com/settings/keys)。以下环境变量是可选的，但需要在环境中设置以支持替代模型：

- **OpenAI** `OPENAI_API_KEY`
- **Groq** `GROQ_API_KEY`
- **Gemini** `GEMINI_API_KEY`
- **Anthropic** `ANTHROPIC_API_KEY`
- **Cohere** `COHERE_API_KEY`
- **Mistral** `MISTRAL_API_KEY`
- **OpenAI 兼容** `OPENAI_COMPATIBLE_ENDPOINT` 和 `OPENAI_COMPATIBLE_API_KEY`

例如，如果你正在运行像 [localai](https://localai.io/) 这样的工具，你可以设置 `OPENAI_COMPATIBLE_ENDPOINT` 并可选地设置 `OPENAI_COMPATIBLE_API_KEY`，这样在 LiteLLM 下的 UI 模型选择器中就会列出可用的模型。

### Ollama

你也可以使用 [Ollama](https://ollama.com) 提供的模型。[安装 Ollama](https://ollama.com/download) 并拉取一个模型，如 [Llava](https://ollama.com/library/llava)。如果 Ollama 不在 http://127.0.0.1:11434 上运行，你可以将 `OLLAMA_HOST` 环境变量设置为你的 Ollama 实例的主机和端口。例如，在 Docker 中运行时，你需要指向 http://host.docker.internal:11434，如下所示。

### Docker（首选）

以下命令会从你的 shell 环境中转发指定的 API 密钥，并告诉 Docker 使用在你机器上运行的 Ollama 实例。

```bash
export ANTHROPIC_API_KEY=xxx
export OPENAI_API_KEY=xxx
docker run --rm --name openui -p 7878:7878 -e OPENAI_API_KEY -e ANTHROPIC_API_KEY -e OLLAMA_HOST=http://host.docker.internal:11434 ghcr.io/wandb/openui
```

现在你可以访问 [http://localhost:7878](http://localhost:7878) 并生成新的 UI！

### 从源代码 / Python

假设你已经安装了 git 和 [uv](https://github.com/astral-sh/uv)：

```bash
git clone https://github.com/wandb/openui
cd openui/backend
uv sync --frozen --extra litellm
source .venv/bin/activate
# 设置你想使用的任何 LLM 的 API 密钥
export OPENAI_API_KEY=xxx
python -m openui
```

## LiteLLM

[LiteLLM](https://docs.litellm.ai/docs/) 可用于连接到几乎所有可用的 LLM 服务。我们根据你的环境变量自动生成配置。你可以创建自己的 [代理配置](https://litellm.vercel.app/docs/proxy/configs) 来覆盖此行为。我们在以下位置查找自定义配置：

1. 当前目录中的 `litellm-config.yaml`
2. 在 Docker 容器中运行时的 `/app/litellm-config.yaml`
3. 由 `OPENUI_LITELLM_CONFIG` 环境变量指定的任意路径

例如，要在 Docker 中使用自定义配置，可以运行：

```bash
docker run -n openui -p 7878:7878 -v $(pwd)/litellm-config.yaml:/app/litellm-config.yaml ghcr.io/wandb/openui
```

要从源代码使用 litellm，你可以运行：

```bash
pip install .[litellm]
export ANTHROPIC_API_KEY=xxx
export OPENAI_COMPATIBLE_ENDPOINT=http://localhost:8080/v1
python -m openui --litellm
```

## Groq

要使用超快的 [Groq](https://groq.com) 模型，请将 `GROQ_API_KEY` 设置为你的 Groq api 密钥，可以在[这里找到](https://console.groq.com/keys)。要使用其中一个 Groq 模型，请点击导航栏中的设置图标。

### Docker Compose

> **免责声明：** 这可能会非常慢。如果你有 GPU，可能需要更改 `ollama` 容器的标签以支持它。如果你在 Mac 上运行，请按照上面的说明运行 Ollama 以利用 M1/M2。

从根目录运行：

```bash
docker-compose up -d
docker exec -it openui-ollama-1 ollama pull llava
```

如果你已经在环境中设置了 OPENAI_API_KEY，只需从 `OPENAI_API_KEY` 行中删除 `=xxx`。你还可以将上面命令中的 `llava` 替换为你选择的开源模型 *([llava](https://ollama.com/library/llava) 是目前少数支持图像的 Ollama 模型之一)*。你现在应该能够通过 [http://localhost:7878](http://localhost:7878) 访问 OpenUI。

*如果你对前端或后端进行了更改，则需要运行 `docker-compose build` 才能在服务中反映这些更改。*

## 开发

此存储库中配置了一个 [开发容器](https://github.com/wandb/openui/blob/main/.devcontainer/devcontainer.json)，这是最快开始的方法。

### Codespace

<img src="./assets/codespace.png" alt="New with options..." width="500" />

选择更多选项以创建 Codespace，然后选择 **New with options...**。如果你想获得非常快的启动时间，请选择美国西部地区。你还想配置你的 OPENAI_API_KEY 密钥，或者如果你想要尝试 Ollama，只需将其设置为 `xxx` *(你会需要至少 16GB 的内存)*。

进入代码空间后，可以在一个终端中运行服务器：`python -m openui --dev`。然后在新终端中：

```bash
cd /workspaces/openui/frontend
npm run dev
```

这应该会在端口 5173 上打开另一个服务，这就是你要访问的服务。对前端和后端的所有更改都会自动重新加载并反映在浏览器中。

### Ollama

代码空间会自动安装 ollama 并下载 `llava` 模型。你可以通过 `ollama list` 验证 Ollama 是否正在运行。如果失败，请打开一个新终端并运行 `ollama serve`。在 Codespaces 中，我们在启动时拉取 llava，所以你应该能在列表中看到它。你可以从左上角的应用程序设置齿轮图标中选择 Ollama 模型。你拉取的任何模型，例如 `ollama pull llama`，都会显示在设置模态框中。

<img src="./assets/ollama.png" width="500" alt="选择 Ollama 模型" />

### Gitpod

你可以轻松地通过 Gitpod 使用 Open UI，并预配置了 Open AI。

[![在 Gitpod 中打开](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/wandb/openui)

启动时，Open UI 会自动安装并启动。

在使用 Gitpod 之前：

* 确保你有一个 Gitpod 帐户。
* 要使用 Open AI 模型，请在你的 Gitpod [用户帐户](https://gitpod.io/user/variables) 中设置 `OPENAI_API_KEY` 环境变量。将范围设置为 `wandb/openui`（或者你 fork 的仓库）。

> 注意：其他（本地）模型也可能在更大的 Gitpod 实例类型中使用。所需的模型在 Gitpod 中未预配置，但可以轻松添加，如上所述。

### 资源

参见 [frontend](./frontend/README.md) 和 [backend](./backend/README.md) 目录中的 README 文件。
