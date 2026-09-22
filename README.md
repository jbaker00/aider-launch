# aider-launch

Interactive launcher for [aider](https://aider.chat) against a self-hosted
[llama.cpp](https://github.com/ggml-org/llama.cpp) server (the `--models-dir`
/ `--models-preset` multi-model router mode). Prompts for a server, lists the
models it currently has available (and whether they're already loaded or will
cold-load on first use), lets you pick one, and then execs straight into an
`aider` session pointed at it.

## Why

Running a coding agent against a local model is easy to get wrong in a way
that's confusing rather than broken: wrong host, wrong port, wrong model name
typo'd into `--model openai/...`, or a `claude` session that's silently still
talking to your real Anthropic subscription instead of the local box. This
tool removes the typing (and the ambiguity) from the local path — it's a
different binary than `claude`, so there's no session where you're not sure
which backend you're talking to.

## Requirements

- Python 3 (stdlib only — no dependencies)
- [`aider`](https://aider.chat/docs/install.html) on your `PATH`
- Optionally [`fzf`](https://github.com/junegunn/fzf) for a nicer fuzzy-picker
  UI; falls back to a plain numbered prompt if it's not installed
- A llama.cpp server running in router mode (`llama-server --models-dir ...
  --models-preset ...`) reachable over HTTP, exposing the OpenAI-compatible
  `/v1/models` and `/v1/chat/completions` endpoints

## Install

```sh
git clone <this repo> ~/code/aider-launch
ln -s ~/code/aider-launch/aider-launch ~/.local/bin/aider-launch
```

(or just add `~/code/aider-launch` to your `PATH`)

## Usage

```sh
aider-launch [files...]
```

Any arguments are passed straight through to `aider`, so this works exactly
like `aider <files>` normally does — it just adds an interactive server/model
picker in front of it.

```
$ aider-launch calc.py
Server [beelink1]:
Fetching model list from beelink1:8080 ...

 * 1. qwen2.5-coder-7b-instruct-q4_k_m  (7.6B params, Q4_K - Medium)
   2. dolphin3-8b
   3. qwen2.5-3b-instruct
   ...

 * = already loaded (starts instantly); others cold-load on first use
Pick a model number: 1
Launching aider with qwen2.5-coder-7b-instruct-q4_k_m on beelink1 ...
```

The server prompt defaults to whatever you picked last time (remembered in
`~/.aider-launch.json`, in your home directory — not in this repo). Hit Enter
to reuse it, or type a different hostname / Tailscale name / IP.

## How it works

1. Prompts for a server (defaulting to the last one used).
2. Fetches `http://<server>:8080/v1/models` and shows each model's id, size,
   quantization, and whether it's already loaded.
3. Lets you pick one (via `fzf` if available, otherwise a numbered prompt).
4. Remembers your server + model choice for next time.
5. `exec`s into `aider --openai-api-base http://<server>:8080/v1
   --openai-api-key none --model openai/<chosen> <your extra args>`.

## License

MIT
