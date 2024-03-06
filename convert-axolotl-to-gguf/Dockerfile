FROM nvidia/cuda:12.3.2-devel-ubuntu22.04

WORKDIR /app

# Install required packages
RUN apt-get update && \
    apt-get install -y git python3 python3-pip

RUN git clone https://github.com/ggerganov/llama.cpp.git && \
    cd llama.cpp && \
    pip3 install -r requirements.txt

ENTRYPOINT ["python3", "llama.cpp/convert.py"]
