# CLAUDE.md

Qwen Image 2512 — text-to-image RunPod Serverless endpoint with Lightning LoRA (4-step / 8-step) support. See root `../CLAUDE.md` for monorepo-wide context.

## Handler Parameters (Simplified Input)

| Param | Type | Default | Notes |
|-------|------|---------|-------|
| `prompt` | str | required | Text prompt |
| `seed` | int | random | Reproducibility |
| `width` | int | 1328 | Output width |
| `height` | int | 1328 | Output height |
| `steps` | int | 4 | Also determines LoRA mode via `_resolve_lora_mode()` |
| `negative_prompt` | str | `""` | |
| `batch_size` | int | 1 | |
| `lora` | str | auto | `"4step"`, `"8step"`, or `"none"` — overrides step-based auto-selection |
| `cfg` | float | auto | Auto: 1.0 for Lightning, 4.0 for base |
| `shift` | float | 3.1 | ModelSamplingAuraFlow shift |
| `sampler` | str | `"euler"` | |
| `scheduler` | str | `"simple"` | |

## LoRA Resolution (`_resolve_lora_mode`)

- `steps <= 4` → 4-step LoRA
- `5 <= steps <= 8` → 8-step LoRA
- `steps > 8` → base model (no LoRA, CFG defaults to 4.0)

Explicit `lora` param overrides step-based selection.

## Models

| File | Relative path |
|------|---------------|
| Diffusion (fp8) | `diffusion_models/qwen_image_2512_fp8_e4m3fn.safetensors` |
| Text encoder | `clip/qwen_2.5_vl_7b_fp8_scaled.safetensors` |
| VAE | `vae/qwen_image_vae.safetensors` |
| 4-step LoRA | `loras/Qwen-Image-2512-Lightning-4steps-V1.0-fp32.safetensors` |
| 8-step LoRA | `loras/Qwen-Image-2512-Lightning-8steps-V1.0-fp32.safetensors` |

Base path: `/runpod-volume/models/` when network volume attached, `/comfyui/models/` otherwise. `start.sh` symlinks `/comfyui/models/X` → volume path. `check-models.sh` downloads to `$MODEL_BASE`.

**Sync requirement:** `QWEN_MODELS` dict in `handler.py` and `MODELS` array in `src/check-models.sh` must match.

## Key Node IDs

These are model-specific and **not portable** to other projects:
- KSampler: `"238:230"` — steps, cfg, sampler_name, scheduler, seed
- EmptySD3LatentImage: `"238:232"` — width, height, batch_size
- LoRA switch: `"238:229"` — boolean enables Lightning path
- LoraLoaderModelOnly: `"238:221"` — lora_name from `LORA_FILES`
- Positive text: `"238:227"`, Negative text: `"238:228"`
- Shift: `"238:222"` — ModelSamplingAuraFlow

## Build & Deploy

```bash
docker build -t nyxcoolminds/qwen-img:qwen-2512 .
docker push nyxcoolminds/qwen-img:qwen-2512
docker rmi nyxcoolminds/qwen-img:qwen-2512 && docker builder prune -af
```

Release: `gh release create v1.X.Y --title "v1.X.Y" --notes "description"`

## RunPod Hub Tests

`.runpod/tests.json` — health_check only (300s timeout, RTX 4090, CUDA 12.8).
