# Build Stage
FROM ghcr.io/evanrichter/cargo-fuzz:latest as builder

## Dependencies
RUN wget https://github.com/WebAssembly/binaryen/releases/download/version_109/binaryen-version_109-x86_64-linux.tar.gz && \
    tar xf binaryen-version_109-x86_64-linux.tar.gz && \
    wget https://github.com/WebAssembly/wabt/releases/download/1.0.29/wabt-1.0.29-ubuntu.tar.gz && \
    tar xf wabt-1.0.29-ubuntu.tar.gz

## Add source code to the build stage.
ADD . /src
WORKDIR /src

RUN echo building instrumented harnesses && \
    bash -c "pushd fuzz && cargo +nightly -Z sparse-registry fuzz build && popd" && \
    mv fuzz/target/x86_64-unknown-linux-gnu/release/watgen /watgen && \
    mv fuzz/target/x86_64-unknown-linux-gnu/release/wasm-opt-ttf /wasm-opt-ttf && \
    mv fuzz/target/x86_64-unknown-linux-gnu/release/raw /raw && \
    echo done

RUN echo building non-instrumented harnesses && \
    export RUSTFLAGS="--cfg fuzzing -Clink-dead-code -Cdebug-assertions -C codegen-units=1" && \
    bash -c "pushd fuzz && cargo +nightly -Z sparse-registry build --release && popd" && \
    mv fuzz/target/release/watgen /watgen_no_inst && \
    mv fuzz/target/release/wasm-opt-ttf /wasm-opt-ttf_no_inst && \
    mv fuzz/target/release/raw /raw_no_inst && \
    echo done

# Package Stage
FROM rustlang/rust:nightly

COPY --from=builder /binaryen-version_109/bin/wasm-opt \
                    /wabt-1.0.29/bin/wasm-interp \
                    /usr/bin/
COPY --from=builder /watgen /watgen_no_inst /
COPY --from=builder /wasm-opt-ttf /wasm-opt-ttf_no_inst /
COPY --from=builder /raw /raw_no_inst /
