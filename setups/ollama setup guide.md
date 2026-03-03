# Ollama + Open WebUI Setup Guide

## Install Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Verify installation:

```bash
ollama --version
```

---

## Manage Models

Pull a model:

```bash
ollama pull qwen3:4b
```

List installed models:

```bash
ollama list
```

Remove a model:

```bash
ollama rm qwen3:4b
```

Show model info:

```bash
ollama show qwen3:4b
```

Other models worth trying on 16GB RAM:

```bash
ollama pull llama3.1:8b
ollama pull gemma2:9b
ollama pull mistral:7b
ollama pull phi4-mini
```

---

## Chat in Terminal

Start a chat:

```bash
ollama run qwen3:4b
```

Inside the chat:

- Type your message and press Enter
- Use `/bye` to exit
- Use `/set think` to enable thinking mode
- Use `/set nothink` to disable thinking mode
- Use `/set parameter num_ctx 8192` to increase context window (uses more RAM)

Send a one-off prompt without entering chat:

```bash
echo "explain DNS in simple terms" | ollama run qwen3:4b
```

---

## Open WebUI (Web Search + Better Interface)

### Option A: Python / pip (recommended for personal use)

Lighter, no Docker needed. Requires Python 3.11.

Check your Python version:

```bash
python3 --version
```

If already on 3.11:

```bash
pip install open-webui
open-webui serve
```

If not on 3.11, use uv to manage it:

```bash
pip install uv
uv venv open-webui-env --python 3.11 --seed
source open-webui-env/bin/activate
pip install open-webui
open-webui serve
```

Open in browser: [localhost:8080](http://localhost:8080/)

First visit: create a local admin account (stays on your machine).

Start it anytime:

```bash
# if using uv venv, activate first
source open-webui-env/bin/activate
open-webui serve
```

Stop: press `Ctrl+C` in the terminal.

Update:

```bash
pip install --upgrade open-webui
```

Uninstall:

```bash
pip uninstall open-webui
# if using uv venv, also remove the folder
rm -rf open-webui-env
```

### Option B: Docker (if you prefer containers)

Requires Docker Desktop from [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/)

Install and run:

```bash
docker run -d \
  -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

Open in browser: [localhost:3000](http://localhost:3000/)

Update:

```bash
docker pull ghcr.io/open-webui/open-webui:main
docker stop open-webui
docker rm open-webui
```

Then re-run the install command above.

Stop / Start / Remove:

```bash
docker stop open-webui
docker start open-webui
docker rm open-webui        # remove container
docker rmi ghcr.io/open-webui/open-webui:main  # remove image
```

---

## Uninstall Ollama

Quit Ollama if running, then:

```bash
# Remove binary
sudo rm $(which ollama)

# Remove all models and config
rm -rf ~/.ollama
```

If installed via .dmg: also drag Ollama from Applications to Trash.

---

## Daily Workflow

**Quick offline question:** terminal → `ollama run qwen3:4b`

**Need web search or longer session:** run `open-webui serve` → open [localhost:8080](http://localhost:8080/) → chat with web access

---

## Useful Commands Cheat Sheet

| Task                       | Command                                          |
| -------------------------- | ------------------------------------------------ |
| Install Ollama             | `curl -fsSL https://ollama.com/install.sh \| sh` |
| Pull model                 | `ollama pull qwen3:4b`                           |
| Chat                       | `ollama run qwen3:4b`                            |
| List models                | `ollama list`                                    |
| Delete model               | `ollama rm qwen3:4b`                             |
| Model info                 | `ollama show qwen3:4b`                           |
| Start Open WebUI (pip)     | `open-webui serve`                               |
| Stop Open WebUI (pip)      | `Ctrl+C` in terminal                             |
| Update Open WebUI (pip)    | `pip install --upgrade open-webui`               |
| Start Open WebUI (Docker)  | `docker start open-webui`                        |
| Stop Open WebUI (Docker)   | `docker stop open-webui`                         |
| Update Open WebUI (Docker) | `docker pull ghcr.io/open-webui/open-webui:main` |
| Uninstall Ollama           | `sudo rm $(which ollama) && rm -rf ~/.ollama`    |