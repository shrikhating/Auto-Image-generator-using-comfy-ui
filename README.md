# 🔗 LinkedIn AI Content Engine — Automated Educational Post & Image Pipeline

<div align="center">

![n8n](https://img.shields.io/badge/n8n-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)
![Ollama](https://img.shields.io/badge/Ollama-000000?style=for-the-badge&logo=ollama&logoColor=white)
![ComfyUI](https://img.shields.io/badge/ComfyUI%20%7C%20Stable%20Diffusion-6E3AFA?style=for-the-badge&logo=stablediffusion&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)

**End-to-End Automated LinkedIn Post Engine: Topic → Copy → AI Image → Post**  
_Random Topic Selection → Ollama LLM → Stable Diffusion (ComfyUI) → LinkedIn_

![Status](https://img.shields.io/badge/Status-Operational-brightgreen?style=flat-square)
![Domain](https://img.shields.io/badge/Domain-Content%20Automation%20%7C%20GenAI-blue?style=flat-square)
![Pipeline](https://img.shields.io/badge/Pipeline-n8n%20%7C%20Local%20AI-orange?style=flat-square)

</div>

---

## 📋 Overview

The **LinkedIn AI Content Engine** is a fully automated pipeline that produces and publishes educational LinkedIn posts — from topic selection through AI-generated post copy and image generation — without manual intervention.

The pipeline is orchestrated by **n8n** (self-hosted workflow automation), uses **Ollama** for local LLM post content generation, and **ComfyUI with Stable Diffusion** for AI image generation at 1344×768 resolution. Topics are randomly selected from a configurable bank, ensuring content diversity across automation runs.

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│          LinkedIn AI Content Engine — Automation Pipeline                    │
└─────────────────────────────────────────────────────────────────────────────┘

  ┌──────────────────────────────────┐
  │   TRIGGER                        │
  │   ─ n8n Schedule Node            │
  │     (cron: configurable cadence) │
  └──────────────┬───────────────────┘
                 │
                 ▼
  ┌──────────────────────────────────┐
  │   TOPIC SELECTION                │
  │   ─ Random pick from topic bank  │
  │     (JSON array in n8n config)   │
  │   ─ Topics: IIoT, AI/ML, Python, │
  │     Data Engineering, Industry   │
  │     4.0, Power BI, Cloud, etc.   │
  └──────────────┬───────────────────┘
                 │ Selected topic
                 ▼
  ┌──────────────────────────────────────────────────────────────┐
  │   POST COPY GENERATION (Ollama — Local LLM)                  │
  │                                                              │
  │   ─ n8n HTTP Request Node → POST localhost:11434/api/generate│
  │   ─ Prompt: structured template per post format              │
  │     · Hook line (attention-grabbing opener)                  │
  │     · 5-7 educational bullet points                          │
  │     · Call-to-action                                         │
  │     · 10–15 relevant hashtags                                │
  │   ─ Response: formatted LinkedIn post copy                   │
  └──────────────────────────────────────────────────────────────┘
                 │ Post copy text
                 ▼
  ┌──────────────────────────────────────────────────────────────┐
  │   IMAGE PROMPT GENERATION (Ollama — Structured JSON)         │
  │                                                              │
  │   ─ Second Ollama call requesting JSON-only output           │
  │   ─ Returns:                                                 │
  │     · "positive": detailed SD prompt for the image           │
  │     · "negative": negative prompt (what to avoid)           │
  │     · "style_preset": art style tag                          │
  │   ─ sanitize_url() applied to Ollama endpoint               │
  │     (schema + address correction)                            │
  └──────────────────────────────────────────────────────────────┘
                 │ Structured image prompt JSON
                 ▼
  ┌──────────────────────────────────────────────────────────────┐
  │   IMAGE GENERATION (ComfyUI — Stable Diffusion)              │
  │                                                              │
  │   ─ ComfyUI advanced pipeline with hires-fix                 │
  │   ─ Resolution: 1344×768 (landscape, LinkedIn-optimized)     │
  │   ─ Positive / negative prompts injected                     │
  │   ─ style_preset token prepended to positive prompt          │
  │   ─ Python script POSTs workflow JSON to ComfyUI API         │
  │     (localhost:8188)                                         │
  │   ─ Polls ComfyUI /history endpoint until image ready        │
  └──────────────────────────────────────────────────────────────┘
                 │ Generated image (PNG/JPEG)
                 ▼
  ┌──────────────────────────────────────────────────────────────┐
  │   LINKEDIN POST ASSEMBLY & PUBLISHING                        │
  │   ─ n8n LinkedIn node: post copy + image combined           │
  │   ─ Scheduled or manual trigger publish                      │
  └──────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Technology Stack

| Component | Technology | Purpose |
|---|---|---|
| **Orchestration** | n8n (self-hosted) | End-to-end workflow automation with visual node editor |
| **LLM (Post Copy)** | Ollama (local) | LinkedIn post copy generation from topic prompt |
| **LLM (Image Prompt)** | Ollama (local) | Structured JSON image prompt generation |
| **Image Generation** | ComfyUI + Stable Diffusion | 1344×768 AI image generation with hires-fix pipeline |
| **Config** | `config.json` | All pipeline settings externalized |
| **Language** | Python | ComfyUI API client, image polling, sanitize_url utility |

---

## ✨ Key Features

### 🎲 Random Topic Selection
- Topic bank maintained as a JSON array in n8n workflow configuration
- Topics span: IIoT, Industry 4.0, Python, AI/ML, Power BI, Data Engineering, Cloud Architecture, Automation
- Random selection node ensures no post repeats consecutively

### ✍️ Structured LinkedIn Post Generation (Ollama)
- Prompt template enforces LinkedIn post format:
  - **Hook** — attention-grabbing first line
  - **Body** — 5–7 educational insights or tips
  - **CTA** — call-to-action for engagement
  - **Hashtags** — 10–15 relevant tags for discoverability

### 🖼️ AI Image Prompt Pipeline (Ollama → Stable Diffusion)
- Ollama second call requests **JSON-only structured output** with three fields:
  - `positive`: detailed Stable Diffusion prompt matching the post topic
  - `negative`: what to avoid (artifacts, blur, text errors)
  - `style_preset`: art style tag (e.g., `cyberpunk`, `isometric`, `flat design`)
- `sanitize_url()` utility normalizes the Ollama API URL (handles schema prefix, address formatting)
- Structured JSON parsed by Python and injected into ComfyUI workflow node inputs

### 🎨 ComfyUI Advanced Pipeline
- Full hires-fix pipeline: base generation + upscale + refiner pass
- Output resolution: **1344×768** (horizontal, optimized for LinkedIn post image dimensions)
- `style_preset` token prepended to positive prompt to steer art direction
- Python polls ComfyUI `/history/{prompt_id}` until output image is available

### 🔁 n8n Workflow Orchestration
- All steps connected as n8n nodes with error handling branches
- HTTP Request nodes for Ollama API calls
- Custom code node for JSON parsing and image prompt construction
- LinkedIn integration node for final post submission
- Schedule-based auto-trigger (configurable cadence)

---

## 🔄 Pipeline Workflow

```
TRIGGER (n8n Schedule / Manual)
      │
      ▼
TOPIC SELECT
  └─ Random item from topic bank JSON array

POST COPY (Ollama Call 1)
  └─ Prompt: "Write a LinkedIn educational post about {topic} with hook, bullets, CTA, hashtags"
  └─ Returns: formatted post text

IMAGE PROMPT (Ollama Call 2)
  └─ Prompt: "Return JSON only: {positive, negative, style_preset} for a LinkedIn image about {topic}"
  └─ sanitize_url() applied to endpoint before call
  └─ Multi-strategy JSON parse → extract fields

IMAGE GENERATE (ComfyUI API)
  └─ Inject positive + negative into workflow JSON KSampler nodes
  └─ POST workflow to http://localhost:8188/prompt
  └─ Poll /history/{id} until status = 'complete'
  └─ Download output image

PUBLISH (n8n LinkedIn Node)
  └─ Post copy + image assembled
  └─ LinkedIn API call submits the post
```

---

## 📁 Project Structure

```
linkedin_content_engine/
├── n8n/
│   └── workflow.json              # Exported n8n workflow (importable)
├── comfyui/
│   ├── comfy_client.py            # ComfyUI API client — submit + poll
│   ├── workflow_template.json     # ComfyUI pipeline JSON template
│   └── sanitize_url.py            # URL schema/address normalization utility
├── prompts/
│   ├── post_prompt_template.txt   # LinkedIn post generation prompt
│   └── image_prompt_template.txt  # Structured image prompt generation prompt
├── topics/
│   └── topic_bank.json            # Topic array for random selection
├── config.json                    # All settings externalized
└── requirements.txt
```

---

## ⚙️ Configuration (`config.json`)

```json
{
  "ollama": {
    "endpoint": "http://localhost:11434",
    "post_model": "llama3.1:8b",
    "image_prompt_model": "llama3.1:8b"
  },
  "comfyui": {
    "endpoint": "http://localhost:8188",
    "output_width": 1344,
    "output_height": 768,
    "poll_interval_seconds": 3,
    "max_poll_attempts": 60
  },
  "post": {
    "hashtag_count": 12,
    "bullet_count": 6
  },
  "output_dir": "./generated"
}
```

---

## 🚀 Getting Started

### Prerequisites

- n8n installed and running (`npx n8n`)
- Ollama running locally with `llama3.1:8b` pulled
- ComfyUI installed with Stable Diffusion checkpoint
- LinkedIn Developer App credentials (for n8n LinkedIn node)

### Setup

```bash
git clone https://github.com/shrikhating/linkedin-content-engine.git
cd linkedin_content_engine
pip install -r requirements.txt

# Import n8n workflow
# n8n UI → Import Workflow → select n8n/workflow.json
```

### Run Manually

```bash
# Test ComfyUI client directly
python comfyui/comfy_client.py --topic "IIoT predictive maintenance"

# Or trigger the full n8n workflow manually from n8n dashboard
```

---

## 👤 Author

**Shrikant Khating**  
Senior BI Developer | AI Automation Engineer | Content Intelligence  
📧 shri.khating@gmail.com  
🔗 [LinkedIn](https://linkedin.com/in/shrikant-khating) · [GitHub](https://github.com/shrikhating)

> *Built as a solo personal innovation lab project — no team, no budget, production-grade engineering.*
