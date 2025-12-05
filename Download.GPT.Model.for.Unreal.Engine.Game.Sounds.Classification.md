## OpenAI ChatGPT v1.2025.330 Download.GPT.Model.For.Sounds.md

**Q: I have Python and NodeJS installed on my Windows 11 x86i64 platform. I downloaded WAV audio files from an AAA games SoundFX library and I would now like to download an LLM GPT model to categorize the way sounds could be used in my Unreal Engine 3rd Person MMORPG Combat game based on the Roman gladiators and the Coliseum.**

**A:** Use an audio model (CLAP) plus a small Python script to auto-tag your WAVs; you don’t actually need a full GPT for this. You can add a local LLM later if you want richer prose descriptions.

Below is a concrete, local, CPU-friendly pipeline for Windows 11 + Python.

---

## 1. Overall approach

1. Define a gladiator-combat sound taxonomy (labels like `sword_slash_light`, `shield_block_heavy`, `crowd_cheer_large`, etc.).
2. Use a contrastive audio–text model (CLAP) to score how well each WAV matches each label’s text description.([arXiv][1])
3. For each file, pick the top-N labels and write them to a CSV/JSON.
4. Import that CSV/JSON into Unreal as a Data Table or use it to rename/organize assets.

CLAP is trained on generic audio + text captions and supports zero-shot classification (no fine-tuning needed).([arXiv][1])

---

## 2. Install the tooling (Python)

Assumptions:

* Python 3.10+ on Windows 11 x86_64.
* CPU only is fine; GPU is a bonus.

In a new venv:

```bash
python -m venv venv
venv\Scripts\activate

pip install --upgrade pip
pip install "torch" "torchaudio" --index-url https://download.pytorch.org/whl/cpu
pip install "transformers>=4.47.0" soundfile numpy
```

CLAP is integrated into `transformers` via `ClapModel` and `AutoProcessor`.([AiDocZh][2])

Also install ffmpeg (for robust audio decoding) via e.g. Chocolatey or the official Windows builds.

---

## 3. Define a gladiator sound taxonomy

You can tune this list; the key is to describe *use* in Roman arena combat terms so CLAP has something to latch onto.

Example label set:

```python
GLADIATOR_LABELS = {
    "melee_sword_slash_light":
        "short metal sword slash, close combat, gladius cutting air or light armor, dry, close mic",
    "melee_sword_slash_heavy":
        "heavy metal sword swing with strong impact, brutal gladiator strike, lots of low mid energy",
    "melee_spear_thrust":
        "fast spear or javelin thrust, sharp attack transient, point piercing target",
    "melee_mace_blunt":
        "blunt metal mace hit on armor, heavy low thump with metallic clank",
    "hit_armor_metal_light":
        "light impact on metal armor, small shield tap, subtle metallic ring",
    "hit_armor_metal_heavy":
        "strong metal impact on armor or shield, loud clang and ring, gladiator blocking attack",
    "hit_flesh_light":
        "light flesh hit, muted wet thud, little or no armor",
    "hit_flesh_heavy":
        "heavy body impact with flesh and bone, thick low frequency thump, possible gore",
    "shield_block_wood":
        "wooden shield block, woody thud with small rattle, roman scutum",
    "shield_block_metal":
        "metal shield block, plate clash, ringing edge",
    "footstep_sand":
        "human footsteps on sand or dirt arena floor, soft granular scrapes",
    "footstep_stone":
        "sandaled footsteps on stone, hard clicks and small reverbs, coliseum corridors",
    "vocal_battle_shout":
        "single male gladiator battle shout or yell, aggressive voice, medium reverb",
    "vocal_pain_cry":
        "human pain scream or grunt, hurt vocalization in combat",
    "crowd_cheer_large":
        "large ancient roman coliseum crowd cheering, wide stereo ambience, wall of voices",
    "crowd_boo":
        "crowd booing or disapproving, descending vocal reactions",
    "crowd_murmur_idle":
        "large crowd idle murmur, distant talking ambience, no big cheer",
    "environment_fire":
        "fire or torches burning, crackling flames, medieval ambience",
    "environment_chain_rattle":
        "chains rattling, metal links clanking, dungeon like",
    "ui_menu":
        "short clean user interface click or confirm sound, non diegetic, no ambience",
    "ui_error":
        "short clean user interface error or cancel sound, non diegetic",
}
```

