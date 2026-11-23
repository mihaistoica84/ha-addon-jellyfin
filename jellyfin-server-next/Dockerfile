# https://developers.home-assistant.io/docs/add-ons/configuration#add-on-dockerfile
# Jellyfin server built from source at JELLYFIN_REF with the patches in ./patches applied,
# layered on the official image so ffmpeg, GPU drivers and jellyfin-web stay the official ones.
ARG JELLYFIN_IMAGE=jellyfin/jellyfin:12.4.20261026-211555

FROM --platform=$BUILDPLATFORM ubuntu:26.04 AS server
ARG TARGETARCH
ARG JELLYFIN_REF=v12.4
ENV DOTNET_CLI_TELEMETRY_OPTOUT=1

RUN apt-get update \
 && apt-get install -y --no-install-recommends ca-certificates git dotnet-sdk-10.0 \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /src
RUN git clone --depth 1 --branch "${JELLYFIN_REF}" https://github.com/jellyfin/jellyfin.git .

COPY patches/ /patches/
RUN for patch in /patches/*.patch; do git apply --verbose "${patch}" || exit 1; done

RUN case "${TARGETARCH}" in \
      amd64) DOTNET_ARCH=x64 ;; \
      arm64) DOTNET_ARCH=arm64 ;; \
      *) echo "Unsupported architecture: ${TARGETARCH}" >&2; exit 1 ;; \
    esac \
 && dotnet publish Jellyfin.Server --arch "${DOTNET_ARCH}" --configuration Release \
      --output /server --self-contained -p:DebugSymbols=false -p:DebugType=none

FROM ${JELLYFIN_IMAGE}

# Replace the official server binaries, keeping the bundled web client
RUN find /jellyfin -mindepth 1 -maxdepth 1 ! -name jellyfin-web -exec rm -rf {} +
COPY --from=server /server/ /jellyfin/

# Separate data directory, so the existing Jellyfin add-on's /share/jellyfin is never touched
ENV JELLYFIN_DATA_DIR=/share/jellyfin-next
ENV JELLYFIN_CONFIG_DIR=/share/jellyfin-next/config
ENV JELLYFIN_CACHE_DIR=/share/jellyfin-next/cache
ENV JELLYFIN_LOG_DIR=/share/jellyfin-next/log
ENV XDG_CACHE_HOME=/share/jellyfin-next/cache

ENTRYPOINT ["/jellyfin/jellyfin"]
