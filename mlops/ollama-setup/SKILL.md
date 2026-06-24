---
name: ollama-setup
description: Install, configure, and troubleshoot Ollama on Windows — with Chinese network mirror workarounds.
---

# Ollama Setup & Configuration

Install and configure Ollama for local LLM inference. Covers Windows installation, model storage relocation, PATH setup, and network workarounds for restricted environments (China/GitHub-blocked).

## Trigger conditions

- User asks to install, set up, or configure Ollama
- `ollama: command not found` or Ollama not in PATH
- Need to relocate model storage to another drive
- Ollama pull/download fails in restricted network

## Installation (Windows)

### 1. Download the installer

Ollama.com redirects downloads to GitHub releases (`github.com/ollama/ollama/releases`), which is blocked in China. Use a mirror:

**Recommended mirror** (Cloudflare CDN, fast ~10 MB/s):
```
curl -L -o "$USERPROFILE/Downloads/OllamaSetup.exe" \
  "https://gh-proxy.com/https://github.com/ollama/ollama/releases/download/v0.6.5/OllamaSetup.exe"
```

**Fallback mirrors** (slower):
- `https://kkgithub.com/ollama/ollama/releases/download/v0.6.5/OllamaSetup.exe`

**Do NOT use**:
- `winget install Ollama.Ollama` — downloads from GitHub, will fail in restricted networks
- Direct ollama.com download — 307 redirects to GitHub

Always check for latest version at https://github.com/ollama/ollama/releases and update the URL accordingly.

### 2. Verify download integrity

```bash
ls -lh "$USERPROFILE/Downloads/OllamaSetup.exe"
# Should be ~1 GB
```

If the file is locked (e.g., by a prior failed process), copy to a temp location first:
```bash
cp "$USERPROFILE/Downloads/OllamaSetup.exe" /tmp/OllamaSetup.exe
```

### 3. Install silently

The installer is an NSIS package. Silent install with `/S`:

```bash
# Direct execution may fail with "Device or resource busy" — use cmd.exe:
cmd.exe /c start /wait "" "C:\Users\<user>\Downloads\OllamaSetup.exe" /S
```

Installs to: `C:\Users\<user>\AppData\Local\Programs\Ollama\`

## Configuration

### Model storage relocation

Ollama stores models in `%USERPROFILE%\.ollama\models` by default. Relocate to a drive with more space via environment variable:

```powershell
# Set OLLAMA_MODELS (User-level, persistent)
[Environment]::SetEnvironmentVariable('OLLAMA_MODELS', 'D:\ollama\models', 'User')
```

**Important**: The env var won't take effect until you restart the ollama process. Kill and restart `ollama serve` after setting.

### PATH configuration

Ollama installs to a per-user location not automatically on PATH. Add it:

```powershell
[Environment]::SetEnvironmentVariable('Path',
  [Environment]::GetEnvironmentVariable('Path', 'User') + ';C:\Users\<user>\AppData\Local\Programs\Ollama',
  'User')
```

New terminal sessions will have `ollama` on PATH. For the current session:
```bash
export PATH="$PATH:/c/Users/<user>/AppData/Local/Programs/Ollama"
```

### Start the server

```bash
# Foreground (blocking):
ollama serve

# Background (Hermes terminal):
terminal(background=true): export PATH="..."; ollama serve
```

Verify: `curl http://localhost:11434/api/tags` → `{"models":[]}`

## Pulling models

Model downloads go through `registry.ollama.ai`, which is accessible from China (unlike GitHub). No mirror needed.

```bash
ollama pull qwen2.5:0.5b   # small test model (~400 MB)
ollama pull qwen2.5:7b     # larger model
ollama pull llama3.2:3b    # Meta model
```

## Verification checklist

- [ ] `ollama --version` returns a version
- [ ] `ollama serve` starts without errors
- [ ] `curl http://localhost:11434/api/tags` responds
- [ ] `ollama pull <model>` downloads successfully
- [ ] `ollama run <model>` produces output
- [ ] Models are stored in the configured `OLLAMA_MODELS` directory

## Pitfalls

- **GitHub blocked in China**: The ollama.com download page 307-redirects to GitHub. Always use a mirror for the installer. Winget also pulls from GitHub — it will fail too.
- **Slow initial download from mirrors**: kkgithub.com works but is slow (~30 KB/s). Prefer gh-proxy.com.
- **File locked after failed install**: If a prior install attempt left the file locked, copy it to a temp directory and run from there.
- **OLLAMA_MODELS not taking effect**: Environment variable changes via setx/powershell only apply to new processes. Restart `ollama serve` after setting.
- **Silent install with cmd.exe**: Use `start /wait "" "path" /S` — the empty `""` is the window title, required by `start` syntax when the path is quoted.