You can expand this with magic, boss abilities, UI layers, etc., if your MMORPG deviates from strict realism.

---

## 4. CLAP-based auto-tagger (Python)

This script:

* Walks a directory of WAVs.
* For each file, computes CLAP audio–text similarities.
* Assigns the top-K labels.
* Writes results to `sound_tags.csv`.

```python
import os
import csv
from pathlib import Path

import torch
import torchaudio
from transformers import AutoProcessor, ClapModel

# ---------- CONFIG ----------

AUDIO_DIR = r"C:\path\to\your\AAA_SFX_wavs"
OUTPUT_CSV = r"C:\path\to\sound_tags.csv"
TOP_K = 3  # number of labels per sound

MODEL_ID = "laion/clap-htsat-fused"  # zero-shot audio-text model

GLADIATOR_LABELS = {
    "melee_sword_slash_light":
        "short metal sword slash, close combat, gladius cutting air or light armor, dry, close mic",
    "melee_sword_slash_heavy":
        "heavy metal sword swing with strong impact, brutal gladiator strike, lots of low mid energy",
    "melee_spear_thrust":
        "fast spear or javelin thrust, sharp attack transient, point piercing target",
    "melee_mace_blunt":
        "blunt metal mace hit on armor, heavy low thump with metallic clank",
    "hit_armor_metal_light":
        "light impact on metal armor, small shield tap, subtle metallic ring",
    "hit_armor_metal_heavy":
        "strong metal impact on armor or shield, loud clang and ring, gladiator blocking attack",
    "hit_flesh_light":
        "light flesh hit, muted wet thud, little or no armor",
    "hit_flesh_heavy":
        "heavy body impact with flesh and bone, thick low frequency thump, possible gore",
    "shield_block_wood":
        "wooden shield block, woody thud with small rattle, roman scutum",
    "shield_block_metal":
        "metal shield block, plate clash, ringing edge",
    "footstep_sand":
        "human footsteps on sand or dirt arena floor, soft granular scrapes",
    "footstep_stone":
        "sandaled footsteps on stone, hard clicks and small reverbs, coliseum corridors",
    "vocal_battle_shout":
        "single male gladiator battle shout or yell, aggressive voice, medium reverb",
    "vocal_pain_cry":
        "human pain scream or grunt, hurt vocalization in combat",
    "crowd_cheer_large":
        "large ancient roman coliseum crowd cheering, wide stereo ambience, wall of voices",
    "crowd_boo":
        "crowd booing or disapproving, descending vocal reactions",
    "crowd_murmur_idle":
        "large crowd idle murmur, distant talking ambience, no big cheer",
    "environment_fire":
        "fire or torches burning, crackling flames, medieval ambience",
    "environment_chain_rattle":
        "chains rattling, metal links clanking, dungeon like",
    "ui_menu":
        "short clean user interface click or confirm sound, non diegetic, no ambience",
    "ui_error":
        "short clean user interface error or cancel sound, non diegetic",
}

# ---------- MODEL LOAD ----------

device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

print("Loading CLAP model...")
processor = AutoProcessor.from_pretrained(MODEL_ID)
model = ClapModel.from_pretrained(MODEL_ID).to(device)
model.eval()

label_names = list(GLADIATOR_LABELS.keys())
label_texts = [GLADIATOR_LABELS[k] for k in label_names]

# Pre-encode label texts once
with torch.no_grad():
    text_inputs = processor(text=label_texts, return_tensors="pt", padding=True)
    text_inputs = {k: v.to(device) for k, v in text_inputs.items()}
    text_features = model.get_text_features(**text_inputs)  # [num_labels, dim]
    text_features = torch.nn.functional.normalize(text_features, dim=-1)


def classify_audio(path: Path, top_k: int = 3):
    waveform, sr = torchaudio.load(str(path))

    # CLAP expects 48 kHz by default; resample if needed.:contentReference[oaicite:3]{index=3}
    target_sr = 48000
    if sr != target_sr:
        waveform = torchaudio.functional.resample(waveform, sr, target_sr)
        sr = target_sr

    # Convert to mono if multi-channel
    if waveform.shape[0] > 1:
        waveform = waveform.mean(dim=0, keepdim=True)

    audio_array = waveform.squeeze(0).numpy()

    with torch.no_grad():
        inputs = processor(audios=audio_array, return_tensors="pt", padding=True)
        inputs = {k: v.to(device) for k, v in inputs.items()}

        audio_features = model.get_audio_features(**inputs)  # [1, dim]
        audio_features = torch.nn.functional.normalize(audio_features, dim=-1)

        # cosine similarity ~ dot product after normalization
        sims = (audio_features @ text_features.T).squeeze(0)  # [num_labels]

        topk_vals, topk_idx = torch.topk(sims, k=top_k)
        results = []
        for score, idx in zip(topk_vals.tolist(), topk_idx.tolist()):
            results.append((label_names[idx], float(score)))
        return results


def iter_wavs(root: Path):
    for dirpath, _, filenames in os.walk(root):
        for fn in filenames:
            if fn.lower().endswith((".wav", ".flac", ".ogg")):
                yield Path(dirpath) / fn


def main():
    root = Path(AUDIO_DIR)
    rows = []

    for audio_path in iter_wavs(root):
        rel = audio_path.relative_to(root)
        print(f"Processing {rel}...")
        try:
            tags = classify_audio(audio_path, top_k=TOP_K)
        except Exception as e:
            print(f"  ERROR on {rel}: {e}")
            continue

        row = {
            "relative_path": str(rel),
        }
        for i, (label, score) in enumerate(tags, start=1):
            row[f"tag_{i}"] = label
            row[f"score_{i}"] = f"{score:.4f}"

        rows.append(row)

    fieldnames = ["relative_path"]
    for i in range(1, TOP_K + 1):
        fieldnames += [f"tag_{i}", f"score_{i}"]

    with open(OUTPUT_CSV, "w", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(rows)

    print(f"Wrote {len(rows)} rows to {OUTPUT_CSV}")


if __name__ == "__main__":
    main()
```

