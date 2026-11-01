# hostmngr for OpenWrt 24.10.0 (x86_64)

GitHub Actions project to build the IOPSYS `hostmngr` package for the official
OpenWrt 24.10.0 x86/64 SDK. Wi-Fi integration is deliberately disabled:
`HOSTMNGR_PLATFORM_HAS_WIFI` is not enabled, and `libwifi-7` / `libwifiutils`
are not built or linked.

The build uses the upstream IOPSYS OpenWrt feed package recipes for both
`hostmngr` and `libeasy`, rather than inventing a separate upstream build
system. The hostmngr source documents that Wi-Fi libraries are required only
when `HOSTMNGR_PLATFORM_HAS_WIFI=y`.

## Build online with GitHub Actions

1. Create an empty GitHub repository.
2. Extract this project into the repository root.
3. Commit and push the files.
4. Open **Actions** and run **Build hostmngr (OpenWrt 24.10.0 x86_64)**, or
   push a commit to `main` / `master`.
5. Download the `hostmngr-openwrt-24.10.0-x86_64` artifact from the completed run.

The workflow downloads the matching official SDK, adds the IOPSYS `devel`
feed, installs its `libeasy` and `hostmngr` package definitions, compiles
`libeasy` first and then `hostmngr`, and uploads the resulting IPK files plus
build metadata.

## Target / compatibility

- OpenWrt release: **24.10.0**
- Target: **x86/64**
- SDK toolchain: **GCC 13.3.0 / musl**
- Wi-Fi feature: **disabled**
- Package format: **`.ipk`** (OpenWrt 24.10 uses opkg)

Use this build only on a router running the matching OpenWrt 24.10.0 x86/64
release. Do not install packages built for a different target, release, or
kernel ABI.

## Installing on the router

Copy the downloaded IPKs to the router, then install the hostmngr package:

```sh
opkg install ./libeasy_*.ipk
opkg install ./hostmngr_*.ipk
```

If the hostmngr package declares dependencies that are not already installed,
`opkg` may need access to the configured OpenWrt package feeds to fetch those
dependencies. The workflow also saves every IPK found in the SDK package output
tree, so inspect the artifact before installation.

Check the daemon and ubus object:

```sh
/etc/init.d/hostmngr status
ubus -v list hosts
ubus call hosts show
```

The exact init-script name and package split are determined by the upstream
feed recipe. If the package does not install an init script, start it using the
upstream package's documented service mechanism.

## Build inputs and upstream sources

- hostmngr source: <https://dev.iopsys.eu/network/hostmngr>
- IOPSYS OpenWrt package feed: <https://dev.iopsys.eu/feed/iopsys>
- OpenWrt 24.10.0 x86/64 SDK:
  <https://downloads.openwrt.org/releases/24.10.0/targets/x86/64/>

The IOPSYS hostmngr README lists `libeasy`, OpenWrt UCI/ubox/ubus libraries,
JSON-C, netlink, and netfilter libraries as dependencies. `libwifi-7` and
`libwifiutils` are conditional Wi-Fi dependencies and are intentionally
excluded from this configuration.

## Troubleshooting

- **Feed clone fails:** verify that the public IOPSYS feed is reachable from
  GitHub-hosted runners. If IOPSYS changes its feed branch or access policy,
  update `IOPSYS_FEED_REF` in the workflow.
- **Package path differs:** the workflow prints installed feed package paths
  and fails with a clear error if the feed no longer exposes `hostmngr` and
  `libeasy` under those names.
- **Missing dependency at install time:** run `opkg update` on the router and
  retry. The OpenWrt repository must match the router's exact 24.10.0 release
  and x86/64 target.
- **Source has changed:** this project tracks the IOPSYS feed's `devel` branch
  by default. For reproducible builds, set `IOPSYS_FEED_REF` to a reviewed
  immutable commit or branch in the workflow.

This project creates a build workflow; it does not itself contain prebuilt
binaries. A successful GitHub Actions run is required to produce verified IPKs.
