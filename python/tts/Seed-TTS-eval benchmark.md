Seed-TTS-eval measures both **speech intelligibility** (accuracy of generated text) and **speaker similarity** (how close the voice sounds to the reference).  
It reports separate metrics for **English**, **Chinese**, and **Hard** (mixed or difficult) test sets.

---

### 🧩 1. `test-EN`: English benchmark

| Metric      | Meaning                | Description                                                                                                                                                                                                                                     |
| ----------- | ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **WER / %** | **Word Error Rate**    | Measures how accurately the generated English speech can be transcribed back to text by an ASR model. Lower is better. Formula: (\text{WER} = \frac{S + D + I}{N} \times 100%) where S=Substitutions, D=Deletions, I=Insertions, N=total words. |
| **SIM / %** | **Speaker Similarity** | Percentage similarity between the generated voice and the reference speaker. Typically computed using speaker embeddings (e.g., cosine similarity × 100). Higher is better.                                                                     |

---

### 🇨🇳 2. `test-ZH`: Chinese benchmark

| Metric      | Meaning                  | Description                                                                                                                                        |
| ----------- | ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **CER / %** | **Character Error Rate** | Like WER but for Chinese characters (since Chinese doesn’t have spaces). Lower is better. Formula: (\text{CER} = \frac{S + D + I}{N} \times 100%). |
| **SIM / %** | **Speaker Similarity**   | Same as above — similarity between synthesized and real speaker voices, expressed as a percentage. Higher means more similar.                      |

---

### ⚙️ 3. `test-Hard`: Difficult benchmark

| Metric      | Meaning                  | Description                                                                                                                        |
| ----------- | ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| **CER / %** | **Character Error Rate** | Measures intelligibility on challenging inputs (e.g., noisy text, code-switching, rare words, or emotional tone). Lower is better. |
| **SIM / %** | **Speaker Similarity**   | Voice consistency under difficult conditions — tests model robustness. Higher is better.                                           |

---

### ✅ Summary

| Test Set      | Intelligibility Metric | Similarity Metric | Ideal Direction       |
| ------------- | ---------------------- | ----------------- | --------------------- |
| **test-EN**   | WER ↓                  | SIM ↑             | Lower WER, higher SIM |
| **test-ZH**   | CER ↓                  | SIM ↑             | Lower CER, higher SIM |
| **test-Hard** | CER ↓                  | SIM ↑             | Lower CER, higher SIM |

---

**In short:**

- **WER/CER** → how understandable the speech is.

- **SIM** → how similar the generated voice is to the reference.

- **test-Hard** → stress test for robustness.

Would you like me to show how these metrics are actually computed from model outputs (e.g., via `whisper` + `speaker encoder` pipeline)?

# CV3-Eval Benchmark (or “CV3-Eval”) in TTS

---

## 🔍 Clues from “Eliminating stability hallucinations in LLM-based TTS” paper

