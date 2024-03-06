FROM nvidia/cuda:12.3.2-devel-ubuntu22.04

WORKDIR /app

# Install required packages
RUN apt-get update && \
    apt-get install -y git python3 python3-pip

# axolotl's instructions have a bug (well, they use flash-attn, which has
# a bug). it's necessary to install a specific version of torch before
# installing flash-attn, otherwise you'll get an error:
#   ModuleNotFoundError: No module named 'torch'
# tracked here: https://github.com/OpenAccess-AI-Collective/axolotl/issues/1144

# there's also a bug in the bitsandbytes package. it doesn't seem to use the
# right GPU .so file. something to do with 12.3.2.
# mentioned here: https://github.com/oobabooga/text-generation-webui/issues/2414#issuecomment-1569703000

RUN git clone https://github.com/OpenAccess-AI-Collective/axolotl && \
    cd axolotl && \
    pip3 install packaging && \
    pip3 install torch==2.1.1 torchvision==0.16.1 torchaudio==2.1.1 --index-url https://download.pytorch.org/whl/cu121 && \
    pip3 install -e '.[flash-attn,deepseed]'
