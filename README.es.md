# Introducción

Este repositorio contiene mi despliegue de IA en infraestructura local. El objetivo es hacer un despliegue de IA usando recursos locales y sin utilizar servicios externos para:

- Limitar el coste del uso de la IA
- Proteger la privacidad de los datos
- Controlar la experiencia del usuario
- Garantizar el cumplimiento de la normativa AI Act europea, mediante la implementación de los sistemas de control de acceso, guardarrailes y observabilidad, con persistencia tanto de la infrastructura (modelos, motores, ...) como de los log de acceso.

Actualmente este es un trabajo en curso. 

## Alcance

Se pretende proveer de los siguientes servicios:

- **IA local generativa de texto (LLM)** que permita:
    - Interactuar con el usuario en modo ChatBot, incluyendo preguntas directas o diálogos completos
    - Generar respuestas sobre una base de conocimientos predefinida usando técnicas [RAG](https://es.wikipedia.org/wiki/Generaci%C3%B3n_aumentada_por_recuperaci%C3%B3n) apoyadas en base de datos vectorial
    - Mejorar las respuestas RAG mediante búsquedas on-line (web)
    - Integrar herramientas externas usando APIs públicas / [MCP](https://modelcontextprotocol.io/docs/2026-07-28/getting-started/intro)
    - Resolver problemas complejos usando el [agentes de IA](https://es.wikipedia.org/wiki/Agente_de_inteligencia_artificial) que realizan razonamiento profundo e invocan las herramientas externas
- **IA Local para soporte al desarrollo** que permita integracion con los frameworks actuales: Claude, Codex CLI en modo terminal e integrado con VSCode
- Generación de **imágenes simples** (nivel ilustracion básica, meme, viñeta, ...)
- Generación de **audio** en local TTS con posibilidad de clonado de voz multiidioma (al menos inglés, español, alemán y francés)
- Soporte al **estudio**, combinando generación aumentada con RAG y soporte a la documentación (tipo NotebookLM)
- Adicional: Reconocimiento documental avanzado 

Se ha descartado la generación de video, dada la limitación de recursos disponibles.

## Objetivos y restricciones

- Utilizar **herramientas de software libre** siempre que sea posible. Cuando no lo sea, identificar las mejores alternativas comerciales que mejor se integren en el despliegue, y en su caso, que sean de uso gratuito para uso local
- Definir una arquitectura de despliegue que permita:
    - Despliegue **multiplataforma (Linux, macOS)**
    - Despliegue **modular** que permita componer soluciones complejas mediantes elementos simples (modelos, motores, servidores, ...)
    - Poder elegir el motor de cómputo (Cuda GPU, CPU, MLX, ...) para poder aprovechar el mejor rendimiento en cada plataforma

## Herramientas seleccionadas
- Motor de inferencia: [Ollama](https://ollama.com/)
    - Alternativa: [vLLM](https://vllm.ai/)
    - Alternativa: [llama.cpp](https://github.com/ggml-org/llama.cpp)
- Frameworks de IA: [Open WebUI](https://openwebui.com/) con las siguientes herramientas
    - Base de datos vectorial: [Chroma](https://github.com/chroma-core/chroma)
    - Motor de búsqueda WEB: [SearxNG](https://github.com/searxng/searxng)
    - Alternativa: [LibreChat](https://www.librechat.ai/es)
    - Alternativa: [AnythingLLM](https://anythingllm.com/)
    - Alternativa: [Dify](https://dify.ai/)
- Desarrollo
    - Claude: [Claude](https://claude.ai/)
    - Codex CLI: [Codex CLI](https://learn.chatgpt.com/docs/codex/cli)
    - [Visual Studio Code](https://code.visualstudio.com/)
- Generación de imágenes en local: [ComfyUI](https://github.com/comfyanonymous/ComfyUI)
    - Alternativa: Ejecución de modelos de generación desde el motor de inferencia. Ej, en ollama `ollama run x/flux2-klein` o `ollama run x/z-image-turbo` 
- Plataforma de estudio en local: [Open Notebook](https://www.open-notebook.ai/)
- Generación de audio / TTS: [VoiceBox](https://voicebox.sh/) sobre modelo Qwen-TTS
- Reconocimiento avanzado de documentos: [Marker](https://github.com/datalab-to/marker)
    - **IMPORTANTE**: Marker usa por debajo [Surya OCR](https://github.com/datalab-to/surya) que requiere una licencia comercial para usar los modelos en entornos empresariales

## Entorno actual

Usamos las siguientes máquinas.

- Servidor on-premise
    - PC HP Victus
    - Intel i5-12400F
    - 32GB RAM
    - GPU NVIDIA 1660
    - VRAM 6GB
    - Oracle Linux 10.1 (RedHat 10.1 compatible)

- Portatil
    - Macbook PRO 2025
    - M4
    - 48 GB RAM (compartida CPU-GPU)


## Arquitectura del despliegue
- Para cumplir la multiplataforma y la modularidad se utilizan contendedores
- El motor de contenedores es Podman. Podman es el contenedor incluido por omisión en las versiones recientes de RedHat y plataformas compatibles. En MacOS se utiliza [podman-desktop](https://formulae.brew.sh/cask/podman-desktop) para tener la comodidad de una GUI para administrar los contenedores, aunque si ha restricciones de espacio se puede utilizar sin GUI (`brew install podman`)
    - Alternativas: Docker o Rancher
- Se utilizan los contenedores en modo **rootless** (usando un usuario local) 
- Para orquestación se utiliza la tecnología de [quadlets](https://www.redhat.com/en/blog/quadlet-podman). La decisión por utilizar esta solución vino de que el soporte de podman compose es más limitado y porque el manejo es más simple en ambas plataformas
    - Alternativas: Docker-compose si se usa Docker, Kubernetes si aumenta el número de módulos


# Despliegue de Open WebUI

- [Servidor Linux](deploy.openwebui.linux.es.md)
- [Portatil MacOS](deploy.openwebui.macos.es.md)