This is fully local: models and audio stay on your machine.

---

## 5. Optional: add a local LLM for nicer descriptions

If you later want GPT-style prose like:

> “Use this as a heavy shield-block for elite gladiators; map to high-tier block skill with longer reverb.”

You can:

1. Install a small local LLM via e.g. Ollama or LM Studio (Llama-family, Qwen, Mistral, etc.).([n8n Blog][3])
2. For each file, pass:

   * File name
   * Top-K labels from CLAP
   * Any extra desired metadata
3. Ask the LLM to generate:

   * A one-line design description.
   * Suggested game tags (`Category`, `SubCategory`, `Intensity`, `Distance`, `Layer`).

You can merge that back into the same CSV.

---

## 6. Node.js option (if you prefer JS)

Transformers.js exposes an `audio-classification` pipeline that runs entirely in Node and can load compatible audio models.([Hugging Face][4])

Basic shape:

```js
import { pipeline } from '@xenova/transformers';

const classifier = await pipeline(
  'audio-classification',
  'Xenova/wav2vec2-large-xlsr-53' // or another audio model
);

const out = await classifier('path/to/sound.wav');
// out = [ { label: '...', score: 0.97 }, ... ]
```

For generic SFX, CLAP-style models usually work better than speech ASR models, so Python + CLAP is the more direct fit right now.

---

## 7. Bringing tags into Unreal

Typical Unreal workflow:

