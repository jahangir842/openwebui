Here is your text rewritten without changing its meaning or structure—just reformatted slightly for clarity and consistency:

---

**Open WebUI Singularity Container Setup with llama.cpp Backend**  
This repository provides instructions to build and run a Singularity container for Open WebUI, a user-friendly AI interface, configured to use a `llama.cpp` server as the backend. The container is built from the official Docker image `ghcr.io/open-webui/open-webui:main` and set up with persistent storage, writable static assets, and integration with a `llama.cpp` server running on `http://localhost:8000`.

---

### Prerequisites

- **Operating System:** Linux (e.g., Ubuntu 22.04 or later)  
- **Singularity:** Version 3.8 or higher (tested with 4.3.0)  
- **llama.cpp Server:** Running locally on `http://localhost:8000` (provides an OpenAI-compatible API)  
- **Root Privileges:** Required for building the Singularity image  
- **Dependencies:** `openssl` for generating secret keys  

---

### Installation

**1. Install Singularity**  
Ensure Singularity is installed. On Ubuntu, run:  
```bash
sudo apt-get update && sudo apt-get install -y singularity-container
```

**Verify the version:**  
```bash
singularity --version
```

Expected output:  
```text
singularity-ce version 4.3.0
```

**2. Set Up `llama.cpp` Server**  
Ensure your `llama.cpp` server is running with an OpenAI-compatible API. For example:  
```bash
./server -m <model.gguf> --host 0.0.0.0 --port 8000 --api
```

**Verify the server is accessible:**  
```bash
curl http://localhost:8000/v1/models
```

Expected output: JSON listing available models, e.g., `{"data": [...]}`.  
**Note:** Ensure `/v1/chat/completions` is available—refer to `llama.cpp` documentation.

---

### Building the Singularity Image

Build the `.sif` image from the official Docker image:  
```bash
sudo singularity build openwebui.sif docker://ghcr.io/open-webui/open-webui:main
```

Creates `openwebui.sif` in the current directory.  
**Note:** Internet access is required to pull the Docker image from GHCR.

---

### Setting Up Persistent Storage

Create directories:  
```bash
mkdir -p ~/open-webui-data
chmod -R u+rwX ~/open-webui-data

mkdir -p ~/open-webui-static
chmod -R u+rwX ~/open-webui-static
```

- `~/open-webui-data`: Persistent data (database, user settings)  
- `~/open-webui-static`: Static assets (icons, manifests)  

---

### Running the Container

Run the container with the following command:  
```bash
singularity run \
  --bind ~/open-webui-data:/app/backend/data \
  --bind ~/open-webui-static:/app/backend/open_webui/static \
  --env OPENAI_API_BASE_URL=http://localhost:8000/v1 \
  --env WEBUI_SECRET_KEY=$(openssl rand -hex 32) \
  openwebui.sif /app/backend/start.sh
```

**Options explained:**

- `--bind ~/open-webui-data:/app/backend/data`: Mount persistent data  
- `--bind ~/open-webui-static:/app/backend/open_webui/static`: Writable static assets  
- `--env OPENAI_API_BASE_URL=http://localhost:8000/v1`: llama.cpp API endpoint  
- `--env WEBUI_SECRET_KEY=$(openssl rand -hex 32)`: Random secret key  
- `/app/backend/start.sh`: Startup script  

**Access:** Open WebUI will be available at `http://localhost:8080`.

---

### Troubleshooting

**Port Conflicts**  
Ensure port 8080 is free:  
```bash
netstat -tuln | grep 8080
```

Stop conflicting service or modify the port in `/app/backend/start.sh`.

**llama.cpp Server Not Running**  
Check server:  
```bash
curl http://localhost:8000/v1/models
```

If it fails, restart:  
```bash
./server -m <model.gguf> --host 0.0.0.0 --port 8000 --api
```

Ensure `/v1` endpoint is correct; refer to `llama.cpp` documentation.

**Filesystem Errors**  
Check write permissions:  
```bash
ls -ld ~/open-webui-data ~/open-webui-static
touch ~/open-webui-data/test ~/open-webui-static/test
rm ~/open-webui-data/test ~/open-webui-static/test
```

**Missing Environment Variables**  
If you see `ValueError: Required environment variable not found`, inspect the script:  
```bash
singularity shell openwebui.sif
cat /app/backend/start.sh
```

Add missing variables using `--env`, e.g., `--env VARIABLE=value`.

**Image Corruption**  
Rebuild image:  
```bash
sudo singularity build openwebui.sif docker://ghcr.io/open-webui/open-webui:main
```

**llama.cpp Compatibility**  
Test endpoint:  
```bash
curl -X POST http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "<model-name>", "messages": [{"role": "user", "content": "Hello"}]}'
```

If it fails, check the `llama.cpp` version or flags.

---

### Optional: Custom Definition File

Create `openwebui.def`:  
```
Bootstrap: docker
From: ghcr.io/open-webui/open-webui:main

%environment
    export OPENAI_API_BASE_URL=http://localhost:8000/v1
    export WEBUI_SECRET_KEY=$(openssl rand -hex 32)
    export CORS_ALLOW_ORIGIN=http://localhost:8080
    export USER_AGENT=OpenWebUI/0.6.5

%runscript
    exec /app/backend/start.sh
```

**Build and run:**  
```bash
sudo singularity build openwebui.sif openwebui.def

singularity run \
  --bind ~/open-webui-data:/app/backend/data \
  --bind ~/open-webui-static:/app/backend/open_webui/static \
  openwebui.sif
```

**Note:** For production, do **not** hardcode `WEBUI_SECRET_KEY`. Set it dynamically.

---

### Notes

- **Version:** Open WebUI v0.6.5, Singularity 4.3.0, `llama.cpp` server  
- **Static Files:** Written to `/app/backend/open_webui/static`  
- **CORS:** Use specific origin in production  
- **llama.cpp Configuration:** Ensure correct model and API flags  
- **Documentation:** Refer to official Open WebUI and `llama.cpp` GitHub repos

---

### License

This setup is based on Open WebUI, licensed under the BSD-3-Clause License.

---

Would you like this converted into a downloadable PDF or markdown file?
