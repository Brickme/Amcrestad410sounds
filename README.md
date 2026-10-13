# Amcrest AD410 Doorbell Setup & Custom Audio

Documentation and scripts for managing custom audio alerts, dynamic sound responses, and integrations for the Amcrest AD410 doorbell camera.

## Features

- **Custom Audio Playback**: Push G.711A encoded audio files directly to the doorbell speaker via HTTP CGI commands.
- **Home Assistant Integration**: Trigger sound alerts dynamically using Home Assistant automations or dashboard buttons.
- **Audio Conversion Pipeline**: Convert `.mp3`, `.wav`, or other audio clips into the required G.711A format using FFmpeg.

## Prerequisites

- **Amcrest AD410 Doorbell** connected to your local network.
- **FFmpeg** installed via WSL, Linux, or platform of choice (for audio encoding).
- **Home Assistant** (optional, for automation triggers).

## Setup & Usage

### 1. Convert Audio to G.711A
Run the following FFmpeg command in Windows PowerShell (via WSL) or terminal to encode your sound file:

```bash ## Default
ffmpeg -i input.mp3 -acodec pcm_alaw -ar 8000 -ac 1 output.raw
```
## I uesd
```
ffmpeg -i input.mp3 -f s16le -acodec pcm_s16le -ar 16000 -ac 1 output.raw
```

## Default
``` 
curl -u admin:YOUR_PASSWORD "http://DOORBELL_IP/cgi-bin/audio.cgi?action=postAudio&httptype=singlepart" \
  --header "Content-Type: Audio/G.711A" \
  --data-binary "@output.raw"
```
## I used
```
#!/usr/bin/env bash

DOORBELL_IP="192.168.xx.xxx"   # Your AD410 IP
USER="admin"
PASSWORD="yourpassword"

# Take passed argument ($1) if provided, otherwise default to output.raw
AUDIO_FILE="${1:-/config/www/uploads/output.raw}"

# Check if the file actually exists before running
if [ ! -f "$AUDIO_FILE" ]; then
  echo "Error: Audio file '$AUDIO_FILE' not found."
  exit 1
fi

FILE_SIZE=$(stat -c%s "$AUDIO_FILE" 2>/dev/null || stat -f%z "$AUDIO_FILE")
DURATION=$(( FILE_SIZE / 16000 ))
TIMEOUT=$(( DURATION + 2 ))

echo "Uploading $AUDIO_FILE (${DURATION}s audio, timeout set to ${TIMEOUT}s)..."

curl --digest -u "${USER}:${PASSWORD}" \
     --max-time "${TIMEOUT}" \
     -H "Content-Type: Audio/G.711A" \
     --data-binary "@${AUDIO_FILE}" \
     "http://${DOORBELL_IP}/cgi-bin/audio.cgi?action=postAudio&httptype=singlepart&sampleRate=8000&channel=1"

echo ""
echo "Audio play command finished."
```
