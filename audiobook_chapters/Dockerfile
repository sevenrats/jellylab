# Multi-stage Dockerfile for building Jellyfin server + web into a test image
# Extends the official jellyfin/jellyfin image with our built code

# ============================================================
# Stage 1: Build the web client
# ============================================================
FROM node:24-bookworm-slim AS web-build

RUN apt-get update && apt-get install -y --no-install-recommends git ca-certificates && rm -rf /var/lib/apt/lists/*
WORKDIR /src/jellyfin-web
RUN git clone --depth 1 --branch feat/audiobook-chapters https://github.com/sevenrats/jellyfin-web.git .
RUN npm ci --no-audit
RUN npm run build:production

# ============================================================
# Stage 2: Build the server
# ============================================================
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS server-build

RUN apt-get update && apt-get install -y --no-install-recommends git ca-certificates && rm -rf /var/lib/apt/lists/*
WORKDIR /src/jellyfin
RUN git clone --depth 1 --branch feat/audiobook_chapters https://github.com/sevenrats/jellyfin.git .
RUN dotnet publish Jellyfin.Server/Jellyfin.Server.csproj \
    --configuration Release \
    --output /build/server

# ============================================================
# Stage 3: Use official Jellyfin runtime, swap in our builds
# ============================================================
FROM jellyfin/jellyfin:unstable AS runtime

# Replace server binaries with our build
COPY --from=server-build /build/server /jellyfin

# Replace web client with our build
COPY --from=web-build /src/jellyfin-web/dist /jellyfin/jellyfin-web
