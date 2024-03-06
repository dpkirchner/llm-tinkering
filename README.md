This stuff assumes you have a nvidia GPU handy and that you have the nvidia toolkit packages installed. Run `nvidia-smi` to see if you do.

Then run this to bring up nitro and a container that has axolotl inside:

```
docker compose up -d
```

Once you've done this, download a model to your `models` directory. I suggest starting with something like (`cd models && wget https://huggingface.co/TheBloke/CapybaraHermes-2.5-Mistral-7B-GGUF/resolve/main/capybarahermes-2.5-mistral-7b.Q5_K_M.gguf`).

Then run: `./scripts/loadmodel /models/capybarahermes-2.5-mistral-7b.Q5_K_M.gguf` and `./scripts/chat-completion "What's up?"` to see if you get a useful response:

```json
{
  "choices": [
    {
      "finish_reason": null,
      "index": 0,
      "message": {
        "content": "☀️ Good morning! How can I help you today?\n\n",
        "role": "assistant"
      }
    }
  ],
  "created": 1709763205,
  "id": "ifQYMayCCIKu40wxW3wj",
  "model": "_",
  "object": "chat.completion",
  "system_fingerprint": "_",
  "usage": {
    "completion_tokens": 17,
    "prompt_tokens": 14,
    "total_tokens": 31
  }
}
```

(OK so this isn't useful.)

TODO: Document using axolotl to finetune.

Some axolotl notes on fine tuning that worked for me.

1) Preprocess a base model (openllama-3b for this, based on what you see in axolotl's official README). Run this in the axolotl docker container: `python3 -m axolotl.cli.preprocess /configs/example-openllama-3b-config.yml`. This doesn't take too long and writes to `/models/openllama-3b-dataset-prepared`.
2) Fine tune the model using the same config: `accelerate launch -m axolotl.cli.train /configs/example-openllama-3b-config.yml`. This takes a long time.
3) Convert the model to .gguf format (TODO Document this)
4) Load the model into nitro
5) Run chat-completion to test it.
