# syntax=docker/dockerfile:1
FROM rust:1.99.1-bookworm@sha256:e12566d5abc17a0da741628782cdb504b68b8961f5bfcf677936bf3e8b32de29 AS builder
WORKDIR /source
COPY source/ ./
ENV RUST_MIN_STACK=16777216
ENV CPR_VERSION=3.29.2-rds.1
ENV CPR_GIT_SHA=2221f400da29731c8f62464dd097945b85cf82f1
ENV CPR_BUILD_TYPE=release
RUN rustup component add rustfmt clippy
RUN cargo fmt --all --manifest-path backend/Cargo.toml -- --check
RUN --mount=type=cache,target=/usr/local/cargo/registry \
    --mount=type=cache,target=/usr/local/cargo/git \
    --mount=type=cache,target=/source/backend/target \
    cargo test --manifest-path backend/Cargo.toml -p codex-proxy-rs --test main --locked bootstrap::config:: \
    && cargo clippy --manifest-path backend/Cargo.toml -p gateway-store --all-targets --all-features --locked -- -D warnings \
    && cargo build --manifest-path backend/Cargo.toml --release --locked --bin codex-proxy-rs \
    && cp backend/target/release/codex-proxy-rs /tmp/codex-proxy-rs

# Preserve upstream frontend, runtime dependencies and official plugins verbatim.
FROM ghcr.io/zyycn/codex-proxy-rs@sha256:4b78b221fc18fbc069a6b1f80eab88634a2b722df1c475044b133085fc1d62cf
COPY --from=builder --chown=cpr:cpr /tmp/codex-proxy-rs /app/bin/codex-proxy-rs
LABEL org.opencontainers.image.version="3.29.2-rds.1" \
      org.opencontainers.image.revision="2221f400da29731c8f62464dd097945b85cf82f1" \
      io.rayless.cpr.patch="postgres-password-rds-v1"
