# Lunar V2

Main native Lunar V2 application and runtime.

## Repositories

- `lunar_v2_frontend/` is tracked as a git submodule and contains the experimental native WebView frontend prototype for Lunar V2.
- The frontend repo is intentionally separate from the core runtime while the API contract and embedding model are still evolving.
- `lunar_protect` is being designed as a separate private repository for the MSVC-compatible code-protection toolchain. It will be pinned as a submodule after its private remote is available.

## Frontend Integration

- The current integration plan is documented in [docs/frontend-integration.md](docs/frontend-integration.md).
- Short version: frontend stays in its own repository, real bridge handlers are implemented in `lunar_v2`, and the frontend submodule is consumed as UI assets by the main build.

## Protection Toolchain

- The repository decision and staged rollout are documented in [docs/lunar-protect-integration.md](docs/lunar-protect-integration.md).
- `windows-msvc-protected` is the opt-in integration preset. It discovers a sibling `../lunar_protect` checkout until the private remote is available, generates protected values and a private manifest, embeds a per-build watermark, enables Windows CFG/CET hardening, and builds the CPU tests.
- `windows-msvc-protected-cuda` applies the same protected control-plane build while also staging the native CUDA/TensorRT detector module.
- The current KeyAuth slice is a passive, testable API 1.3 adapter plus a timed lease and runtime-capability boundary. Endpoint, application, owner ID, and Ed25519 response key are pinned at protected-build time; missing license configuration remains fail-closed.

```powershell
cmake --preset windows-msvc-protected
cmake --build --preset windows-msvc-protected
ctest --test-dir build/windows-msvc-protected --output-on-failure

cmake --preset windows-msvc-protected-cuda
cmake --build --preset windows-msvc-protected-cuda
```

## Notes

- Clone with submodules enabled, or run `git submodule update --init --recursive` after checkout.
