# LocalTextToSpeech

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
Local Text To Speech Model
