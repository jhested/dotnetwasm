FROM --platform=$BUILDPLATFORM scratch
COPY ./bin/Debug/net7.0/WasmConsoleApp.wasm /WasmConsoleApp.wasm
ENTRYPOINT [ "WasmConsoleApp.wasm" ]