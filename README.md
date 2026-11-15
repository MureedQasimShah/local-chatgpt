# Run Your Own ChatGPT Locally with One Docker Command

Run a private, ChatGPT-style AI on your own machine with **Open WebUI + Ollama**.
No API key. No monthly bill. Works offline. Your data never leaves your machine.

> Tested on my local Ubuntu laptop (15 GB RAM, no GPU) with `llama3.2`.

- **Open WebUI**: the ChatGPT-style web interface
- **Ollama**: the engine that downloads and runs open models (Llama, Qwen, Gemma, DeepSeek, Phi...)
- Both ship together in a single Docker image: `ghcr.io/open-webui/open-webui:ollama`

---

## Table of Contents

1. [Requirements](#1-requirements)
2. [Install Docker](#2-install-docker)
3. [Run It: One Command](#3-run-it-one-command)
4. [Open It on localhost](#4-open-it-on-localhost)
5. [Download Models](#5-download-models)
6. [Use It from the Terminal](#6-use-it-from-the-terminal)
7. [Use It as an API](#7-use-it-as-an-api)
8. [Monitor Resources](#8-monitor-resources)
9. [Stop, Start, Update, Uninstall](#9-stop-start-update-uninstall)
10. [GPU Support (Optional)](#10-gpu-support-optional)
11. [Troubleshooting](#11-troubleshooting)

---

## 1. Requirements

| | Minimum | Recommended |
|---|---|---|
| RAM | 8 GB | 16 GB+ |
| Disk | 10 GB free | 30 GB+ (models are 1-5 GB each) |
| CPU | Any 64-bit | Modern 4+ cores |
| GPU | Not required | NVIDIA GPU makes it much faster |
| OS | Linux, macOS, or Windows | Linux |

**Rule of thumb:** 8 GB RAM runs 1B-4B models. 16 GB runs 7B-8B models. Bigger models need a GPU.

---

## 2. Install Docker

Already have Docker? Check with `docker --version` and skip to [step 3](#3-run-it-one-command).

### Ubuntu

```bash
# Remove old/conflicting packages (safe if none are installed)
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove -y $pkg; done

# Add Docker's official GPG key
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the Docker repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Debian

```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove -y $pkg; done

sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Fedora

```bash
sudo dnf remove -y docker docker-client docker-client-latest docker-common docker-latest \
  docker-latest-logrotate docker-logrotate docker-selinux docker-engine-selinux docker-engine

sudo dnf -y install dnf-plugins-core
sudo dnf config-manager addrepo --from-repofile=https://download.docker.com/linux/fedora/docker-ce.repo
# Older Fedora (40 and below):
# sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo

sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
```

### RHEL / CentOS Stream / Rocky / AlmaLinux

```bash
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
# CentOS Stream: use https://download.docker.com/linux/centos/docker-ce.repo

sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
```

### Arch Linux / Manjaro

```bash
sudo pacman -Syu --noconfirm docker docker-compose
sudo systemctl enable --now docker
```

### Any Linux: the quick way (official convenience script)

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

### macOS and Windows

Install **Docker Desktop**: https://www.docker.com/products/docker-desktop/
On Windows, enable WSL 2 when the installer asks. Then open Docker Desktop and wait until it says *Engine running*.

### After installing (Linux): run Docker without `sudo`

```bash
sudo systemctl enable --now docker     # start Docker now and on every boot
sudo usermod -aG docker $USER          # add yourself to the docker group
newgrp docker                          # apply the group now (or log out and back in)
```

### Verify Docker works

```bash
docker --version
docker run --rm hello-world
```

If you see *"Hello from Docker!"*, you're ready.

---

## 3. Run It: One Command

```bash
docker run -d \
  -p 3000:8080 \
  -p 11434:11434 \
  -e OLLAMA_HOST=0.0.0.0:11434 \
  -v ollama:/root/.ollama \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:ollama
```

What each flag does:

| Flag | Meaning |
|---|---|
| `-d` | Run in the background |
| `-p 3000:8080` | Web UI at `http://localhost:3000` |
| `-p 11434:11434` | Ollama API at `http://localhost:11434` (for curl and your apps) |
| `-e OLLAMA_HOST=0.0.0.0:11434` | Lets Ollama accept connections from outside the container. **Without this, port 11434 won't work.** |
| `-v ollama:/root/.ollama` | Saves downloaded models in a volume, so they survive restarts |
| `-v open-webui:/app/backend/data` | Saves your chats, users, and settings |
| `--name open-webui` | Container name, used in all commands below |
| `--restart always` | Starts again automatically after a reboot |
| `:ollama` image tag | Open WebUI **with Ollama built in**. One container, nothing else to install. |

The first run downloads the image (a few GB), so give it a few minutes.

### Check that it's running

```bash
docker ps
```

Wait until the `STATUS` column shows `(healthy)`. To watch the startup logs:

```bash
docker logs -f open-webui      # Ctrl+C to stop watching
```

---

## 4. Open It on localhost

Open your browser and go to:

```
http://localhost:3000
```

1. Click **Get started**
2. Create your account. **The first account becomes the admin.** It's stored only on your machine.
3. You'll see a ChatGPT-style screen with **no model** yet. That's normal. Go to the next step.

Check from the terminal:

```bash
curl -I http://localhost:3000            # should return HTTP/1.1 200 OK
curl http://localhost:11434/api/version  # should return {"version":"..."}
```

---

## 5. Download Models

### Option A: from the browser (easiest)

1. Click **Select a model** (top left)
2. Type a model name, for example `llama3.2`
3. Click **Pull "llama3.2" from Ollama.com**
4. Wait for the download, select the model, and start chatting

### Option B: from the terminal

```bash
docker exec -it open-webui ollama pull llama3.2
```

Refresh the browser and the model appears.

### Models to try (one per family)

| Family | Command | Size | Good for |
|---|---|---|---|
| Llama (Meta) | `ollama pull llama3.2` | ~2 GB | General chat, start here |
| Qwen (Alibaba) | `ollama pull qwen3:4b` | ~2.5 GB | Smart small model with a thinking mode |
| Gemma (Google) | `ollama pull gemma3:4b` | ~3.3 GB | Understands images (vision) |
| DeepSeek | `ollama pull deepseek-r1:1.5b` | ~1.1 GB | Reasoning: shows its thinking |
| Phi (Microsoft) | `ollama pull phi4-mini` | ~2.5 GB | Logic and math |
| Qwen Coder | `ollama pull qwen2.5-coder:3b` | ~1.9 GB | Code, Dockerfiles, YAML |
| Embeddings | `ollama pull nomic-embed-text` | ~0.3 GB | Chat with your documents (RAG) |

Run each command with `docker exec -it open-webui` in front of it.
Browse every model and the newest versions at https://ollama.com/library

### Things to try in the UI

- **Compare models:** click **+** next to the model name and send one prompt to two models side by side
- **Chat with a PDF:** click **+** in the message box, upload a file, and ask questions about it
- **Vision:** upload a screenshot to `gemma3:4b` and ask what it shows
- **Reasoning:** give `deepseek-r1` a logic puzzle and watch its thinking

---

## 6. Use It from the Terminal

```bash
# Chat directly in the terminal (type /bye to exit)
docker exec -it open-webui ollama run llama3.2

# One-shot question
docker exec -it open-webui ollama run llama3.2 "Explain Docker in one line"

# List downloaded models
docker exec -it open-webui ollama list

# Show models loaded in memory, and whether they use CPU or GPU
docker exec -it open-webui ollama ps

# Delete a model to free disk space
docker exec -it open-webui ollama rm phi4-mini
```

**Tip:** add an alias so you can just type `ollama`:

```bash
echo "alias ollama='docker exec -it open-webui ollama'" >> ~/.bashrc
source ~/.bashrc
ollama list
```

---

## 7. Use It as an API

### Ollama API

```bash
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2",
  "prompt": "Explain Kubernetes in one line",
  "stream": false
}'
```

### OpenAI-compatible API

Apps that use the OpenAI API can use your local model by changing the base URL:

```bash
curl http://localhost:11434/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "llama3.2",
    "messages": [{"role": "user", "content": "Write a Dockerfile for a Node.js app"}]
  }'
```

Python example (`pip install openai`):

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:11434/v1", api_key="ollama")  # any key works
reply = client.chat.completions.create(
    model="llama3.2",
    messages=[{"role": "user", "content": "What is a Kubernetes Pod?"}],
)
print(reply.choices[0].message.content)
```

---

## 8. Monitor Resources

```bash
docker stats open-webui        # live CPU and RAM usage (Ctrl+C to exit)
docker exec -it open-webui ollama ps
docker system df -v | grep -E "ollama|open-webui"   # disk used by the volumes
```

Send a prompt in the browser while `docker stats` is running and watch the usage jump.

---

## 9. Stop, Start, Update, Uninstall

```bash
# Stop and start
docker stop open-webui
docker start open-webui

# Restart
docker restart open-webui
```

### Update to the latest version

Models and chats live in the volumes, so they stay after the update.

```bash
docker pull ghcr.io/open-webui/open-webui:ollama
docker rm -f open-webui
# then run the command from step 3 again
```

### Proof that your data persists

```bash
docker rm -f open-webui          # delete the container
# run the step 3 command again
docker exec -it open-webui ollama list   # your models are still here
```

### Uninstall completely

```bash
docker rm -f open-webui
docker volume rm ollama open-webui                 # WARNING: deletes all models and chats
docker rmi ghcr.io/open-webui/open-webui:ollama
```

---

## 10. GPU Support (Optional)

Have an NVIDIA GPU? Install the NVIDIA driver and the **NVIDIA Container Toolkit**:
https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html

Then add `--gpus=all` to the run command:

```bash
docker run -d --gpus=all \
  -p 3000:8080 -p 11434:11434 \
  -e OLLAMA_HOST=0.0.0.0:11434 \
  -v ollama:/root/.ollama -v open-webui:/app/backend/data \
  --name open-webui --restart always \
  ghcr.io/open-webui/open-webui:ollama
```

Check it worked: `docker exec -it open-webui ollama ps` should show `GPU` in the PROCESSOR column.

---

## 11. Troubleshooting

| Problem | Fix |
|---|---|
| `permission denied ... docker.sock` | Run `sudo usermod -aG docker $USER`, then log out and back in |
| `Cannot connect to the Docker daemon` | Run `sudo systemctl start docker` |
| `port is already allocated` on 3000 | Use another port, for example `-p 3001:8080`, then open `localhost:3001` |
| "No model" in the dropdown | Normal on first run. Download one ([step 5](#5-download-models)) |
| `curl: (7) Failed to connect` on port 11434 | You're missing `-p 11434:11434`. Recreate the container with the step 3 command. |
| `curl: (56) Connection reset` on port 11434 | You're missing `-e OLLAMA_HOST=0.0.0.0:11434`. Recreate the container. |
| Page doesn't load right after start | Wait for `(healthy)` in `docker ps`, or check `docker logs open-webui` |
| Answers are very slow | Use a smaller model (1B-4B), close heavy apps, or use a GPU |
| Model crashes or runs out of memory | The model is too big for your RAM. Pick a smaller size tag, such as `:1b` or `:3b`. |
| Running out of disk | `docker exec -it open-webui ollama rm <model>` |

---

## Links

- Open WebUI: https://github.com/open-webui/open-webui
- Ollama: https://ollama.com
- Model library: https://ollama.com/library
- Docker install docs: https://docs.docker.com/engine/install/

---

**Next:** deploying this on Kubernetes. Follow **Deploy with Qasim** so you don't miss it.
