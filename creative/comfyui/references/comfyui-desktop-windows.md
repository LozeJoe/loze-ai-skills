# ComfyUI Desktop — Windows 专属

> NVIDIA RTX 4060 8GB, Windows 11, ComfyUI Desktop 0.25.1

## 安装路径

ComfyUI Desktop 安装后的目录结构：
```
D:\Comfy-Desktop\
├── ComfyUI-Cache/
├── ComfyUI-Installs/     ← 各版本 ComfyUI 核心
└── ComfyUI-Shared/       ← 共享数据，所有版本共用
    ├── input/
    ├── models/
    │   ├── checkpoints/   ← ⚠️ 模型放这里
    │   ├── loras/
    │   ├── vae/
    │   ├── controlnet/
    │   ├── clip/
    │   ├── clip_vision/
    │   ├── embeddings/
    │   ├── upscale_models/
    │   └── ... (其他模型类型目录)
    └── output/
```

**关键**: 模型放在 `D:\Comfy-Desktop\ComfyUI-Shared\models\checkpoints\`，不是标准 ComfyUI 路径。

## 端口与服务

- 默认端口: `8188`
- 健康检查: `curl http://localhost:8188/system_stats`
- 队列查看: `curl http://localhost:8188/queue`
- 节点信息: `curl http://localhost:8188/object_info`

## CUDA 要求

NVIDIA 驱动需 >= 某个版本才能支持 CUDA。已知问题：
- 驱动 566.36 → `cudaErrorNotSupported`，需更新驱动
- RTX 4060 8GB 可跑 SDXL、Flux Dev fp8

## 首次启动检查清单

1. ComfyUI Desktop 启动后访问 `http://localhost:8188` 确认 UI 正常
2. 检查 `system_stats` 确认 GPU 被识别
3. 检查 `object_info/CheckpointLoaderSimple` 确认有模型
4. 如果 models/checkpoints 为空 → 需下载模型

## 推荐的轻量模型

对于 8GB VRAM (RTX 4060)：
| 模型 | 大小 | 风格 |
|------|------|------|
| DreamShaper 8 | ~2GB | 全能暖色调 |
| SD 1.5 原版 | ~4GB | 稳定通用 |
| Juggernaut XL | ~6.5GB | 写实质感 |

下载方式：用浏览器从 HuggingFace/CivitAI 下载 `.safetensors` 文件，放入 `D:\Comfy-Desktop\ComfyUI-Shared\models\checkpoints\`，然后刷新 ComfyUI 页面。

## 生成图片后

输出文件在 `D:\Comfy-Desktop\ComfyUI-Shared\output\`。
