# Build Sif Image:

```bash
sudo singularity build openwebui.sif openwebui.def
```

### Make directories in host system:

```bash
mkdir -p openwebui_data/{database,config,logs,static,cache/audio}
```

### Set Permissions:

```bash
sudo chmod -R 777 openwebui_data/
```

### Run the container

```bash
singularity run \
  --bind openwebui_data/database:/app/backend/database \
  --bind openwebui_data/config:/app/backend/config \
  --bind openwebui_data/logs:/app/backend/logs \
  --bind openwebui_data/static:/app/backend/open_webui/static \
  --bind openwebui_data/cache:/app/backend/data/cache \
  --writable-tmpfs \
  openwebui.sif
```
