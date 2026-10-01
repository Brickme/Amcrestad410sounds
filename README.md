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

```bash
ffmpeg -i input.mp3 -acodec pcm_alaw -ar 8000 -ac 1 output.raw
```

```
curl -u admin:YOUR_PASSWORD "http://DOORBELL_IP/cgi-bin/audio.cgi?action=postAudio&httptype=singlepart" \
  --header "Content-Type: Audio/G.711A" \
  --data-binary "@output.raw"
```