- The authors use **CV3-Eval** as one of the test sets (along with Seed-TTS-Eval) to evaluate **stability hallucinations** — i.e. unwanted repetition, omissions, or misalignment in synthesized speech. ([arXiv](https://arxiv.org/abs/2509.19852?utm_source=chatgpt.com "Eliminating stability hallucinations in llm-based tts models via attention guidance"))

- They propose a metric called **Optimal Alignment Score (OAS)** to evaluate the alignment between text tokens and speech tokens. They show that integrating OAS into training helps reduce stability hallucinations when tested on Seed-TTS-Eval and **CV3-Eval**. ([arXiv](https://arxiv.org/abs/2509.19852?utm_source=chatgpt.com "Eliminating stability hallucinations in llm-based tts models via attention guidance"))

So CV3-Eval likely focuses on **robustness / stability** aspects, especially alignment fidelity, in TTS models under challenging input/text conditions.

---

## 🛠 What CV3-Eval probably measures / contains

Based on the context, here’s a plausible breakdown of what CV3-Eval might involve:

| Characteristic                        | Likely Purpose                                                                                                                       | Notes / Speculation                                                                                                                                                                                                   |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Stability / Hallucination test**    | To detect issues such as repetitions, skips, or misalignments in TTS output                                                          | Because the paper uses CV3-Eval specifically in that context ([arXiv](https://arxiv.org/abs/2509.19852?utm_source=chatgpt.com "Eliminating stability hallucinations in llm-based tts models via attention guidance")) |
| **Token-level text–speech alignment** | To evaluate how well the model follows the input text in correct order, with minimal drift                                           | The paper’s OAS metric is about text–speech alignment ([arXiv](https://arxiv.org/abs/2509.19852?utm_source=chatgpt.com "Eliminating stability hallucinations in llm-based tts models via attention guidance"))        |
| **Diverse / difficult inputs**        | Likely includes challenging sentences (e.g. long sentences, nested structure, unusual tokens)                                        | To stress test stability                                                                                                                                                                                              |
| **Automatic metrics**                 | Probably uses automatic metrics (like OAS, error rates) rather than purely human evaluation                                          | In line with the trend toward more automatic benchmarks                                                                                                                                                               |
| **Complement to Seed-TTS-Eval**       | While Seed-TTS-Eval probably covers common error and similarity metrics, CV3-Eval is more about durability under edge / stress cases | That seems to be how it's being used in the mentioned paper                                                                                                                                                           |

In **CV3-Eval**, **“DNSMOS”** is used as an **audio quality metric** (non-intrusive objective score) to assess synthesized speech quality (correlating with human perception) in both Hard-ZH and Hard-EN subsets. ([GitHub](https://github.com/FunAudioLLM/CV3-Eval?utm_source=chatgpt.com "FunAudioLLM/CV3-Eval"))

Here’s more detail:

---

## 🔍 What is DNSMOS

- DNSMOS stands for **Deep Noise Suppression Mean Opinion Score**. ([arXiv](https://arxiv.org/pdf/2010.15258?utm_source=chatgpt.com "arXiv:2010.15258v2 [cs.SD] 10 Feb 2021"))

- It is a **non-intrusive perceptual objective metric** that estimates speech quality without requiring a clean reference signal. ([arXiv](https://arxiv.org/pdf/2010.15258?utm_source=chatgpt.com "arXiv:2010.15258v2 [cs.SD] 10 Feb 2021"))

- DNSMOS was originally proposed to evaluate noise suppression systems, but here it’s repurposed to score TTS output in CV3-Eval, treating degraded or synthesized speech like “noisy” input to assess overall perceptual quality. ([GitHub](https://github.com/FunAudioLLM/CV3-Eval?utm_source=chatgpt.com "FunAudioLLM/CV3-Eval"))

- In particular, CV3-Eval uses DNSMOS to capture _audio quality_ complementing content consistency (WER/CER) and speaker similarity (SIM). ([GitHub](https://github.com/FunAudioLLM/CV3-Eval?utm_source=chatgpt.com "FunAudioLLM/CV3-Eval"))

---

## ✅ Role of DNSMOS in Hard-ZH and Hard-EN

- In the **Hard** subsets (Hard-ZH, Hard-EN), the input text is more challenging (e.g. complicated expressions, rare words, noisy transcripts), so performance is evaluated under stress conditions. ([arXiv](https://arxiv.org/pdf/2509.19852?utm_source=chatgpt.com "Eliminating stability hallucinations in llm-based tts models ..."))

- For those subsets, CV3-Eval computes:
  
  1. **Content consistency** (via CER for Chinese, WER for English)
  
  2. **Speaker similarity** (SIM)
  
  3. **DNSMOS** as the audio quality measure

- DNSMOS thus gives a scalar “quality score” that is expected to correlate well with human listening judgments, even when there is no clean reference. In the CV3-Eval benchmark, comparing the DNSMOS across models helps understand which model yields more natural and perceptually pleasing audio in the difficult (“hard”) settings. ([GitHub](https://github.com/FunAudioLLM/CV3-Eval?utm_source=chatgpt.com "FunAudioLLM/CV3-Eval"))

---

If you like, I can pull up the mathematical definition of DNSMOS or its variants, or show example DNSMOS values for different TTS models in the Hard subsets. Do you want me to fetch that?

# how to compute the TTS metric

**how modern TTS benchmarks (like Seed-TTS-Eval or CV3-Eval)** actually **compute their metrics programmatically** from generated audio outputs.

---

## 🧠 Overview

When evaluating a TTS model, you typically have:

| Component          | Input                                       | Output                       |
| ------------------ | ------------------------------------------- | ---------------------------- |
| **TTS model**      | Text →                                      | **Generated speech (audio)** |
| **Reference data** | Ground-truth text + reference speaker audio | Used for metrics             |

After generating all outputs, evaluation scripts compute 3 main kinds of metrics:

1. **Intelligibility** → WER or CER

2. **Speaker similarity** → SIM (using speaker embeddings)

3. **Audio quality / robustness** → DNSMOS, MOSNet, etc.

---

## 🔹 1. WER (Word Error Rate) / CER (Character Error Rate)

**Purpose:** Measures **how correctly the generated speech matches the input text**.

### 🧩 Pipeline

```
[TTS output audio] → [ASR model (e.g., Whisper)] → [Predicted transcript]
→ Compare with → [Original text] → Compute WER or CER
```

### ⚙️ Implementation details

- **ASR backend:** OpenAI Whisper or WhisperX is typically used because it’s multilingual and robust.

- **WER formula:**  
    [  
    \text{WER} = \frac{S + D + I}{N}  
    ]  
    where
  
  - S = substitutions
  
  - D = deletions
  
  - I = insertions
  
  - N = number of words in reference text

- **CER formula (for Chinese):**  
    [  
    \text{CER} = \frac{S + D + I}{N_\text{chars}}  
    ]

- **Example (Python):**
  
  ```python
  import jiwer
  wer = jiwer.wer(ref_text, asr_text) * 100
  ```

---

## 🔹 2. SIM (Speaker Similarity)

**Purpose:** Measures **how similar the speaker’s voice is to the reference speaker**.

### 🧩 Pipeline

```
[Reference audio] → [Speaker encoder] → [Embedding A]
[TTS output audio] → [Speaker encoder] → [Embedding B]
→ Cosine similarity(A, B)
```

### ⚙️ Implementation details

- **Speaker encoder model:**  
    Commonly used ones include:
  
  - `Resemblyzer` (GE2E-based)
  
  - `SpeechBrain ECAPA-TDNN`
  
  - `WavLM`-based encoders

- **Cosine similarity formula:**  
    [  
    \text{SIM} = \frac{A \cdot B}{|A| |B|} \times 100  
    ]  
    → Values range roughly from 0% (completely different) to 100% (identical).

- **Example (Python):**
  
  ```python
  from scipy.spatial.distance import cosine
  sim = (1 - cosine(embed_ref, embed_tts)) * 100
  ```

---

## 🔹 3. test-Hard (Robustness)

**Purpose:** Evaluate model under **stressful conditions**, e.g.:

- Long or complex sentences

- Code-switching (mixing EN + ZH)

- Noisy or low-quality reference audio

- Emotion / punctuation / special tokens

The **same metrics (WER/CER, SIM)** are computed, but the **input dataset** is deliberately “harder”.  
This reveals stability, attention drift, and hallucination problems in the TTS model.

---

## 🔹 4. DNSMOS (Perceptual Audio Quality)

**Purpose:** Estimate **perceived naturalness / cleanness** of audio _without_ needing a reference.

### 🧩 Pipeline

```
[TTS output audio] → [Pretrained DNSMOS model] → [Quality score 1–5]
```

- Developed by Microsoft for Deep Noise Suppression Challenge.

- Uses a deep network trained to predict MOS (Mean Opinion Score) from noisy speech.

### ⚙️ Typical implementation:

```python
from dnsmos import compute_dnsmos
score = compute_dnsmos(audio_path)
```

Higher DNSMOS ≈ more natural / pleasant audio.

---

## 🔹 5. End-to-end Example Workflow (CV3-Eval-like)

```python
# 1. Run ASR to get transcript
asr_text = whisper.transcribe(audio_path)["text"]

# 2. Compute WER / CER
wer = jiwer.wer(reference_text, asr_text) * 100

# 3. Compute SIM
embed_ref = speaker_encoder(reference_audio)
embed_tts = speaker_encoder(audio_path)
sim = (1 - cosine(embed_ref, embed_tts)) * 100

# 4. Compute DNSMOS
dns_score = dnsmos(audio_path)

# Collect results
results = {"WER": wer, "SIM": sim, "DNSMOS": dns_score}
```

Then average all results across the dataset for **test-EN**, **test-ZH**, and **test-Hard** subsets.

---

## ✅ Summary Table

| Metric        | Computed via                      | Measures               | Ideal                       |
| ------------- | --------------------------------- | ---------------------- | --------------------------- |
| **WER / CER** | ASR (Whisper) + text alignment    | Intelligibility        | ↓ Lower                     |
| **SIM**       | Speaker encoder cosine similarity | Voice identity         | ↑ Higher                    |
| **DNSMOS**    | Non-intrusive neural estimator    | Audio naturalness      | ↑ Higher                    |
| **test-Hard** | Subset of harder texts            | Robustness / stability | N/A (compare across models) |

---

If you want, I can give you a **Python evaluation pipeline template** (using Whisper + Resemblyzer + DNSMOS) that reproduces the CV3-Eval-style metrics locally — would you like that