1. Put `sound_tags.csv` next to your project.
2. Create a UStruct (e.g. `FTaggedSoundRow`) with:

   * `FName RelativePath`
   * `FName Tag1`, `float Score1`, etc.
3. Create a Data Table asset using that UStruct, import the CSV.
4. At load time or in your content pipeline tools:

   * Match `RelativePath` to `USoundWave` assets.
   * Push tags into:

     * Asset naming conventions.
     * `Metadata` or a custom `UDataAsset` map.
     * Gameplay tag containers for runtime sound selection.

If you want, I can next write a variant of the script that emits a CSV layout tailored exactly to a specific Unreal UStruct you define.
```js
[1]: https://arxiv.org/html/2211.06687v4?utm_source=chatgpt.com "Large-scale Contrastive Language-Audio Pretraining with ..."
[2]: https://aidoczh.com/hf/docs/transformers/v4.47.1/en/model_doc/clap.html?utm_source=chatgpt.com "CLAP"
[3]: https://blog.n8n.io/open-source-llm/?utm_source=chatgpt.com "The 11 best open-source LLMs for 2025"
[4]: https://huggingface.co/docs/transformers.js/en/api/pipelines?utm_source=chatgpt.com "pipelines"
```
**Q: How to download Mistral3 to generate Game taxonomy tags?**

**A:** Use a small Ministral-3 “Instruct” model locally (8B is a good sweet spot) and call it from Python to generate tags.

Below are two practical ways on Windows 11.

---

## 1. Easiest: Ollama + Ministral-3 (recommended)

Ministral-3 (3B / 8B / 14B, Base/Instruct/Reasoning) is published as open-weight edge models with vision, available via Ollama and Hugging Face.([Mistral AI][1])
Ollama exposes a local HTTP API that you can call from Python.

### 1.1 Install Ollama on Windows

1. Go to the Ollama site and install the Windows build (standard installer).([apidog][2])
2. After install, open PowerShell and confirm:

```powershell
ollama --version
```

### 1.2 Pull a Ministral-3 model

For taxonomy/tagging, an 8B “Instruct” quantized model is a good balance of quality vs RAM (~6–10 GB depending on quantization).([Hugging Face][3])

In PowerShell:

```powershell
# Lightweight 8B instruct, Q4_K_M quantization
ollama pull ministral-3:8b-instruct-2512-q4_K_M
```

Ollama’s library shows all Ministral-3 tags (3B/8B/14B, various quantizations).([Ollama][4])

You can test it interactively:

```powershell
ollama run ministral-3:8b-instruct-2512-q4_K_M
```

Type a prompt, Ctrl+C to exit.

### 1.3 Call Ministral-3 from Python to generate game tags

Ollama starts a local HTTP server (default `http://127.0.0.1:11434`). Minimal Python example:

```python
import json
import requests

MODEL = "ministral-3:8b-instruct-2512-q4_K_M"
OLLAMA_URL = "http://127.0.0.1:11434/api/chat"

def generate_tags(sound_name: str, description: str):
    system = (
        "You are a game audio tagging assistant. "
        "Given a sound description for a Roman gladiator 3rd-person MMORPG, "
        "output ONLY a compact JSON array of tags, no prose. "
        "Tags should be short identifiers like "
        "\"melee_sword_slash_heavy\", \"shield_block_wood\", "
        "\"crowd_cheer_large\", \"ui_menu\", etc."
    )

    user = f"Sound name: {sound_name}\nDescription: {description}\nTags:"

    payload = {
        "model": MODEL,
        "messages": [
            {"role": "system", "content": system},
            {"role": "user", "content": user},
        ],
        "stream": False,
    }

    resp = requests.post(OLLAMA_URL, json=payload, timeout=120)
    resp.raise_for_status()
    content = resp.json()["message"]["content"].strip()

    # Model should return something like: ["melee_sword_slash_heavy", "hit_armor_metal_heavy"]
    try:
        tags = json.loads(content)
    except json.JSONDecodeError:
        # fallback: you can log or manually inspect content
        tags = [content]

    return tags


if __name__ == "__main__":
    tags = generate_tags(
        sound_name="metal_hit_03",
        description="Heavy metal sword impact on armor, loud clang, short tail"
    )
    print(tags)
```

