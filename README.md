# 🎨 LinkedIn Image Generator — Ollama + ComfyUI AI Image Pipeline

<div align="center">

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Ollama](https://img.shields.io/badge/Ollama-000000?style=for-the-badge&logo=ollama&logoColor=white)
![ComfyUI](https://img.shields.io/badge/ComfyUI-6E3AFA?style=for-the-badge&logoColor=white)
![Stable Diffusion](https://img.shields.io/badge/Stable%20Diffusion-FF6B35?style=for-the-badge&logoColor=white)

**LLM-Driven Prompt Engineering → Advanced Stable Diffusion Pipeline → 1344×768 LinkedIn Images**  
_Ollama generates structured prompt JSON · ComfyUI hires-fix pipeline produces final image_

![Status](https://img.shields.io/badge/Status-Operational-brightgreen?style=flat-square)
![Resolution](https://img.shields.io/badge/Output-1344×768-blue?style=flat-square)
![Pipeline](https://img.shields.io/badge/Pipeline-Hires--Fix%20Advanced-orange?style=flat-square)

</div>

---

## 📋 Overview

The **LinkedIn Image Generator** is a Python pipeline that converts a text topic into a production-ready 1344×768 LinkedIn post image by chaining two local AI systems:

1. **Ollama LLM** — receives a topic and returns a structured JSON object containing a `positive` Stable Diffusion prompt, a `negative` prompt, and a `style_preset` art direction tag.
2. **ComfyUI with Stable Diffusion** — executes an advanced hires-fix generation pipeline using the LLM-constructed prompt to produce the final image at 1344×768.

All configuration is externalized to `config.json`. The `sanitize_url()` utility handles URL schema and address normalization to prevent connection errors when Ollama or ComfyUI endpoint strings are malformed.

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│           LinkedIn Image Generator — Two-Stage AI Pipeline                   │
└─────────────────────────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────┐
  │   INPUT                          │
  │   ─ topic: str                   │
  │     ("IIoT predictive           │
  │      maintenance" etc.)          │
  └──────────────┬───────────────────┘
                 │
                 ▼
  ┌──────────────────────────────────────────────────────────────┐
  │  STAGE 1 — PROMPT ENGINEERING (Ollama LLM)                   │
  │                                                              │
  │  sanitize_url(config["ollama"]["endpoint"])                  │
  │    └─ Normalizes schema prefix (http:// / https://)         │
  │    └─ Fixes IP/hostname formatting                          │
  │    └─ Ensures no trailing slash conflicts                    │
  │                                                              │
  │  POST → {sanitized_endpoint}/api/generate                    │
  │  Payload:                                                    │
  │  {                                                           │
  │    "model": "llama3.1:8b",                                  │
  │    "prompt": "Return JSON only. No preamble. No markdown.   │
  │      Fields: positive (SD prompt), negative (SD negative),  │
  │      style_preset (art style tag). Topic: {topic}",         │
  │    "stream": false                                           │
  │  }                                                           │
  │                                                              │
  │  Multi-strategy JSON parse on response:                     │
  │  Strategy 1: json.loads(raw_response)                       │
  │  Strategy 2: regex extract {...} block                      │
  │  Strategy 3: key-value reconstruction from partial output   │
  │                                                              │
  │  Output: {"positive": "...", "negative": "...",             │
  │           "style_preset": "..."}                            │
  └──────────────────────────────────────────────────────────────┘
                 │ Structured prompt JSON
                 ▼
  ┌──────────────────────────────────────────────────────────────┐
  │  STAGE 2 — IMAGE GENERATION (ComfyUI / Stable Diffusion)    │
  │                                                              │
  │  Workflow template loaded from workflow_template.json        │
  │                                                              │
  │  Prompt injection into workflow nodes:                       │
  │  ─ KSampler positive input ← style_preset + ", " + positive │
  │  ─ KSampler negative input ← negative                       │
  │  ─ EmptyLatentImage: width=1344, height=768                 │
  │                                                              │
  │  Advanced hires-fix pipeline:                               │
  │  ─ Base generation at lower resolution                      │
  │  ─ Upscale pass                                             │
  │  ─ Refiner KSampler pass at 1344×768                        │
  │                                                              │
  │  POST workflow → http://localhost:8188/prompt               │
  │  ─ Response: {"prompt_id": "uuid"}                          │
  │                                                              │
  │  Poll loop: GET /history/{prompt_id}                        │
  │  ─ Check until status "completed"                           │
  │  ─ Extract output image filename                            │
  │  ─ Download image from /view endpoint                       │
  │                                                              │
  │  Output: 1344×768 PNG saved to ./output/{timestamp}.png     │
  └──────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Technology Stack

| Component | Technology | Purpose |
|---|---|---|
| **Language** | Python | Pipeline orchestration |
| **LLM** | Ollama (local) | Structured prompt JSON generation from topic |
| **Image Generation** | ComfyUI | Stable Diffusion API server with workflow execution |
| **SD Model** | Stable Diffusion (ComfyUI checkpoint) | Advanced hires-fix image generation |
| **URL Normalization** | `sanitize_url()` (custom utility) | Endpoint string normalization before API calls |
| **Config** | `config.json` | All settings externalized from code |
| **HTTP Client** | Python `requests` | Ollama and ComfyUI API calls |

---

## ✨ Key Features

### 🧠 LLM Prompt Engineering (Ollama)
- Topic-to-structured-prompt in a single Ollama call
- Response is strictly constrained to JSON-only output (no preamble, no markdown fences)
- Three-strategy parser handles all common model output deviations:
  - **Strategy 1**: Clean `json.loads()` on raw response
  - **Strategy 2**: Regex `{...}` block extraction from responses with surrounding text
  - **Strategy 3**: Key-by-key reconstruction when model returns partial/malformed JSON

### 🔧 `sanitize_url()` Utility
A utility function that normalizes any Ollama or ComfyUI endpoint string before making API calls:
```python
sanitize_url("localhost:11434")      → "http://localhost:11434"
sanitize_url("http://localhost:11434/") → "http://localhost:11434"
sanitize_url("192.168.0.5:8188")    → "http://192.168.0.5:8188"
```
This prevents `MissingSchema`, `InvalidSchema`, and double-slash URL errors — a common failure point when config values are edited without schema prefix.

### 🎨 ComfyUI Advanced Hires-Fix Pipeline
- **Base pass**: generates image at a lower resolution for speed
- **Upscale pass**: scales base result up to 1344×768 using ESRGAN or latent upscaler
- **Refiner pass**: second KSampler run at full 1344×768 for sharpness and detail
- Workflow executed as ComfyUI API JSON — no ComfyUI UI interaction required

### 📐 LinkedIn-Optimized Output (1344×768)
- Horizontal format optimized for LinkedIn post image card display
- PNG output at full resolution suitable for upload without additional resizing

---

## 🔄 Pipeline Workflow

```
INPUT: topic = "Industry 4.0 predictive maintenance"

STAGE 1 — PROMPT ENGINEERING
  └─ sanitize_url() → endpoint normalized
  └─ Ollama API call: topic → structured JSON
  └─ Multi-strategy JSON parse
  └─ Output:
       positive: "futuristic factory floor, holographic displays, 
                  glowing sensor nodes, blue ambient lighting, 
                  isometric view, ultra-detailed, 4K"
       negative: "blurry, text errors, watermark, low quality"
       style_preset: "cyberpunk industrial"

STAGE 2 — IMAGE GENERATION
  └─ Load workflow_template.json
  └─ Inject: positive = style_preset + ", " + positive
  └─ Inject: negative → negative KSampler node
  └─ Set: EmptyLatentImage width=1344, height=768
  └─ POST to ComfyUI /prompt
  └─ Poll /history/{prompt_id} until complete
  └─ Download output image
  └─ Save: output/20260626_143022.png

OUTPUT: 1344×768 PNG — LinkedIn-ready
```

---

## 📁 Project Structure

```
linkedin_image_generator/
├── main.py                        # Pipeline entry point
├── prompt_engineer.py             # Ollama call + multi-strategy JSON parser
├── comfy_client.py                # ComfyUI API: submit workflow + poll + download
├── sanitize_url.py                # URL schema/address normalization utility
├── workflow_template.json         # ComfyUI advanced hires-fix workflow JSON
├── config.json                    # All configuration externalized
├── output/                        # Generated images (timestamped PNG)
└── requirements.txt
```

---

## ⚙️ Configuration (`config.json`)

```json
{
  "ollama": {
    "endpoint": "http://localhost:11434",
    "model": "llama3.1:8b",
    "stream": false
  },
  "comfyui": {
    "endpoint": "http://localhost:8188",
    "poll_interval_seconds": 3,
    "max_poll_attempts": 60
  },
  "output": {
    "width": 1344,
    "height": 768,
    "format": "png",
    "directory": "./output"
  }
}
```

---

## 🚀 Getting Started

### Prerequisites

- Python 3.10+
- Ollama running locally with `llama3.1:8b` pulled
- ComfyUI installed and running with a Stable Diffusion checkpoint loaded
- ComfyUI: Advanced hires-fix workflow configured in `workflow_template.json`

### Installation

```bash
git clone https://github.com/shrikhating/linkedin-image-generator.git
cd linkedin_image_generator
pip install -r requirements.txt
```

### Run

```bash
python main.py --topic "Python asyncio for beginners"
```

Output saved to: `output/YYYYMMDD_HHMMSS.png`

---

## 👤 Author

**Shrikant Khating**  
Senior BI Developer | AI Automation Engineer | Python Developer  
📧 shri.khating@gmail.com  
🔗 [LinkedIn](https://linkedin.com/in/shrikant-khating) · [GitHub](https://github.com/shrikhating)

> *Built as a solo personal innovation lab project — no team, no budget, production-grade engineering.*
