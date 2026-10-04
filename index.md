---
layout: default
title: Local Text to Speech
description: A local Kokoro script that turns pasted text into one MP3.
samwiki: true
---

<p class="sw-level sw-level-intermediate">
  <span class="sw-level-idx">Level 2</span>
  <span class="sw-level-name">Intermediate</span>
</p>

<section class="sw-lede" aria-labelledby="what-title">
  <div class="sw-lede-copy">
    <h2 id="what-title">Speech from text on your own machine</h2>
    <p>Local Text to Speech reads text you paste into the terminal and writes a single MP3. The engine is <a href="https://github.com/hexgrad/kokoro">Kokoro</a>, run locally through <code>KPipeline</code>. The voice is <code>af_bella</code>, language code <code>a</code> (American English), speed <code>1</code>. Each chunk of audio comes back as a NumPy array. The script concatenates those arrays, writes <code>temp_full.wav</code> at 24,000 Hz, and exports <code>output_full.mp3</code> at 192 kbps with pydub.</p>
    <p>It is for someone who wants spoken audio from their own text on a machine they administer. A draft, a study note, or a long passage goes in through standard input. The file that does the work is the script in this repository. Kokoro’s weights load in the same process that reads the text.</p>
    <p>SamWiki places this project at <strong>Level 2, intermediate</strong>. The control flow is linear: read until end of input, generate, join, export, play, and delete the temporary WAV. A finished MP3 depends on a local Python environment with Kokoro and its weights, NumPy, soundfile, and pydub, plus ffmpeg for the MP3 encode. Playback opens Windows Media Player with <code>start /min wmplayer</code>. The file the script is built to leave behind is <code>output_full.mp3</code>.</p>
    <p>Empty input stops the script at the prompt. If the generator returns no chunks, it exits before writing audio. After a successful export, <code>temp_full.wav</code> is removed and the MP3 remains.</p>
  </div>
  <aside class="sw-find" aria-labelledby="who-title">
    <h2 id="who-title">What you bring to it</h2>
    <ul>
      <li><strong>Pasted text</strong> Standard input until Ctrl+D (Unix) or Ctrl+Z then Enter (Windows), including blank lines and punctuation.</li>
      <li><strong>One voice</strong> <code>af_bella</code> at speed 1, American English.</li>
      <li><strong>One file out</strong> Chunks joined in order, then one 192 kbps MP3.</li>
      <li><strong>A local run</strong> Model load, synthesis, and the write happen in this process.</li>
      <li><strong>Level 2 · intermediate</strong> Python packaging and a local TTS model, with Windows as the playback target.</li>
    </ul>
  </aside>
</section>

## The script

```python
import sys
import os
import soundfile as sf
import numpy as np
from kokoro import KPipeline
from pydub import AudioSegment

# --- TEXT INPUT LOGIC ---
print("Enter/Paste your text. Press Ctrl+D (Unix) or Ctrl+Z (Windows) then Enter to save:")
# This reads everything including newlines, paragraphs, and punctuation until EOF
text = sys.stdin.read().strip()

if not text:
    print("Error: No text provided.")
    sys.exit()
# --------------------------------

print("--- Kokoro TTS ---")
print("Status: Loading Model...")
pipeline = KPipeline(lang_code='a') 

print("Status: Processing Text...")
generator = pipeline(text, voice='af_bella', speed=1)

# Create an empty list to hold all the generated audio chunks
audio_chunks = []

# Loop through the generator and collect the audio chunks
for i, (gs, ps, audio) in enumerate(generator):
    print(f"Status: Generating Part {i+1}...")
    audio_chunks.append(audio)

if not audio_chunks:
    print("Error: No audio was generated.")
    sys.exit()

print("Status: Concatenating audio...")
# Join all the numpy arrays into one single continuous array
full_audio = np.concatenate(audio_chunks)

# Define file names
temp_wav = 'temp_full.wav'
final_mp3 = 'output_full.mp3'

print("Status: Writing temporary WAV...")
# Save the combined audio. Note: 24000 is the standard sample rate for Kokoro
sf.write(temp_wav, full_audio, 24000)

print(f"Status: Exporting to {final_mp3}...")
audio_segment = AudioSegment.from_wav(temp_wav)
audio_segment.export(final_mp3, format="mp3", bitrate="192k")

print("Status: Playing...")
# Play the final combined file
os.system(f'start /min wmplayer "{os.path.abspath(final_mp3)}"')

# Clean up the temporary WAV file
if os.path.exists(temp_wav):
    os.remove(temp_wav)

print("--- Finished! ---")
```

## Contribute

Corrections and small improvements belong in a pull request on this repository.

<div class="sw-actions">
  <a class="sw-btn sw-btn-source" href="https://github.com/sdcastillo/LocalTextToSpeech">View on GitHub</a>
  <a class="sw-btn sw-btn-pr" href="https://github.com/sdcastillo/LocalTextToSpeech/compare" target="_blank" rel="noopener noreferrer">Contribute / Open a PR</a>
</div>