Integrate this with your CLAP pipeline by feeding in the CLAP-derived text labels or manual descriptions and standardizing on a game-specific tag vocabulary.

---

## 2. Direct Python download via Hugging Face (no Ollama)

If you prefer to run the model directly in Python:

1. Install recent `transformers`, `torch`, and `accelerate`. Mistral 3/Ministral family is supported via the latest Transformers v5 line.([Unsloth Docs][5])

   ```bash
   pip install --upgrade "transformers" "accelerate" "torch"
   ```

2. Use the official Ministral-3 Instruct checkpoint, e.g. 8B:([Hugging Face][6])

   ```python
   from transformers import AutoModelForCausalLM, AutoTokenizer
   import torch

   model_id = "mistralai/Ministral-3-8B-Instruct-2512"

   tokenizer = AutoTokenizer.from_pretrained(model_id)
   model = AutoModelForCausalLM.from_pretrained(
       model_id,
       torch_dtype=torch.float16 if torch.cuda.is_available() else torch.float32,
       device_map="auto"
   )

   def llm_tags(prompt: str, max_new_tokens: int = 128):
       inputs = tokenizer(prompt, return_tensors="pt").to(model.device)
       with torch.no_grad():
           out = model.generate(
               **inputs,
               max_new_tokens=max_new_tokens,
               do_sample=False,
           )
       text = tokenizer.decode(out[0], skip_special_tokens=True)
       return text

   prompt = (
       "You are a game audio tagging assistant for a Roman gladiator MMORPG.\n"
       "Sound: heavy shield block with metal clang, close and dry.\n"
       "Return ONLY a JSON array of tags:\n"
   )

   print(llm_tags(prompt))
   ```

For Windows 11 CPU-only, this will be slow at BF16; you’d typically want a quantized GGUF variant (e.g. the official GGUF release) and run it via `llama.cpp`, LM Studio, or Ollama.([Hugging Face][3])

---

## 3. Which variant to choose for your use case

For your Unreal 3rd-person gladiator MMORPG taxonomy:

* Use **Ministral-3 8B Instruct** (Ollama `ministral-3:8b-instruct-2512-q4_K_M` or HF `Ministral-3-8B-Instruct-2512`) as default.
* Drop to **3B** if you want very low memory usage.
* Move to **14B** if you want slightly richer reasoning or longer prompts and have the VRAM/RAM headroom.([NVIDIA Developer][7])

If you want, I can next sketch the exact JSON schema for your “game taxonomy tags” (e.g., `category`, `weapon_type`, `impact_intensity`, `distance`, `diegetic_layer`) and adapt the prompt around that.
```js
[1]: https://mistral.ai/news/mistral-3?utm_source=chatgpt.com "Introducing Mistral 3"
[2]: https://apidog.com/blog/run-mistral-3-locally/?utm_source=chatgpt.com "How to Run Mistral 3 Locally"
[3]: https://huggingface.co/mistralai/Ministral-3-8B-Instruct-2512-GGUF?utm_source=chatgpt.com "mistralai/Ministral-3-8B-Instruct-2512-GGUF"
[4]: https://ollama.com/library/ministral-3/tags?utm_source=chatgpt.com "Tags · ministral-3"
[5]: https://docs.unsloth.ai/new/ministral-3?utm_source=chatgpt.com "Ministral 3: How to Run & Fine-tune"
[6]: https://huggingface.co/mistralai/Ministral-3-8B-Instruct-2512?utm_source=chatgpt.com "mistralai/Ministral-3-8B-Instruct-2512"
[7]: https://developer.nvidia.com/blog/nvidia-accelerated-mistral-3-open-models-deliver-efficiency-accuracy-at-any-scale/?utm_source=chatgpt.com "NVIDIA-Accelerated Mistral 3 Open Models Deliver ..."
```
**Q: gpt-oss:120b is available!**

