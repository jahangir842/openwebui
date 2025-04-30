# Deploy_llms_llama_cpp_server_GPU

### ✅ 1. **Ensure Singularity is Installed**

You’ll need Singularity installed — ideally version **≥3.8** for better Docker support.

Check version:
```bash
singularity --version
```

If you need to install it on Ubuntu (from source):

[Singularity Installation ](https://github.com/jahangir842/linux-notes/tree/main/virtualization/singularity)

---

### ✅ 2. **Install llama.cpp**

Installation Guide:

- [Blog](https://medium.com/@hassanjaved917127/how-to-install-and-run-llama-cpp-with-gguf-models-on-ubuntu-22-04-lts-37120a8a30ac
)


- [Install llama.cpp](https://github.com/jahangir842/LLMs-offline/tree/main/llama.cpp)

---

### ✅ 2. **Save Your Definition File**

open your project and create a file called `llama.def`:

```bash
nano llama.def
```

Paste your full definition into the file and save it.

---

### ✅ 3. **Build the Singularity Container**

You **must use `sudo`** when building containers with Docker as the base.

```bash
sudo singularity build llama.sif llama.def
```

> 🔹 This downloads the Docker image `nvidia/cuda:12.4.0-devel-ubuntu22.04`, installs everything inside it, and outputs `llama.sif`.

---

### ✅ 4. **Run the Container with GPU Support**

To use your GPU, use the `--nv` flag (for NVIDIA drivers):

```bash
singularity run --nv --bind /path/to/models:/models llama.sif -m /models/model.gguf -p "Hello world" -n 128 --n-gpu-layers 35
```

---

### ✅ 5. **Test the Binary Directly (Optional)**

To skip the runscript and invoke the `main` binary directly:

```bash
singularity exec --nv llama.sif /opt/llama.cpp/build/bin/main -m /models/model.gguf -p "Hi" -n 64 --n-gpu-layers 35
```

---

### ✅ 6. **Useful Notes**

- **`--nv`** injects host NVIDIA drivers into the container.
- The `--bind` option maps directories from the host into the container. Useful for:
  - Model files
  - Logs
  - Configuration

Example:
```bash
--bind /mnt/data/models:/models
singularity build llama.sif llama.def
```

## Usage

### Basic Usage

```bash
singularity run --nv --bind /path/to/models:/models <container_name>.sif -m /models/model.gguf -p "Your prompt" -n 128 --n-gpu-layers 35
```

### Server Mode (A100 Fixed Container)

```bash
singularity run --nv --bind /path/to/models:/models llama_a100_fixed.sif --server -m /models/model.gguf --host 0.0.0.0 --port 8080 --n-gpu-layers 100 --tensor-split 0.25,0.25,0.25,0.25 --n-ctx 4096

singularity run --app server --nv --bind /home/hassan/llm_models/:/models llama_a100_fixed.sif -m /models/Ganymede-Llama-3.3-3B-Preview.f16.gguf --host 0.0.0.0 --port 8080 --n-gpu-layers 35
```

### Multi-GPU Configuration

For multi-GPU setups, use the A100 containers with tensor splitting:

```bash
singularity run --nv --bind /path/to/models:/models llama_a100_fixed.sif -m /models/model.gguf -p "Your prompt" -n 128 --n-gpu-layers 100 --tensor-split 0.25,0.25,0.25,0.25
```

## Important Notes

- The `--nv` flag is required for GPU acceleration
- Models must be in GGUF format
- Bind your models directory using `--bind /path/to/models:/models`
- For multi-GPU setups, adjust the `--tensor-split` values according to your GPU configuration
- The A100 Fixed container is recommended for production use as it includes stability improvements

## Web Interface

The repository includes a simple web interface for interacting with the server. To use it:

1. Start the server as described above
2. Access the web interface at `http://127.0.0.1:PORT/index.html`

The web interface supports:
- Chat and completion modes
- System prompt configuration
- Streaming and non-streaming responses
- Local chat history storage
- Multiple independent chat sessions
- Customizable settings through the UI

## Configuration

### Server Configuration (A100 Fixed Container)

5. **Multi-part model issues:**
   - Ensure all model parts are accessible
   - Verify parts are properly ordered (-m part1.gguf -m part2.gguf)
   - Check that GGML_MULTI_MODEL_LOAD=1 environment variable is set
  
---


## Create Share Folder

### Step 1: Install NFS Common

Install the NFS client utilities on each machine that will access the shared directory. Follow these steps:

1. Update the package index to ensure the latest versions are available:
   ```bash
   sudo apt update
   ```

2. Install the NFS common package:
   ```bash
   sudo apt install nfs-common -y
   ```

---

### Step 2: Create a Mount Point

A mount point is required on the client machine to access the shared directory from the server. Create one with the following command:

```bash
sudo mkdir -p /mnt/singularity
sudo mount 192.168.3.124:/home/jahangir/actions-runner/_work/deploy_llms_llama_cpp_server_GPU/deploy_llms_llama_cpp_server_GPU /mnt/singularity
echo "192.168.3.124:/home/jahangir/actions-runner/_work/deploy_llms_llama_cpp_server_GPU/deploy_llms_llama_cpp_server_GPU /mnt/singularity nfs defaults 0 0" | sudo tee -a /etc/fstab
cd /mnt/singularity
```

### Mount it on boot

```bash
echo "192.168.3.124:/home/jahangir/actions-runner/_work/deploy_llms_llama_cpp_server_GPU/deploy_llms_llama_cpp_server_GPU /mnt/singularity nfs defaults 0 0" | sudo tee -a /etc/fstab
```

---

### ✅ 7. **Verify CUDA Works in Container**

You can test CUDA functionality with:

```bash
singularity exec --nv llama.sif nvidia-smi
```

---

Here’s the updated guide with PR Description details added, including a summary and issue resolution section:

---

## 💻 Commit Message Format

Use Conventional Commits:  
- `feat:` New features  
- `fix:` Bug fixes  
- `docs:` Documentation changes  
- `style:` Code style changes  
- `refactor:` Code refactoring  
- `test:` Tests added/modified  
- `chore:` Maintenance tasks  
- `perf:` Performance improvements  
- `ci:` CI/CD updates  
- `revert:` Revert commits  

**Examples:**  
```
feat: add new break announcement feature  
fix: resolve message cooldown timing issue  
docs: update installation instructions  
chore: upgrade OpenAI client to latest version  
```

## Git Branch Naming Format

Prefix branches with a category to indicate intent.  
**Common Prefixes:**  
- `feature/` New features  
- `bugfix/` Bug fixes  
- `hotfix/` Urgent production fixes  
- `docs/` Documentation updates  
- `refactor/` Code refactoring  
- `test/` Adding or updating tests  

**Examples:**  
- `feature/add-user-auth`  
- `bugfix/login-crash`  

## Issue Title Format

Keep titles concise, specific, and actionable. Use a problem or goal statement.  
- Start with a capital letter, no period at the end.  
- Include ticket ID if applicable (e.g., `[JIRA-123]`).  

**Examples:**  
- `Add user authentication endpoint`  
- `Fix login crash on invalid input`  
- `[JIRA-789] Update README with setup steps`  

## PR Title Format

Summarize the change clearly, aligning with the commit type. Use imperative mood.  
- Include ticket ID if applicable.  
- Keep it short (50-72 characters).  

**Examples:**  
- `feat: Add search filter by category`  
- `fix: Resolve timeout in payment API`  
- `[JIRA-456] docs: Document new config options`  

## PR Description Format

Provide context and details in two sections:  
1. **Summary**: Briefly explain what the PR does and why.  
2. **Issue Resolved**: State how it addresses the issue (include ticket ID if applicable).  

**Example:**  
```
Summary:  
This PR adds a category filter to the search functionality, improving user experience by allowing targeted searches.

Issue Resolved:  
Fixes [#5]
```

## Naming Summary

**Example Workflow:**  
- **Issue:** `Add search filter by category`  
- **Branch:** `feature/add-search-filter`  
- **Commit:** `feat(search): Implement filter by category`  
- **PR Title:** `feat: Add search filter by category`  
- **PR Description:**  
  ```
  Summary:  
  Adds a category filter to search for better usability.  

  Issue Resolved:  
  Resolves missing filter functionality ([JIRA-789]).
  ```

--- 

## 🤝 Contributing

1. Fork the repository
2. Create feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes using conventional commit format
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open Pull Request

## Support

For issues and feature requests, please open an issue in the repository.

## License

This project is licensed under the MIT License - see the LICENSE file for details. 
