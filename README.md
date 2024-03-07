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

Some axolotl notes on fine tuning that worked for me. I'll mention where each command needs to be run.

> Models are saved in your `models` directory. This directory is mounted as `/models` inside containers. You'll see both paths used in these instructions.

1. Preprocess a base model (openllama-3b for this, based on what you see in axolotl's official README). **In the axolotl container**: `python3 -m axolotl.cli.preprocess /configs/example-openllama-3b-config.yml`.
  a. This doesn't take too long and writes to `models/openllama-3b-dataset-prepared`.
2. Fine tune the model using the same config. **In the axolotl container**: `accelerate launch -m axolotl.cli.train /configs/example-openllama-3b-config.yml`.
  a. This takes a long time, many hours, and writes to `models/openllama-3b-lora-out`.
  b. Recommend execing into docker from inside tmux so the process continues (and you can reattach to it) if you disconnect.
  c. You'll see a bunch of Python-y dumped objects describing your model's layers when this is done.
3. Merge the models. **In the axolotl container**: `python3 -m axolotl.cli.merge_lora /configs/example-openllama-3b-config.yml --lora_model_dir=/models/openllama-3b-lora-out`
  a. This is fast. It'll write to `models/openllama-3b-lora-out/merged`.
4. Convert the model to .gguf format (necessary to use with nitro). **Run this in the VM, in the base directory of this repo**: `docker run --rm -v $(pwd)/models:/models convert-axolotl-to-gguf:latest /models/openllama-3b-lora-out/merged --outfile /models/openllama-3b-lora-out/merged.gguf --outtype q8_0`
  a. Build the converter image (if you don't already have it): `cd convert-axolotl-to-gguf && docker build -t convert-axolotl-to-gguf .`
  b. This is fast. Writes to `models/openllama-3b-lora-out/merged.gguf`
5. Load the model into nitro. **Run this in the VM, in the base directory of this repo**: `./scripts/loadmodel /models/openllama-3b-lora-out/merged.gguf`
6. Run chat-completion to test it. **Run this in the VM, in the base directory of this repo**: `./scripts/chat-completion "Hello world, how are you doing?"`
