---
title: 斯坦福小镇简化版--AI-Town--搭建运行
catalog: true
date: 2025-02-04 19:05:08
subtitle:
header-img:
tags:
- 机器学习
---
<meta name="referrer" content="no-referrer"/>
[斯坦福小镇](https://github.com/joonspk-research/generative_agents)在 GitHub 开源，由于需要调用 GPT4 接口，费用不低，所以这里有一个简化版 [AI-Town](https://github.com/a16z-infra/ai-town)，区别是可以使用本地大语言模型，简化了许多操作，可以本地免费搭建运行，当然其基本思想是一样的。

此外还有个[汉化版 AI-Town](https://github.com/Steven-Luo/ai-town-cn)，对中文做了支持，修改了以下文件：
1. 修改 data/characters.ts，将角色的 identity 和 plan 翻译成中文
2. 回话、记忆引导 Prompt 中文化，对 convex/agent/memory.ts、convex/agent/conversation.ts 中相关的 Prompt 翻译为中文
3. 修改 convex/util/llm.ts，将 LLM 和 Embedding 模型修改为对中文支持较好的模型

![](https://upload-images.jianshu.io/upload_images/2708793-8e347a5b7e154a09.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1、安装 node 和 npm

由于是一个 node 项目，所以第一步需要安装 node 和 npm，其中 node 版本需要为 18。
```
# 安装
nvm install 18.12.0

# 切换到指定版本
nvm use 18.12.0

# 查看版本
node -v
npm -v
```

### 2、安装 Ollama

[Ollama](https://ollama.com/) 是一个开源的 LLM（大型语言模型）服务工具，用于简化在本地运行大语言模型，降低使用大语言模型的门槛，能够在本地环境快速实验、管理和部署最新大语言模型，包括如 Llama 3、Phi 3、Mistral、Gemma 等开源的大型语言模型。支持的模型：https://ollama.com/library

下载模型，模型较大，下载可能需要较长时间：
```
## 英文版依赖
ollama pull llama3
ollama pull mxbai-embed-large

## 中文版依赖
ollama pull qwen2:7b
ollama pull znbang/bge:large-zh-v1.5-q8_0
```

下载成功后，运行模型：
```
# 英文版运行 llama3 模型
ollama run llama3

# 中文版运行 qwen2:7b 模型
ollama run llama3
```

### 3、使用 Convex 进行本地部署
 
[Convex](https://dashboard.convex.dev/) 是一个全栈 TypeScript 开发平台，旨在为开发者提供构建产品所需的一切后端应用功能，整合了多种必要的工具和服务，使开发者能够更加高效地创建、部署和管理他们的应用程序。


Convex 依赖 `just`，后续会使用 `just` 命令，先安装 `just`，然后下载并后台运行 convex。
```
# For Macs:
# 安装 just
brew install just

# 从GitHub上下载Convex后端的预编译二进制文件
curl  -L -O https://github.com/get-convex/convex-backend/releases/latest/download/convex-local-backend-aarch64-apple-darwin.zip

# 解压
unzip convex-local-backend-aarch64-apple-darwin.zip

# 初始化convex，仅第一次需要
just convex run init

# Runs the convex
./convex-local-backend
```

同时还需要去 [Convex](https://dashboard.convex.dev/) 后台，注册登陆并新建一个工程。

![](https://upload-images.jianshu.io/upload_images/2708793-02fc6de185890316.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 4、克隆代码
```
git clone https://github.com/a16z-infra/ai-town.git ## 中文版使用：git clone git@github.com:Steven-Luo/ai-town-cn.git

cd ai-town

npm install

# 同时运行前端和后端
npm run dev
```

第一次运行可能会遇到 `No CONVEX_DEPLOYMENT set` 的问题，原因是 Convex 部署环境变量未正确设置。需要：

1. 确保 Convex 后端正在运行
2. 设置 Convex 部署环境变量 `npx convex dev`，这个命令会提示你选择一个现有的项目或创建一个新的项目。如果你已经有一个项目，选择 existing project 并选择该项目。这将修改 .env.local 文件，添加 CONVEX_DEPLOYMENT 环境变量。
3. 运行初始化脚本 `just convex run init`



![](https://upload-images.jianshu.io/upload_images/2708793-81f0e7c4b3d31de3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
