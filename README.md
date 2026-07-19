# Sovereign Brain

A sovereign "second brain" for brainstorming, decision making, working on
**personal projects**, and offloading thoughts from the human's brain — local &
private.

This project is quite opinionated, mostly driven by resource constraints — almost
all decent LLMs cannot run on a typical consumer PC or laptop. Here I provide a
solution that actually works.

It's **harness-independent**: just a folder of markdown. Point any agent that
supports an `AGENTS.md` (and, optionally, [Agent Skills](https://github.com/rolznz/sovereign-brain-skills))
at it, backed by a local model.

## How it's organized

- `AGENTS.md` — who your brain is (its personality "Atlas", and its purpose). Always
  in context.
- **Your data** — thoughts, todos, project ideas, brainstorms, history. These files
  appear in this folder as you use it. The whole folder *is* your brain — version
  it with git and back it up.
- **Skills** *(optional)* — structured capabilities (capture a thought, log a
  project idea, run a brainstorm, track a habit…). Installed separately, see below.

## Prerequisites

- Local machine with at least 32GB RAM — or as little as 8GB if you use the
  smaller Bonsai 27B model (see below)
- A local LLM — e.g. [llama.cpp](https://github.com/ggml-org/llama.cpp) serving
  [Qwen3.6-35B-A3B](https://huggingface.co/unsloth/Qwen3.6-35B-A3B-MTP-GGUF) or
  [Bonsai 27B](https://huggingface.co/prism-ml/Bonsai-27B-gguf)
- An agent harness that reads `AGENTS.md` (and, ideally, Agent Skills) — bring your own

## Getting started

> **Sandbox the agent (recommended).** Your agent can write files and run commands, so
> run it inside a throwaway VM where it can't touch your host. [Multipass](https://canonical.com/multipass)
> is an easy option — then do the steps below **inside the VM**:
>
> ```sh
> multipass launch --name sovereign-brain
> multipass shell sovereign-brain
> ```

1. Make a new, empty folder and download
   [`AGENTS.md`](https://raw.githubusercontent.com/rolznz/sovereign-brain/refs/heads/master/AGENTS.md)
   into it:

   ```sh
   mkdir my-brain && cd my-brain
   curl -O https://raw.githubusercontent.com/rolznz/sovereign-brain/refs/heads/master/AGENTS.md
   ```
2. Have a local model running (see [Run a local model](#run-a-local-model) below).
3. Open your agent in that folder and type **`start`**. It'll guide you from there. 🧠

## Run a local model

### 1. Start the model download (do this first — it's large)

The model file is several GB, so kick it off before anything else.

**32GB+ RAM:** Qwen3.6-35B-A3B is the best model for the job. I recommend the
**3-bit XXS quant** — small but still smart enough:

- [Qwen3.6-35B-A3B-UD-IQ3_XXS.gguf](https://huggingface.co/unsloth/Qwen3.6-35B-A3B-MTP-GGUF?show_file_info=Qwen3.6-35B-A3B-UD-IQ3_XXS.gguf)

**Less RAM (as little as 8GB):** grab the **1-bit Bonsai 27B** instead — a 3.9GB
download that's not as sharp as Qwen but still handles tool calling reliably:

- [Bonsai-27B-Q1_0.gguf](https://huggingface.co/prism-ml/Bonsai-27B-gguf?show_file_info=Bonsai-27B-Q1_0.gguf)

### 2. Set up llama.cpp (the inference engine)

Follow the [llama.cpp quick start](https://github.com/ggml-org/llama.cpp#quick-start)
for your OS. If you **build from source**, these commands work:

```sh
# CPU only
cmake -B build
# …or for an NVIDIA GPU
cmake -B build -DGGML_CUDA=ON

cmake --build build --config Release -j
```

### 3. Run the model

Once the `.gguf` has finished downloading, start the server:

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
> kept on the CPU), and `-c` (context size) depending on your hardware.
>
> **Running Bonsai 27B instead?** Point `-m` at `Bonsai-27B-Q1_0.gguf` and drop
> the `-ncmoe` flag — it's a dense model, so there are no expert layers to place.

llama.cpp prints a URL when it starts — open it and confirm you can chat with the
LLM. You now have a local LLM running; point your agent harness at it.

## Optional follow-ups

- **Always-on with systemd.** Create a systemd service so llama.cpp starts
  automatically on boot — no need to launch it by hand each time.
- **Speech to text.** Add [VoxType](https://voxtype.io/) for local speech-to-text so
  you can dictate to your brain. I use parakeet — super fast even on CPU (leaving
  more VRAM for the LLM).
- **Back it up.** Periodically copy your brain folder to a flash drive — it's your
  second brain, don't lose it.
- **Give it a wallet.** Add the [Alby](https://github.com/getAlby/payments-skill)
  payments skill so your agent can have permissioned access to your wallet and pay
  on your behalf.

## License

The Unlicense — released into the public domain. See [LICENSE](LICENSE).
