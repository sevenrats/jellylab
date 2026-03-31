# Multi-stage Dockerfile for building Jellyfin server + web into a test image
# Pulls source from GitHub forks

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
# Stage 3: Runtime image
# ============================================================
FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS runtime

# Install ffmpeg (required for media probing/transcoding)
RUN apt-get update \
    && apt-get install -y --no-install-recommends ffmpeg \
    && rm -rf /var/lib/apt/lists/*

# Create jellyfin user
RUN groupadd -r jellyfin && useradd -r -g jellyfin -m jellyfin

# Copy server build
COPY --from=server-build /build/server /jellyfin/server

# Copy web client build
COPY --from=web-build /src/jellyfin-web/dist /jellyfin/web

# Create data directories
RUN mkdir -p /config /cache /media && chown -R jellyfin:jellyfin /config /cache /media /jellyfin

USER jellyfin

EXPOSE 8096

ENTRYPOINT ["dotnet", "/jellyfin/server/jellyfin.dll", \
    "--datadir", "/config", \
    "--cachedir", "/cache", \
    "--webdir", "/jellyfin/web"]
