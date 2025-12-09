## OpenAI ChatGPT v1.2025.330 Unreal.Engine.WAV.Plugins.md

**Q: Can Unreal Engine process 32-bit float encoded audio?**

**A:** Yes, but with important constraints.

1. Import pipeline (SoundWave assets)

   * As of UE 5.7, the official “Importing Audio Files” docs list supported bit depths as **16-bit and 24-bit PCM only**. 32-bit float WAV is not listed as a supported import format. ([Epic Games Developers][1])
   * All imported audio (WAV/OGG/FLAC/etc.) is converted to **16-bit PCM `.wav` internally** when you import/cook. So even if a future version lets you drag in 32-bit float, it will be down-converted to 16-bit in the engine’s content pipeline. ([Epic Games Developers][1])
   * Community reports for UE4.27/UE5 show that 32-bit float WAV import either fails or is rejected as unsupported; you typically must convert to 16- or 24-bit integer first. ([Epic Developer Community Forums][2])

2. Internal mixing / runtime processing

   * The **Audio Mixer** runs internally on **32-bit float buffers**. Encoded sources (Ogg/Opus/etc.) are decoded to uncompressed **32-bit float** PCM, then sample-rate converted and mixed there. ([Epic Games Developers][3])
   * Audio capture devices (e.g., via `UAudioCaptureComponent` / AudioCaptureCore) can operate at **32-bit float** bit depth on the input side; some capture clients assume 32-bit float samples. ([Epic Games Developers][4])

3. Plugins / procedural paths

   * Plugins such as Runtime Audio Importer (and similar procedural `USoundWaveProcedural`-based systems) explicitly convert imported data to **32-bit IEEE float PCM** and feed that directly to the audio renderer. ([Georgy Dev Docs][5])

4. Practical summary

   * Unreal’s **runtime audio engine absolutely can process 32-bit float PCM** internally.
   * The **standard asset import pipeline does not preserve 32-bit float**; it expects 16-/24-bit integer and converts everything to 16-bit PCM for playback.
   * If you truly need an end-to-end 32-bit float workflow, you must use a procedural or plugin-based path (C++ / MetaSounds / Runtime Audio Importer) rather than standard `.wav` import.
```
[1]: https://dev.epicgames.com/documentation/en-us/unreal-engine/importing-audio-files?utm_source=chatgpt.com "Importing Audio Files | Unreal Engine 5.7 Documentation"
[2]: https://forums.unrealengine.com/t/32-bit-fp-wav-file-support/265889?utm_source=chatgpt.com "32-bit FP WAV File Support? - Audio"
[3]: https://dev.epicgames.com/documentation/en-us/unreal-engine/audio-mixer-overview-in-unreal-engine?utm_source=chatgpt.com "Audio Mixer Overview in Unreal Engine"
[4]: https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/AudioCaptureCore?utm_source=chatgpt.com "AudioCaptureCore | Unreal Engine 5.7 Documentation"
[5]: https://docs.georgy.dev/runtime-audio-importer/overview/?utm_source=chatgpt.com "Overview | Georgy Dev Docs"
```
