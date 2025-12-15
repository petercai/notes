Good question. Since you care about **voice-cloning + offline TTS** in **Chinese**, and want support for **CPU, CUDA, MPS** (i.e. flexibility in hardware), I can highlight a few of the **most promising engines (as of 2025)** that are often considered *stronger or more flexible than VoxCPM — depending on what you care about (pronunciation accuracy, voice-cloning fidelity, latency, control, etc.).

---

## 🔎 Top voice-cloning / offline-TTS engines to try (2025)

|Model / Engine|What it's especially good at / Why worth trying (vs VoxCPM)|
|---|---|
|**IndexTTS2**|- Zero-shot voice cloning (i.e. clone from a short reference). ([TTS Wiki](https://tts.wiki/index.php/IndexTTS2?utm_source=chatgpt.com "IndexTTS2 - TTS Wiki")) - Explicit support for Chinese (and English, Japanese), with a **character + pinyin hybrid modeling** method — helps handle polyphonic Chinese characters and improves pronunciation accuracy. ([arXiv](https://arxiv.org/abs/2502.05512?utm_source=chatgpt.com "IndexTTS: An Industrial-Level Controllable and Efficient Zero-Shot Text-To-Speech System")) - Uses a strong vocoder (e.g. BigVGAN2) + conformer-based encoder → yields more natural and clear voice output. ([GitHub](https://github.com/index-tts/index-tts?utm_source=chatgpt.com "index-tts/index-tts: An Industrial-Level Controllable and ...")) - Allows controllable parameters: emotion, pause/duration, prosody — useful for dubbing or expressive speech. ([GitHub](https://github.com/index-tts/index-tts?utm_source=chatgpt.com "index-tts/index-tts: An Industrial-Level Controllable and ..."))|
|**CosyVoice2** (or “CosyVoice”)|- Supports multilingual TTS including Chinese, and optionally various dialects. ([SiliconFlow](https://www.siliconflow.com/articles/en/best-open-source-text-to-speech-models?utm_source=chatgpt.com "The Best Open Source Text-to-Speech Models in 2025")) - Zero-shot voice cloning and real-time (or low-latency) synthesis — helpful if you need interactive use rather than long batch renders. ([CosyVoice](https://cosyvoice.org/?utm_source=chatgpt.com "CosyVoice \| Multilingual TTS Model")) - Good tradeoff between quality and speed/latency (e.g. low first-packet latency). ([SiliconFlow](https://www.siliconflow.com/articles/en/best-open-source-text-to-speech-models?utm_source=chatgpt.com "The Best Open Source Text-to-Speech Models in 2025"))|
|**MiniMax‑Speech**|- Newer autoregressive TTS model with a learnable speaker encoder: achieves high fidelity in voice-cloning (timbre matches reference well). ([arXiv](https://arxiv.org/abs/2505.07916?utm_source=chatgpt.com "MiniMax-Speech: Intrinsic Zero-Shot Text-to-Speech with a Learnable Speaker Encoder")) - Supports **many languages** (32 languages claimed), which suggests potential for Chinese + mixed language usage (though I’d check whether there’s a good Chinese voice example for your target). ([arXiv](https://arxiv.org/abs/2505.07916?utm_source=chatgpt.com "MiniMax-Speech: Intrinsic Zero-Shot Text-to-Speech with a Learnable Speaker Encoder")) - Because of modern design, might offer a good balance between voice quality, expressiveness, and flexibility for offline / custom use. ([arXiv](https://arxiv.org/abs/2505.07916?utm_source=chatgpt.com "MiniMax-Speech: Intrinsic Zero-Shot Text-to-Speech with a Learnable Speaker Encoder"))|

---

## ✅ Why these may outperform VoxCPM (for _your_ use case)

- **Pronunciation & language-specific optimization**: IndexTTS2’s hybrid character + pinyin modeling is a big advantage for Chinese — helps avoid the common problem of mispronouncing polyphonic characters.
    
- **Fine-grained control**: When you need more control over emotion, pacing, pause/timing (e.g. for dubbing, narration, voice-overs), IndexTTS2 and CosyVoice2 give more control than many “general” TTS systems.
    
- **Flexibility in hardware & deployment**: Because these models are open-source and (in many cases) efficiently implemented, you’ll have a better shot at running on CPU / GPU / MPS depending on what you have — which matters for offline or private use.
    
- **Voice-cloning fidelity**: Especially for “clone from a short sample → produce speech with similar timbre/voice quality,” newer models like MiniMax-Speech or IndexTTS2 may have improved over earlier ones (and possibly over VoxCPM depending on the sample quality).
    

---

## ⚠️ Trade-offs / What to watch out for

- **Setup complexity & dependencies**: Especially for models with custom vocoders / encoders (BigVGAN2, conformer, hybrid-pinyin logic), installation and inference configuration might require more work than a “plug-and-play” model.
    
- **Voice-cloning consistency depends heavily on reference audio** — like with any TTS/cloning: a clean, well-recorded reference helps a lot.
    
- **Model limitations**: Even the best open-source TTS might still struggle with extremely expressive speech, very long passages, or idiosyncratic accents/voices.
    
- **Licensing / usage constraints**: Always good to check licences if you plan to use it commercially.
    

---

## 🎯 My recommendation for you (based on what you told me)

Since your focus is **Chinese voice-cloning, offline usage, and quality close to original voice**, I’d recommend you **start with IndexTTS2** — try it with several reference samples, tune prosody/emotion, and compare results. If you want lower latency / more interactive or “assistant-style” usage (or test a different voice style), also try **CosyVoice2**. If you want to experiment with newer architectures and maybe even multilingual or cross-language voice-cloning, give **MiniMax-Speech** a try.

---

If you like — I can also **pull up 5–10 publicly available sample voices (Chinese as well as multilingual) for these models** (IndexTTS2, CosyVoice2, MiniMax…) — you can directly download them and test cloning quality without recording your own voice. Would you like me to build that sample-voice list for you now?