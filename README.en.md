# Introduction

This repository contains my local AI infrastructure deployment. The goal is to deploy AI using local resources, without relying on external services, in order to:

* Limit the cost of using AI
* Protect data privacy
* Control the user experience
* Ensure compliance with the European AI Act by implementing access control systems, guardrails, and observability, with persistent storage for both the infrastructure (models, inference engines, etc.) and access logs

> **Project status:** This repository documents an ongoing experimental laboratory. The architecture, selected tools, configurations, and deployment procedures may change as the evaluation progresses.

## Scope

The objective is to provide the following services:

* **Local generative text AI (LLM)** capable of:

  * Interacting with users in chatbot mode, including both direct questions and full conversations
  * Generating answers based on a predefined knowledge base using [RAG](https://en.wikipedia.org/wiki/Retrieval-augmented_generation) techniques supported by a vector database
  * Enhancing RAG responses through online (web) searches
  * Integrating external tools through public APIs and [MCP](https://modelcontextprotocol.io/docs/2026-07-28/getting-started/intro)
  * Solving complex problems using [AI agents](https://en.wikipedia.org/wiki/Intelligent_agent) capable of performing deep reasoning and invoking external tools
* **Local AI for software development**, supporting integration with current frameworks and tools such as Claude, Codex CLI in terminal mode, and Visual Studio Code
* Generation of **simple images** (basic illustrations, memes, comic panels, etc.)
* Local **audio generation / TTS**, including multilingual voice cloning capabilities (at least English, Spanish, German, and French)
* **Study support**, combining RAG-based augmented generation with document-based assistance (similar to NotebookLM)
* Advanced document recognition

Video generation has been excluded due to the limitations of the available hardware resources.

## Objectives and Constraints

* Use **open-source software** whenever possible. When this is not possible, identify the best commercial alternatives that integrate well with the deployment and, where applicable, are free to use locally.
* Define a deployment architecture that supports:

  * **Cross-platform deployment (Linux and macOS)**
  * A **modular architecture** that makes it possible to compose complex solutions from simple components (models, inference engines, servers, etc.)
  * The ability to select the appropriate compute engine (CUDA GPU, CPU, MLX, etc.) in order to achieve the best performance on each platform

## Selected Tools

* Inference engine: [Ollama](https://ollama.com/)

  * Alternative: [vLLM](https://vllm.ai/)
  * Alternative: [llama.cpp](https://github.com/ggml-org/llama.cpp)

* AI framework: [Open WebUI](https://openwebui.com/), with the following components:

  * Vector database: [Chroma](https://github.com/chroma-core/chroma)
  * Web search engine: [SearxNG](https://github.com/searxng/searxng)
  * Alternative: [LibreChat](https://www.librechat.ai/)
  * Alternative: [AnythingLLM](https://anythingllm.com/)
  * Alternative: [Dify](https://dify.ai/)

* Development:

  * Claude: [Claude](https://claude.ai/)
  * Codex CLI: [Codex CLI](https://learn.chatgpt.com/docs/codex/cli)
  * [Visual Studio Code](https://code.visualstudio.com/)

* Local image generation: [ComfyUI](https://github.com/comfyanonymous/ComfyUI)

  * Alternative: Running image-generation models directly from the inference engine. For example, with Ollama: `ollama run x/flux2-klein` or `ollama run x/z-image-turbo`

* Local study platform: [Open Notebook](https://www.open-notebook.ai/)

* Audio generation / TTS: [VoiceBox](https://voicebox.sh/), based on the Qwen-TTS model

* Advanced document recognition: [Marker](https://github.com/datalab-to/marker)

  * **IMPORTANT**: Marker uses [Surya OCR](https://github.com/datalab-to/surya) internally, and its models require a commercial license for use in enterprise environments

## Current Environment

The following machines are currently used.

* On-premises server

  * HP Victus PC
  * Intel i5-12400F
  * 32 GB RAM
  * NVIDIA GTX 1660 GPU
  * 6 GB VRAM
  * Oracle Linux 10.1 (compatible with Red Hat Enterprise Linux 10.1)

* Laptop

  * MacBook Pro 2025
  * Apple M4
  * 48 GB unified memory (shared between CPU and GPU)

# Deployment Architecture

## Global concepts
* Containers are used to support both cross-platform deployment and modularity.
* The container engine is Podman. Podman is the default container engine included in recent versions of Red Hat and compatible platforms. On macOS, [Podman Desktop](https://formulae.brew.sh/cask/podman-desktop) is used to provide the convenience of a GUI for container management, although it can also be used without a GUI when disk space is limited (`brew install podman`).

  * Alternatives: Docker or Rancher
* Containers are used in **rootless** mode, running under a local user account.
* [Quadlets](https://www.redhat.com/en/blog/quadlet-podman) are used for orchestration. This solution was chosen because Podman Compose support is more limited, while Quadlets provide simpler management across both platforms.

  * Alternatives: Docker Compose when using Docker, or Kubernetes if the number of modules increases

## Supported Platforms

The laboratory currently targets the currently existing two different local computing platforms:

- **Linux / x86-64 / NVIDIA GPU**
  - CUDA-based acceleration
  - Oracle Linux / Red Hat-compatible environment

- **macOS / Apple Silicon**
  - Apple Silicon unified memory architecture
  - MLX-based acceleration where supported

The deployment architecture aims to keep the higher-level services as portable as possible while allowing each platform to use the most appropriate AI inference backend.

## High level architecture

```
Local AI Infrastructure
├── Common LLM inferece engine
│   └── Ollama
│
├── AI Framework
│   └── Open WebUI
│   └── SearxNG
│   └── Chroma
│   └── MCP
│       └── WIP...
│
├── Image Generation
│   └── ComfyUI
│
├── Audio / TTS
│   └── VoiceBox / Qwen-TTS
│
├── Document Processing
│   └── Marker / Surya OCR
│
└── Study Platform
    └── Open Notebook
```

## AI Models

AI models are managed independently from the services that use them whenever possible.

Different model formats and runtimes may be used depending on the target platform:

- GGUF models
- Ollama-compatible models
- MLX models
- Diffusion models for image generation
- Specialized OCR and document-processing models

Model selection and evaluation are documented separately from the infrastructure deployment.

# Repository Structure

```text
.
├── openwebui/          # Open WebUI deployment
├── comfyui/            # Local image generation deployment
├── marker/             # Document recognition deployment
├── notebooks/          # Study and knowledge-management tools
├── models/             # Model documentation and evaluation
└── docs/               # General documentation
```

## Open WebUI Deployment

* [Linux Server](openwebui/README.linux.es.md)
* [macOS Laptop](openwebui/README.macos.es.md)
