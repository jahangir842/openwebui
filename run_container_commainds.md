# Start Ollama
singularity exec --bind /path/to/ollama/models:/root/.ollama ollama.sif ollama serve

# Then in another terminal: ????
singularity exec openwebui.sif ./start.sh
