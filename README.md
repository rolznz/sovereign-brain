# Sovereign Brain

A sovereign "second brain" for brainstorming, decision making, working on **personal projects**, offloading thoughts from the human's brain - local & private.

This project is quite opinionated, mostly driven by resource constraints - that almost all decent LLMs cannot run on a typical consumer PC or laptop. Here I provide a solution that actually works.

## Prerequisites

- Local machine with at least 32GB RAM
- [Pi agent](https://pi.dev/) - Your agent harness
- [pi-llama-cpp](https://pi.dev/packages/pi-llama-cpp) - Connect your agent to local inference
- [llama.cpp](https://github.com/ggml-org/llama.cpp) - Local inference
- [Qwen3.6-35B-A3B-UD-IQ3_XXS.gguf](https://huggingface.co/unsloth/Qwen3.6-35B-A3B-MTP-GGUF) - Local model

## Getting started

1. [Download](https://github.com/rolznz/sovereign-brain/archive/refs/heads/master.zip) the sovereign brain template and extract it.
2. Run `git init`
3. Make sure llama is running your local model and a host and port, and pi-llama-cpp is installed and configured.
4. Run `pi` inside the folder, and type "hi".

## Detailed guide

A full, step-by-step walkthrough of the whole stack. Here's what you'll set up:

| # | Piece | Tool |
|---|-------|------|
| 1 | LLM inference | [llama.cpp](https://github.com/ggml-org/llama.cpp) |
| 2 | Local LLM | [Qwen3.6-35B-A3B](https://huggingface.co/unsloth/Qwen3.6-35B-A3B-MTP-GGUF) |
| 3 | 24/7 setup | systemd |
| 4 | Sandboxed agent VM | [Multipass](https://canonical.com/multipass) |
| 5 | Agent harness | [Pi](https://pi.dev/) |
| 6 | Local speech to text | [VoxType](https://voxtype.io/) |
| 7 | Backups | A flash drive |
| 8 | Wallet | [Alby](https://getalby.com/ai?ref=sovereignbrain) payments skill |

Steps 1–5 get you to a working, talking brain. Steps 6–8 are optional follow-ups.

### 1. Start the model download (do this first — it's large)

The model file is several GB, so kick off the download before anything else. I
recommend the **3-bit XXS quant** — it's small but still smart enough to do the job:

- [Qwen3.6-35B-A3B-UD-IQ3_XXS.gguf](https://huggingface.co/unsloth/Qwen3.6-35B-A3B-MTP-GGUF?show_file_info=Qwen3.6-35B-A3B-UD-IQ3_XXS.gguf)

While it downloads, move on to the next step.

### 2. Set up llama.cpp (the inference engine)

Follow the [llama.cpp quick start](https://github.com/ggml-org/llama.cpp#quick-start)
for your OS.

If you **build from source**, these commands work:

```sh
# CPU only
cmake -B build
# …or for an NVIDIA GPU
cmake -B build -DGGML_CUDA=ON

cmake --build build --config Release -j
```

### 3. Run the model

Once the `.gguf` file from step 1 has finished downloading, start the server:

```sh
./build/bin/llama-server \
  -m Qwen3.6-35B-A3B-UD-IQ3_XXS.gguf \
  -ngl 24 -np 1 -fa on \
  -ctk q4_0 -ctv q4_0 \
  -c 262144 \
  --host 0.0.0.0 --port 8088 \
  -ncmoe 38 --no-mmap --jinja
```

> **Tune for your machine.** The flags above are tuned for an **NVIDIA 4060
> laptop**. Adjust `-ngl` (layers offloaded to the GPU), `-ncmoe` (expert layers
> kept on the CPU), and `-c` (context size) up or down depending on how powerful
> your machine is.

llama.cpp prints a URL when it starts — open it and confirm you can chat with the
LLM. You now have a local LLM running.

### 4. Create the sandboxed agent VM (Multipass)

The agent runs inside a VM so it can't touch your host system. Install
[Multipass](https://canonical.com/multipass), create a VM, and shell into it:

```sh
multipass launch --name sandbox-pi-llama
multipass shell sandbox-pi-llama
```

### 5. Install the agent harness (Pi) inside the VM

The remaining commands run **inside the VM**:

1. Install [Pi](https://pi.dev/) and the
   [`pi-llama-cpp`](https://pi.dev/packages/pi-llama-cpp) plugin.
2. Download this template into the VM, extract it, and `cd` into the extracted folder.
3. Start the agent and say hi:

   ```sh
   pi
   # then type:
   hi
   ```

Verify the agent responds — it's talking to your llama.cpp server from step 3.
That's your sovereign brain, alive. 🧠

### 6. Always-on with systemd (optional)

Create a systemd service so llama.cpp starts automatically on boot — no need to
launch it by hand each time.

You can also add a startup script that waits for the server to be healthy, then
drops you straight into your brain:

```sh
gnome-terminal -- bash -c 'until curl -sf http://localhost:8088/health | grep -q "ok"; do sleep 2; done; exec multipass exec sandbox-pi-llama -- bash -ic "cd ~/sovereign-brain && pi; exec bash"'
```

### 7. Talk to it with VoxType (optional)

Add [VoxType](https://voxtype.io/) for local speech-to-text so you can dictate to
your brain instead of typing. I use parakeet as it's super fast even on CPU (leaving more VRAM for the LLM)

### 8. Back it up (optional)

Periodically copy your brain folder to a flash drive — it's your second brain,
don't lose it.

### 9. Give it a wallet with Alby (optional)

Add the [Alby](https://github.com/getAlby/payments-skill) payments skill so your agent
can have permissioned access to your wallet and pay on your behalf.

## License

The Unlicense — released into the public domain. See [LICENSE](LICENSE).