**A:** Use `gpt-oss:120b` as a drop-in replacement for Ministral-3 in the “LLM tagging” stage: keep CLAP (or any audio model) to extract semantics from WAVs, then ask `gpt-oss:120b` to map those semantics into your gladiator/MMORPG taxonomy.

Below is a concrete way to wire it up assuming `gpt-oss:120b` is exposed via an Ollama-style HTTP API.

---

## 1. High-level pipeline

1. Audio → semantics:

   * Use CLAP or another audio-text model to get a short caption or N generic labels per file (you already have that pattern).
2. Semantics → game taxonomy:

   * For each sound, send a compact prompt to `gpt-oss:120b`:

     * Sound filename
     * Short description / CLAP labels
     * Fixed JSON schema for your tags
   * Parse the JSON, write it into your CSV / Unreal DataTable.

`gpt-oss:120b` is overkill for this, which is good: you can afford strict structured output and richer, game-aware tags.

---

## 2. Example JSON schema for tags

You can tune these fields, but this is a good starting point for a Roman gladiator arena:

```json
{
  "category": "melee | ranged | movement | crowd | environment | ui",
  "weapon_type": "sword | spear | mace | shield | unarmed | none",
  "impact_intensity": "none | light | medium | heavy",
  "material_primary": "metal | flesh | wood | stone | sand | cloth | mixed",
  "distance": "close | mid | far",
  "layer": "diegetic | ui | ambience",
  "emotion": "neutral | tense | triumphant | fearful | brutal",
  "suggested_tags": [
    "melee_sword_slash_heavy",
    "hit_armor_metal_heavy",
    "gladiator_elite"
  ]
}
```

---

## 3. Python: calling `gpt-oss:120b` via HTTP

This assumes a local chat endpoint identical to Ollama’s (`POST /api/chat`), with the model name `gpt-oss:120b`.

```python
import json
import requests

MODEL = "gpt-oss:120b"
API_URL = "http://127.0.0.1:11434/api/chat"  # adjust if different

SYSTEM_PROMPT = """
You are a game audio taxonomy engine for a 3rd person MMORPG about Roman gladiators and the Coliseum.

Given:
- a sound asset name
- one or more short descriptions of how the sound actually feels/behaves

You MUST output ONLY valid JSON (no comments, no prose) matching this schema:

{
  "category": "melee | ranged | movement | crowd | environment | ui",
  "weapon_type": "sword | spear | mace | shield | unarmed | none",
  "impact_intensity": "none | light | medium | heavy",
  "material_primary": "metal | flesh | wood | stone | sand | cloth | mixed",
  "distance": "close | mid | far",
  "layer": "diegetic | ui | ambience",
  "emotion": "neutral | tense | triumphant | fearful | brutal",
  "suggested_tags": ["kebab_case_tag", "..."]
}

- Choose the single best value for each scalar field.
- suggested_tags: 3–7 short identifiers in kebab_case or snake_case.
"""

def query_gpt_oss(sound_name: str,
                  descriptions: list[str],
                  extra_context: str | None = None) -> dict:
    desc_block = "\n".join(f"- {d}" for d in descriptions)
    user_prompt = (
        f"Sound asset name: {sound_name}\n"
        f"Descriptions:\n{desc_block}\n"
    )
    if extra_context:
        user_prompt += f"\nExtra context: {extra_context}\n"

    payload = {
        "model": MODEL,
        "messages": [
            {"role": "system", "content": SYSTEM_PROMPT},
            {"role": "user", "content": user_prompt},
        ],
        "stream": False,
    }

    resp = requests.post(API_URL, json=payload, timeout=300)
    resp.raise_for_status()
    content = resp.json()["message"]["content"].strip()

    # Strict JSON expectation; if it fails you can add a repair step
    return json.loads(content)


if __name__ == "__main__":
    # Example: use CLAP output as descriptions
    sound_name = "metal_hit_gladiator_03"
    clap_labels = [
        "heavy metal impact on armor",
        "short ringing clang, close perspective",
        "combat hit, not ambient"
    ]

    result = query_gpt_oss(sound_name, clap_labels)
    print(json.dumps(result, indent=2))
```

