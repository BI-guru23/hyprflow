# Hyprflow

A free, open-source voice-to-text tool for Linux — the WhisperFlow alternative for Hyprland users.

Hyprflow enables seamless speech-to-text input anywhere on your system. Press a keybind to start recording, press again to stop, and your transcribed text is automatically typed into the active window.

## Features

- **GPU-accelerated transcription** via whisper.cpp (CUDA, Vulkan, etc.)
- **System-wide voice input** — works in any application
- **Minimal footprint** — lightweight bash script with no background daemon
- **Waybar integration** — optional visual recording indicator
- **Auto-paste** — transcribed text is copied to clipboard and pasted automatically

## Prerequisites

- [Hyprland](https://hyprland.org/) (Wayland compositor)
- [PipeWire](https://pipewire.org/) (for `pw-record`)
- [wl-clipboard](https://github.com/bugaevc/wl-clipboard) (for `wl-copy`)
- [whisper.cpp](https://github.com/ggerganov/whisper.cpp) (compiled with GPU support)

## Installation

### 1. Build whisper.cpp

Clone and compile whisper.cpp with your preferred GPU backend:

```bash
git clone https://github.com/ggerganov/whisper.cpp
cd whisper.cpp
```

**For CUDA (NVIDIA):**
```bash
cmake -B build -DGGML_CUDA=ON
cmake --build build --config Release
```

**For Vulkan:**
```bash
cmake -B build -DGGML_VULKAN=ON
cmake --build build --config Release
```

**For CPU only:**
```bash
cmake -B build
cmake --build build --config Release
```

### 2. Download a Whisper model

```bash
cd whisper.cpp
./models/download-ggml-model.sh base.en
```

Available models: `tiny`, `base`, `small`, `medium`, `large` (append `.en` for English-only variants).

### 3. Configure Hyprflow

Edit the `hyprflow` script and set the paths to your whisper.cpp binary and model:

```bash
WHISPER_BIN="/path/to/whisper.cpp/build/bin/whisper-cli"
WHISPER_MODEL="/path/to/whisper.cpp/models/ggml-base.en.bin"
```

### 4. Add keybind to Hyprland

Add to your Hyprland config (`~/.config/hypr/hyprland.conf` or a sourced file):

```bash
$hyprflow = /path/to/hyprflow/hyprflow
bindd = SUPER, SPACE, Hyprflow, exec, $hyprflow
```

## Waybar Integration (Optional)

For a visual recording indicator in Waybar:

### 1. Add the indicator script

Create `~/.config/waybar/indicators/hyprflow-indicator.sh`:

```bash
#!/bin/bash

if pgrep -f "rec.*hyprflow" >/dev/null; then
  echo '{"text": "󱑽 ", "class": "active"}'
else
  echo '{"text": ""}'
fi
```

Make it executable:
```bash
chmod +x ~/.config/waybar/indicators/hyprflow-indicator.sh
```

### 2. Add module to Waybar config

In `~/.config/waybar/config.jsonc`:

```jsonc
{
  "modules-right": ["custom/hyprflow-indicator", ...],
  
  "custom/hyprflow-indicator": {
    "exec": "$XDG_CONFIG_HOME/waybar/indicators/hyprflow-indicator.sh",
    "on-click": "hyprflow",
    "signal": 9,
    "return-type": "json"
  }
}
```

### 3. Add styling (optional)

In `~/.config/waybar/style.css`:

```css
#custom-hyprflow-indicator {
  min-width: 12px;
  margin: 0 8px;
}

#custom-hyprflow-indicator.active {
  color: #f38ba8; /* or your preferred recording color */
}
```

## Usage

1. Press `SUPER + SPACE` (or your configured keybind) to **start recording**
2. Speak your text
3. Press `SUPER + SPACE` again to **stop recording**
4. The transcribed text is automatically pasted into the active window

## How It Works

1. **Recording**: Uses `pw-record` to capture audio at 16kHz mono (optimal for Whisper)
2. **Transcription**: Processes audio through whisper.cpp with GPU acceleration
3. **Output**: Copies text to clipboard via `wl-copy` and triggers paste via `hyprctl`

## Troubleshooting

**No audio captured:**
- Ensure PipeWire is running: `systemctl --user status pipewire`
- Check your default audio input device

**Slow transcription:**
- Verify GPU acceleration is working (check whisper.cpp build flags)
- Try a smaller model (`tiny.en` or `base.en`)

**Text not pasting:**
- Ensure `wl-clipboard` is installed
- Some applications may not support `CTRL+SHIFT+V` — text is still in clipboard

## Acknowledgments

- [whisper.cpp](https://github.com/ggerganov/whisper.cpp) — High-performance C/C++ port of OpenAI's Whisper
- [WhisperFlow](https://github.com/lavafroth/whisperflow) — Inspiration for this project
