# Sovereign Brain

This is an opinionated project on how to run a local sovereign brain.

## What it's for

A sovereign "second brain" for brainstorming, decision making, working on **personal projects**, offloading thoughts from the human's brain - local & private.

## Prerequisites

- Local machine with at least 32GB RAM
- [Pi agent](https://pi.dev/) - Your agent harness
- [pi-llama-cpp](https://pi.dev/packages/pi-llama-cpp) - Connect your agent to local inference
- [llama.cpp](https://github.com/ggml-org/llama.cpp) - Local inference
- [Qwen3.6-35B-A3B-UD-IQ3_XXS.gguf](https://huggingface.co/unsloth/Qwen3.6-35B-A3B-MTP-GGUF) - Local model

## Getting started

1. Download the template zip file and extract it.
2. Run `git init`
2. Make sure llama is running your local model and a host and port, and pi-llama-cpp is installed and configured.
3. Run `pi` inside the folder, and type "hi".

## License

The Unlicense — released into the public domain. See [LICENSE](LICENSE).