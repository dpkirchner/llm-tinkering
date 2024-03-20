FROM nvidia/cuda:12.3.2-devel-ubuntu22.04

WORKDIR /app

# Install required packages
RUN apt-get update && \
    apt-get install -y git cmake numactl uuid-dev && \
    git clone --recurse https://github.com/dpkirchner/nitro nitro && \
    cd nitro && \
    git checkout fix-content-type && \
    ./install_deps.sh && \
    mkdir build && \
    cd build && \
    cmake .. -DDEBUG=ON -DLLAMA_CUBLAS=ON -DLLAMA_CUDA_F16=ON -DLLAMA_CUDA_DMMV_X=64 -DLLAMA_CUDA_MMV_Y=32 && \
    cmake --build . --config Release -j $(nproc) && \
    apt-get remove --purge -y git cmake && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Expose the port
EXPOSE 3928

# Change the permissions of the nitro binary to make it executable
RUN chmod +x /app/nitro/build/nitro

# Workaround a possible bug in the nvidia/cuda image (they're excluding useful files by default!?)
RUN echo /usr/local/cuda-12.3/compat >> /etc/ld.so.conf.d/cuda-12-compat.conf
RUN ldconfig

# Set the command to run the nitro binary with numactl limiting to cores 0-7
ENTRYPOINT ["/app/nitro/build/nitro"]
CMD ["1", "0.0.0.0", "3928"]
