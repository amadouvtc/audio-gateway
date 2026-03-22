# --- STAGE 1: The Base ---
# We install all tools once so they are cached for all sub-stages.
FROM mcr.microsoft.com/devcontainers/cpp:1-ubuntu-24.04 AS base
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends \
    libasound2-dev cmake clang ccache cppcheck \
    && rm -rf /var/lib/apt/lists/*
WORKDIR /app

# --- STAGE 2: The Audit (The "Microscope") ---
# This stage is for Static Analysis. If this fails, CI turns RED here.
FROM base AS audit
COPY . .
RUN --mount=type=cache,target=/root/.ccache \
    cmake -B build -DCMAKE_EXPORT_COMPILE_COMMANDS=ON && \
    cppcheck --project=build/compile_commands.json --enable=all --error-exitcode=1 -i build/_deps

# --- STAGE 3: The Builder ---
# We only compile here. We copy the "clean" code from the audit stage.
FROM base AS builder
COPY --from=audit /app .
RUN --mount=type=cache,target=/root/.ccache \
    cmake -B build -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_COMPILER_LAUNCHER=ccache && \
    cmake --build build --config Release

# --- STAGE 4: The Runtime (The Final Artifact) ---
FROM ubuntu:24.04 AS runtime
RUN apt-get update && apt-get install -y libasound2 alsa-utils && rm -rf /var/lib/apt/lists/*
RUN useradd -m gatewayuser && usermod -aG audio gatewayuser
USER gatewayuser
WORKDIR /app
COPY --from=builder /app/build/audio_gateway .
ENTRYPOINT ["./AudioGateway"]