You can integrate this into your existing CLAP tagging script by:

* After computing CLAP top-K labels / captions for each file, call `query_gpt_oss`.
* Merge the returned JSON into your CSV row:

```csv
relative_path,category,weapon_type,impact_intensity,material_primary,distance,layer,emotion,suggested_tags
SFX/weapons/metal_hit_gladiator_03.wav,melee,sword,heavy,metal,close,diegetic,brutal,"['melee_sword_slash_heavy','hit_armor_metal_heavy','gladiator_elite']"
```

That CSV can then be imported into Unreal as a DataTable and used to drive:

* Sound cue selection logic.
* Gameplay tags on `USoundWave` or wrapper `UDataAsset` objects.
* Content browser filters (via tags or metadata).

---

## 4. If `gpt-oss:120b` is served differently

If your `gpt-oss:120b` instance is:

* vLLM / TGI / custom FastAPI:

  * Replace the `API_URL` and payload shape with whatever that service expects (usually a `/generate` or `/v1/chat/completions`-like endpoint).
* Node-based:

  * Same idea, but call from Node and write out JSON/CSV for Unreal.

The core pattern stays identical: audio model → short descriptors → `gpt-oss:120b` → strict JSON tags.

**Q: gpt-oss will be downloaded.**

**A:** Once `gpt-oss:120b` is downloaded and served, the next step is to wire it into your tagging pipeline as the “semantic → game taxonomy” stage.

Below is a concrete plan and minimal code you can drop into your existing Python CLAP script.

---

## 1. Assumptions

I will assume:

* `gpt-oss:120b` is exposed via an OpenAI-compatible Chat Completions API, e.g.:

  * Base URL: `http://127.0.0.1:8000/v1`
  * Model name: `gpt-oss-120b`

If your server exposes an Ollama-style `/api/chat` instead, you only need to swap the call; the prompt and JSON contract stay the same.

---

## 2. Taxonomy schema (fixed contract to the model)

Define a compact JSON schema that will be stable across the project:

```json
{
  "category": "melee | ranged | movement | crowd | environment | ui",
  "weapon_type": "sword | spear | mace | shield | unarmed | none",
  "impact_intensity": "none | light | medium | heavy",
  "material_primary": "metal | flesh | wood | stone | sand | cloth | mixed",
  "distance": "close | mid | far",
  "layer": "diegetic | ui | ambience",
  "emotion": "neutral | tense | triumphant | fearful | brutal",
  "suggested_tags": ["kebab_case_tag", "..."]
}
```

You will ask `gpt-oss` to return only valid JSON in exactly this shape.

---

## 3. Python client for `gpt-oss:120b` (OpenAI-compatible)

Install the OpenAI Python client:

```bash
pip install --upgrade openai
```

Minimal wrapper:

```python
from openai import OpenAI
import json

# Adjust to your local gpt-oss deployment
client = OpenAI(
    base_url="http://127.0.0.1:8000/v1",
    api_key="gpt-oss-local"  # or any dummy string if your server ignores it
)

SYSTEM_PROMPT = """
You are a game audio taxonomy engine for a 3rd person MMORPG about Roman gladiators and the Coliseum.

Given:
- a sound asset name
- one or more short descriptions of how the sound behaves in-game

You MUST output ONLY valid JSON (no comments, no prose) matching this schema:

{
  "category": "melee | ranged | movement | crowd | environment | ui",
  "weapon_type": "sword | spear | mace | shield | unarmed | none",
  "impact_intensity": "none | light | medium | heavy",
  "material_primary": "metal | flesh | wood | stone | sand | cloth | mixed",
  "distance": "close | mid | far",
  "layer": "diegetic | ui | ambience",
  "emotion": "neutral | tense | triumphant | fearful | brutal",
  "suggested_tags": ["kebab_case_tag", "..."]
}

Rules:
- Choose exactly one value for each scalar field.
- suggested_tags: 3 to 7 short identifiers in kebab_case or snake_case.
- Do not add extra fields.
"""

def classify_with_gpt_oss(sound_name: str, descriptions: list[str]) -> dict:
    desc_block = "\n".join(f"- {d}" for d in descriptions)
    user_prompt = (
        f"Sound asset name: {sound_name}\n"
        f"Descriptions:\n{desc_block}\n"
        "Return only JSON:\n"
    )

    resp = client.chat.completions.create(
        model="gpt-oss-120b",
        messages=[
            {"role": "system", "content": SYSTEM_PROMPT},
            {"role": "user", "content": user_prompt},
        ],
        temperature=0.0,
        max_tokens=256,
    )

    content = resp.choices[0].message.content.strip()

    # Expect strict JSON; if not, you can add a repair step later
    return json.loads(content)
```

