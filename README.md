Open WebUI Singularity Container Setup with llama.cpp Backend
This repository provides instructions to build and run a Singularity container for Open WebUI, a user-friendly AI interface, configured to use a llama.cpp server as the backend. The container is built from the official Docker image ghcr.io/open-webui/open-webui:main and set up with persistent storage, writable static assets, and integration with a llama.cpp server running on http://localhost:8000.
Prerequisites

Operating System: Linux (e.g., Ubuntu 22.04 or later)
Singularity: Version 3.8 or higher (tested with 4.3.0)
llama.cpp Server: Running locally on http://localhost:8000 (provides an OpenAI-compatible API)
Root Privileges: Required for building the Singularity image
Dependencies: openssl for generating secret keys

Installation

Install Singularity:Ensure Singularity is installed. On Ubuntu, run:
sudo apt-get update && sudo apt-get install -y singularity-container

Verify the version:
singularity --version

Expected output: singularity-ce version 4.3.0 or similar.

Set Up llama.cpp Server:Ensure your llama.cpp server is running with an OpenAI-compatible API. For example:
./server -m <model.gguf> --host 0.0.0.0 --port 8000 --api

Verify the server is accessible:
curl http://localhost:8000/v1/models

Expected output: A JSON response listing available models (e.g., {"data": [...]}).
Note: The llama.cpp server must expose an OpenAI-compatible API (e.g., /v1/chat/completions). Check the llama.cpp documentation for the correct endpoint and ensure it’s running before starting Open WebUI.


Building the Singularity Image
Build the Singularity SIF image from the official Open WebUI Docker image:
sudo singularity build openwebui.sif docker://ghcr.io/open-webui/open-webui:main


Output: Creates openwebui.sif in the current directory.
Note: Requires internet access to pull the Docker image from GitHub Container Registry (GHCR).

Setting Up Persistent Storage
Create directories for persistent data and static files:
mkdir -p ~/open-webui-data
chmod -R u+rwX ~/open-webui-data

mkdir -p ~/open-webui-static
chmod -R u+rwX ~/open-webui-static


~/open-webui-data: Stores persistent data (e.g., database, user settings).
~/open-webui-static: Stores static assets (e.g., icons, manifests) written during startup.

Running the Container
Run the Open WebUI container, configured to connect to the llama.cpp server:
singularity run \
  --bind ~/open-webui-data:/app/backend/data \
  --bind ~/open-webui-static:/app/backend/open_webui/static \
  --env OPENAI_API_BASE_URL=http://localhost:8000/v1 \
  --env WEBUI_SECRET_KEY=$(openssl rand -hex 32) \
  openwebui.sif /app/backend/start.sh


Options:

--bind ~/open-webui-data:/app/backend/data: Mounts persistent data directory.
--bind ~/open-webui-static:/app/backend/open_webui/static: Mounts writable static assets directory.
--env OPENAI_API_BASE_URL=http://localhost:8000/v1: Points Open WebUI to the llama.cpp server’s OpenAI-compatible API endpoint.
--env WEBUI_SECRET_KEY=$(openssl rand -hex 32): Generates a random secret key for session security.
/app/backend/start.sh: Explicitly runs the startup script.


Access: Open WebUI should be available at http://localhost:8080.


Troubleshooting

Port Conflicts:Ensure port 8080 is free for Open WebUI:
netstat -tuln | grep 8080

If occupied, stop the conflicting service or modify the port in /app/backend/start.sh or by running Uvicorn directly.

llama.cpp Server Not Running:Verify the llama.cpp server:
curl http://localhost:8000/v1/models

If it fails, restart the server:
./server -m <model.gguf> --host 0.0.0.0 --port 8000 --api

Ensure the /v1 endpoint is correct. Some llama.cpp versions may use a different path (e.g., /). Check the server’s documentation or test with curl.

Filesystem Errors:If you see Read-only file system errors, ensure the bind mounts are writable:
ls -ld ~/open-webui-data ~/open-webui-static
touch ~/open-webui-data/test ~/open-webui-static/test
rm ~/open-webui-data/test ~/open-webui-static/test


Missing Environment Variables:If Open WebUI fails with ValueError: Required environment variable not found, inspect /app/backend/start.sh:
singularity shell openwebui.sif
cat /app/backend/start.sh

Add missing variables with --env, e.g., --env VARIABLE=value. For llama.cpp, OPENAI_API_BASE_URL is critical.

Image Corruption:Rebuild the image if issues persist:
sudo singularity build openwebui.sif docker://ghcr.io/open-webui/open-webui:main


llama.cpp Compatibility:Ensure the llama.cpp server supports OpenAI-compatible endpoints (e.g., /v1/chat/completions). Test with:
curl -X POST http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "<model-name>", "messages": [{"role": "user", "content": "Hello"}]}'

If it fails, check the llama.cpp version or configuration.


Optional: Custom Definition File
For a persistent configuration, use a Singularity definition file:
Bootstrap: docker
From: ghcr.io/open-webui/open-webui:main

%environment
    export OPENAI_API_BASE_URL=http://localhost:8000/v1
    export WEBUI_SECRET_KEY=$(openssl rand -hex 32)
    export CORS_ALLOW_ORIGIN=http://localhost:8080
    export USER_AGENT=OpenWebUI/0.6.5

%runscript
    exec /app/backend/start.sh

Save as openwebui.def and build:
sudo singularity build openwebui.sif openwebui.def

Run:
singularity run \
  --bind ~/open-webui-data:/app/backend/data \
  --bind ~/open-webui-static:/app/backend/open_webui/static \
  openwebui.sif

Note: The static WEBUI_SECRET_KEY in %environment may not be ideal for security. Set it dynamically at runtime for production.
Notes

Version: Tested with Open WebUI v0.6.5, Singularity 4.3.0, and llama.cpp server.
Static Files: Open WebUI writes static assets to /app/backend/open_webui/static during startup, requiring a writable bind mount.
CORS: The default CORS_ALLOW_ORIGIN=* is insecure for production. Set CORS_ALLOW_ORIGIN to specific origins (e.g., http://localhost:8080).
llama.cpp Configuration: Ensure the llama.cpp server is configured with the correct model and API endpoint. Some versions may require additional flags (e.g., --chat or --completions).
Documentation: Refer to Open WebUI GitHub and llama.cpp GitHub for additional configuration options.

License
This setup is based on Open WebUI, licensed under the BSD-3-Clause License.
