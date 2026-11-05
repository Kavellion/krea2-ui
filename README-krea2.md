# Krea 2 Turbo UI

A standalone text-to-image app for **krea/Krea-2-Turbo** — Krea AI's
first from-scratch foundation image model (12B dense diffusion
transformer, Qwen3-VL text encoder, Qwen-Image VAE), in the Turbo
checkpoint only: the 8-step distilled generation build. Same shape
as the Kandinsky, Sana, Z-Image, SD 3.5, PixArt, FLUX.1, Chroma,
Qwen, and HiDream UIs in this family — no Jupyter, no Colab. Run
it, a page opens in your browser, type a prompt, hit Generate, a
picture appears underneath.

Features: prompt queue (one prompt per line, all lines share one
batch folder), images-per-prompt batch, seed control (-1 =
random, a fixed seed repeats as S, S+1, S+2...), dynamic prompts
(`{a|b|c}`, re-picked for every image), history (last 10),
Random prompt, size presets up to 2048×2048 with pace labels,
Star batch (copies the batch folder into `images_out/favorites/`,
never overwriting), Unload models, a Stop button — and **two LoRA
slots**, the reason this build exists: Krea 2 LoRAs are trained on
the Raw checkpoint and run on Turbo.

**LoRA slots.** Two slots, each taking a local `.safetensors`
file with its own strength (0.0–1.5 slider + a type-in box that
wins when filled; a typed 0.0 really means off). Adapters are
applied at runtime with `set_adapters` scaling — never fused into
the FP8 weights. Trigger words live in the prompt, per each
LoRA's own card. Files in the ecosystem's real formats load:
diffusers/PEFT files (`transformer.*.lora_A/lora_B`), the official
krea style LoRAs, and Krea-native files (`base_model.model.*`,
the fal / official-trainer format) — the app audits each file's
keys before applying it, and a file whose keys convert to **zero**
transformer modules is a failed load, named in the status line:
the slot never silently generates an unadapted image. Queue lines
can change LoRA per line (`@lora1=C:\loras\file.safetensors | prompt`
or `@lora2=off | prompt`); every line's `settings.txt` block
records its LoRA state.

**Turbo truths.** Guidance is fixed at **0.0** and shown as a
labeled fixed value, not a slider — Turbo is guidance-distilled
and the pipeline's own defaults (28 steps, guidance 4.5) are the
Raw recipe, so the app passes the Turbo values on every call:
8 steps by default (slider 1–16). The negative prompt box is
greyed and labeled: the pipeline documentation says the negative
prompt is *"Ignored when guidance_scale <= 0"*. What you type is
still logged verbatim in `settings.txt` — the paper trail never
pretends it was used.

**Gated model, session-only token.** The weights carry the
**Krea 2 Community License** (open weights, NOT OSI open source:
commercial use is capped under US$1M company-wide annual revenue;
the license carries a content-filtering obligation for deployers;
license questions: opensource@krea.ai). The Hugging Face repo is
gated: accept the license on the repo page once, then paste a
Hugging Face token into the app's Token box for the first
download. The token lives only in that box for this session —
never written to a file or log — and leaving the box empty never
overrides a saved Hugging Face login.

## Requirements

- Windows PC with an NVIDIA GPU (built for an RTX 4070 12 GB and
  32 GB system RAM).
- **Python is included** in this package (embedded Python 3.12 in
  `python_embeded/`, ComfyUI portable style). Nothing to install
  first; no system Python is touched or needed. Python 3.12 is
  required because Diffusers 0.42 dropped Python 3.10.
- ~36 GB free disk for the model download.

## Run

```
install.bat   (once)
run.bat       (every time after)
```

First Generate downloads the model from the repo's component
folders (~35.9 GB) and loads it once; repeats reuse what's loaded.
The loader never downloads the repo's 26.3 GB single file
(`turbo.safetensors`, meant for other tools) — the component
folders carry everything.

## Memory (RTX 4070 12 GB, 32 GB RAM)

| Piece | How it loads | Parked |
|---|---|---|
| Transformer (12B, 28 blocks) | FP8 at load (torchao Float8WeightOnly) | ~13.0 GB |
| Text encoder (Qwen3-VL-4B) | FP8 at load (torchao Float8WeightOnly) | ~4.9 GB |
| VAE (Qwen-Image) | BF16 | ~0.25 GB |
| **Total** | CPU offload always ON | **~18.2 GB** |

That leaves ~13.8 GB of headroom — the roomiest fit of any heavy
model in this family; two fat LoRAs add at most ~1 GB more. There
is deliberately **no precision selector**: a BF16 transformer
would park 26.6 GB next to the encoder (~31.7 GB total) and bust
a 32 GB machine, so BF16 is not offered at all, and if an FP8
load fails the app stops with a plain error instead of silently
parking weights the machine cannot hold.

Expect roughly 1.5–2.5 minutes a frame at 1024×1024 on the
RTX 4070 (8 steps, offload streaming) and several minutes at
2048×2048. These are estimates — the model has not run on the
target PC yet (see Status).

Pictures land in `images_out/<batch name>/`, frames numbered
continuously across runs, with `settings.txt` appended once per
run (prompt, size, steps, the fixed guidance line, LoRA state per
line, seeds, any expanded dynamics verbatim).

## Status

**CODE VERIFIED ONLY.** The full bench suite (real Qwen3-VL
tokenizer/processor load, the app's own loader against a shaped
snapshot of the real repo files, a LoRA suite run against three
real Krea 2 LoRA files covering both ecosystem key formats,
flow/queue/batch/star/settings, installer audit incl. the
bootstrap-sequence scan) passes in a fresh-extract cold verify —
see BUILD-NOTES.txt in the goal folder for the exact tallies and
the LoRA verdict. Publishing waits for the first picture on the
target PC, per family rule: access-form click → fresh-folder
install → single → single with an official LoRA (the A/B) →
small queue with a LoRA swap → batch.