---

## 4. Integrate with your CLAP audio pipeline

Assume your CLAP pipeline already computes, for each file:

* `relative_path`: path to the WAV
* `clap_tags`: list of `(label, score)` pairs
* Optionally, a simple free-text caption

You can convert the CLAP output into condensed descriptions and feed to `gpt-oss`:

```python
def build_descriptions_from_clap(clap_tags: list[tuple[str, float]]) -> list[str]:
    # Example: keep top 3 CLAP labels and embed score in text
    descs = []
    for label, score in clap_tags[:3]:
        descs.append(f"{label.replace('_', ' ')} (confidence {score:.2f})")
    return descs
```

Inside your main loop over WAV files:

```python
rows = []

for audio_path in iter_wavs(root):
    rel = audio_path.relative_to(root)
    print(f"Processing {rel}...")

    try:
        clap_results = classify_audio(audio_path, top_k=3)  # from your CLAP code
        descriptions = build_descriptions_from_clap(clap_results)

        gpt_meta = classify_with_gpt_oss(
            sound_name=str(rel),
            descriptions=descriptions
        )

    except Exception as e:
        print(f"  ERROR on {rel}: {e}")
        continue

    # Flatten JSON into CSV row
    row = {
        "relative_path": str(rel),
        "category": gpt_meta["category"],
        "weapon_type": gpt_meta["weapon_type"],
        "impact_intensity": gpt_meta["impact_intensity"],
        "material_primary": gpt_meta["material_primary"],
        "distance": gpt_meta["distance"],
        "layer": gpt_meta["layer"],
        "emotion": gpt_meta["emotion"],
        "suggested_tags": ",".join(gpt_meta["suggested_tags"]),
    }

    rows.append(row)
```

Then write `rows` to `sound_taxonomy.csv` and import it into Unreal as a Data Table keyed by `relative_path`.

---

## 5. If your `gpt-oss` server is Ollama-style instead

If instead you are exposing `gpt-oss:120b` as an Ollama `/api/chat` model:

* Replace the OpenAI call with:

```python
import requests

def classify_with_gpt_oss_ollama(sound_name: str, descriptions: list[str]) -> dict:
    desc_block = "\n".join(f"- {d}" for d in descriptions)
    user_prompt = (
        f"Sound asset name: {sound_name}\n"
        f"Descriptions:\n{desc_block}\n"
        "Return only JSON:\n"
    )

    payload = {
        "model": "gpt-oss:120b",
        "messages": [
            {"role": "system", "content": SYSTEM_PROMPT},
            {"role": "user", "content": user_prompt},
        ],
        "stream": False,
    }

    resp = requests.post("http://127.0.0.1:11434/api/chat", json=payload, timeout=600)
    resp.raise_for_status()
    content = resp.json()["message"]["content"].strip()
    return json.loads(content)
```

Everything else in the pipeline is unchanged.

---

If you want, I can next give you a concrete Unreal Engine `USTRUCT` + Data Table layout that matches this JSON/CSV so you can use the taxonomy at runtime for sound selection and gameplay tags.
