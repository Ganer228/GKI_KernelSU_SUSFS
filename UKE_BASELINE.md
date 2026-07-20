# Xiaomi Pad 7 (`uke`) — working baseline reconstruction

## Goal

Reproduce the known-working kernel before enabling `CONFIG_USER_NS`.

Known-working `Image`:

- release: `6.1.118uke-Droidspaces-ReSukiSU`
- ReSukiSU: `v4.1.0-2206a7dd`
- Thin LTO
- `CONFIG_USER_NS` disabled
- `CONFIG_NTSYNC=y`
- built-in ZRAM/ZSMALLOC

## Why this branch starts from an old WildKernels commit

The working binary matches the first WildKernels DroidSpaces profile much more closely than the current action. The old profile enabled PID/IPC/SYSVIPC plus IPSet/netfilter, but did not enable USER namespaces. Later revisions removed parts of that profile and added `CONFIG_USER_NS=y` and forced ZRAM changes.

Base commit:

`109dccdabb3fb2bf571b866546d885e2514a4a2b`

## Exact config differences seen between the working kernel and the failed release-match build

The failed build differed in 46 Kconfig entries. The meaningful groups are:

- `CONFIG_USER_NS`: `n` -> `y`
- missing LZ4K/LZ4KD/LZ4K_OPLUS support
- missing 842 compressor support
- missing IPSet and several netfilter options
- ZRAM default compressor changed from `lzo` to `lzo-rle`

The first baseline build must keep `CONFIG_USER_NS=n` and reproduce all other working settings before any USER_NS experiment.

## Required baseline checks

The produced normal `Image` must contain:

```text
# CONFIG_USER_NS is not set
CONFIG_PID_NS=y
CONFIG_IPC_NS=y
CONFIG_SYSVIPC=y
CONFIG_POSIX_MQUEUE=y
CONFIG_NTSYNC=y
CONFIG_ZRAM=y
CONFIG_ZSMALLOC=y
CONFIG_ZRAM_DEF_COMP_LZO=y
CONFIG_IP_SET=y
CONFIG_IP_SET_HASH_IP=y
CONFIG_IP_SET_HASH_NET=y
CONFIG_NETFILTER_XT_SET=y
CONFIG_NETFILTER_XT_MATCH_ADDRTYPE=y
CONFIG_NETFILTER_XT_MATCH_RECENT=y
CONFIG_NETFILTER_XT_TARGET_LOG=y
CONFIG_LTO_CLANG_THIN=y
CONFIG_KSU=y
CONFIG_KSU_TRACEPOINT_HOOK=y
CONFIG_KSU_MULTI_MANAGER_SUPPORT=y
# CONFIG_KSU_SUSFS is not set
```

The target release string is:

```text
6.1.118uke-Droidspaces-ReSukiSU
```

## Test order

1. Build normal `Image` only.
2. Compare embedded config against the known-working `Image`.
3. Compare exported symbol CRCs/KMI.
4. Insert the kernel into the known-working 96 MiB boot template without changing `vendor_boot`, `dtbo`, `init_boot`, `vendor_dlkm`, or slot metadata.
5. Test with `termux-fastboot boot`.
6. Only after the baseline boots, create a twin build with the sole functional change `CONFIG_USER_NS=y`.
