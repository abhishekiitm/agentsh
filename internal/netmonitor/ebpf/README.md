# eBPF connect hook assets

- `connect.bpf.c`: BPF source for TCP connect and UDP sendmsg hooks.
- `connect_bpfel.o` and `connect_bpfel_arm64.o`: generated little-endian objects
  embedded by `program.go`. These files are ignored, not committed.
- `Makefile`: builds both objects with clang's BPF backend and libbpf headers.

## Build from source

On Debian/Ubuntu, install the build dependencies, then generate both objects:

```bash
sudo apt-get install clang gcc make libbpf-dev linux-libc-dev
make ebpf # from the repository root
```

The program uses `linux/bpf.h`'s stable `bpf_sock_addr` context. It does not
access private kernel structs, so generating `vmlinux.h` from a running kernel
is unnecessary. The objects still include BTF map/type metadata.

On macOS or Windows (using a POSIX shell with Docker), generate the same files
in a Linux container before running Go commands:

```bash
make ebpf-docker
```

`make build`, `make test`, and `make smoke` prepare missing or stale objects.
Raw `go build` / `go test` commands require `make ebpf` (or `make ebpf-docker`)
first. Both objects are required on every platform because Go embeds both.
After changing the source, regenerate them before testing. To force a rebuild:

```bash
make -C internal/netmonitor/ebpf clean
make ebpf
```

CI builds the objects from the checked-out source, validates their layouts and
Linux enforcement, and shares them only with jobs in that workflow run.
Release builds independently generate objects from the release ref and supply
them to GoReleaser, Alpine, and macOS builds. No contributor-supplied object
files are used.

## Kernel compatibility

### Kernel 6.x Notes

Kernel 6.x has stricter BPF verifier rules for cgroup socket programs:

1. **Context pointer restrictions**: Cannot pass `ctx` to helper functions after accessing its fields. All context values must be read into local variables first.

2. **Address family**: The `ctx->family` field may not be accessible in all program types. Use the program type to determine the family (e.g., `connect4`/`sendmsg4` = AF_INET, `connect6`/`sendmsg6` = AF_INET6).

3. **Return values**: cgroup/connect and cgroup/sendmsg programs must return 0 (block) or 1 (allow), not negative errno values.

4. **Socket pointer access**: Direct access to `struct sock` via `ctx->sk` is prohibited. Use context fields directly.

### Backward Compatibility

The code patterns used are intentionally conservative to maximize compatibility:

- Extracting context values to local variables works on all kernel versions
- Inferring address family from program type (connect4 = IPv4) is universally correct
- Return values 0/1 are the documented standard for cgroup socket programs

These patterns satisfy both older kernels (which were more lenient) and newer kernels (which are stricter).

### Supported Program Types

- `cgroup/connect4`: TCP connect for IPv4
- `cgroup/connect6`: TCP connect for IPv6
- `cgroup/sendmsg4`: UDP sendmsg for IPv4
- `cgroup/sendmsg6`: UDP sendmsg for IPv6

