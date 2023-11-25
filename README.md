<br>English | [简体中文](README_zh.md)

[![CI](https://github.com/gngpp/ninja/actions/workflows/CI.yml/badge.svg)](https://github.com/gngpp/ninja/actions/workflows/CI.yml)
[![CI](https://github.com/gngpp/ninja/actions/workflows/Release.yml/badge.svg)](https://github.com/gngpp/ninja/actions/workflows/Release.yml)
 <a target="_blank" href="https://github.com/gngpp/ninja/blob/main/LICENSE">
  <img src="https://img.shields.io/badge/license-GPL_3.0-blue.svg"/>
 </a>
  <a href="https://github.com/gngpp/ninja/releases">
    <img src="https://img.shields.io/github/release/gngpp/ninja.svg?style=flat">
  </a><a href="https://github.com/gngpp/ninja/releases">
    <img src="https://img.shields.io/github/downloads/gngpp/ninja/total?style=flat">
  </a>
  [![](https://img.shields.io/docker/image-size/gngpp/ninja)](https://registry.hub.docker.com/r/gngpp/ninja)
  [![Docker Image](https://img.shields.io/docker/pulls/gngpp/ninja.svg)](https://hub.docker.com/r/gngpp/ninja/)

# ninja

Reverse engineered `ChatGPT` proxy (bypass Cloudflare 403 Access Denied)

> If the project is helpful to you, please consider [donating support](https://github.com/gngpp/gngpp/blob/main/SPONSOR.md#sponsor-my-open-source-works) for continued project maintenance, or you can Pay for consulting and technical support services.

### Features

- API key acquisition
- Email/password account authentication (Google/Microsoft third-party login not supported)
- Supports obtaining RefreshToken
- `ChatGPT-API`/`OpenAI-API`/`ChatGPT-to-API` Http API proxy (for third-party client access)
- Support IP proxy pool (support using Ipv6 subnet as proxy pool)
- ChatGPT WebUI
- Very small memory footprint

> Limitations: This cannot bypass OpenAI's outright IP ban

### ArkoseLabs

Sending `GPT-4/GPT-3.5/Creating API-Key` dialog requires sending `Arkose Token` as a parameter. There are only two supported solutions for the time being.

1) Use HAR

- Supports HAR feature pooling, can upload multiple HARs at the same time, and use rotation training strategy

> The `ChatGPT` official website sends a `GPT-4` session message, and the browser `F12` downloads the `https://tcr9i.chat.openai.com/fc/gt2/public_key/35536E1E-65B4-4D96-9D97-6ADB7EFF8147` interface. HAR log file, use the startup parameter `--arkose-gpt4-har-dir` to specify the HAR directory path to use (if you do not specify a path, use the default path `~/.gpt4`, you can directly upload and update HAR ), the same method applies to `GPT-3.5` and other types. Supports WebUI to upload and update HAR, request path: `/har/upload`, optional upload authentication parameter: `--arkose-har-upload-key`

2) Use [YesCaptcha](https://yescaptcha.com/i/1Cc5i4) / [CapSolver](https://dashboard.capsolver.com/passport/register?inviteCode=y7CtB_a-3X6d)

> The platform performs verification code parsing, start the parameter `--arkose-solver` to select the platform (use `YesCaptcha` by default), `--arkose-solver-key` fill in `Client Key`

- Both solutions are used, the priority is: `HAR` > `YesCaptcha` / `CapSolver`
- `YesCaptcha` / `CapSolver` is recommended to be used with HAR. When the verification code is generated, the parser is called for processing. After verification, HAR is more durable.

> Currently OpenAI has updated `Login` which requires verification of `Arkose Token`. The solution is the same as `GPT-4`. Fill in the startup parameters and specify the HAR file `--arkose-auth-har-dir`. To create an API-Key, you need to upload the HAR feature file related to the Platform. The acquisition method is the same as above.

> Recently, `OpenAI` has canceled the `Arkose` verification for `GPT-3.5`. It can be used without uploading HAR feature files (uploaded ones will not be affected). After compatibility, `Arkose` verification may be turned on again, and startup parameters need to be added. `--arkose-gpt3-experiment` enables the `GPT-3.5` model `Arkose` verification processing, and the WebUI is not affected.

### Http Server

#### Public interface, `*` represents any `URL` suffix

- ChatGPT-API
  - `/public-api/*`
  - `/backend-api/*`
  
- OpenAI-API
  - `/v1/*`

- Platform-API
  - `/dashboard/*`

- ChatGPT-To-API
  - `/v1/chat/completions`
  > About using `ChatGPT` to `API`, use `AceessToken` directly as `API Key`

- Files-API
  - `/files/*`
  > Image and file upload and download API proxy, the API returned by the `/backend-api/files` interface has been converted to `/files/*`

- Authorization
  - Login: `/auth/token`, form `option` optional parameter, default is `web` login, returns `AccessToken` and `Session`; parameter is `apple`/`platform`, returns `AccessToken` and `RefreshToken`
  - Refresh `RefreshToken`: `/auth/refresh_token`
  - Revoke `RefreshToken`: `/auth/revoke_token`
  - Refresh `Session`: `/api/auth/session`, send a cookie named `__Secure-next-auth.session-token` to call refresh `Session`, and return a new `AccessToken`
  
  > `Web login`, a cookie named: `__Secure-next-auth.session-token` is returned by default. The client only needs to save this cookie. Calling `/api/auth/session` can also refresh `AccessToken`

  > About the method of obtaining `RefreshToken`, use the `ChatGPT App` login method of the `Apple` platform. The principle is to use the built-in MITM agent. When the `Apple device` is connected to the agent, you can log in to the `Apple platform` to obtain `RefreshToken`. It is only suitable for small quantities or personal use `(large quantities will seal the device, use with caution)`. For detailed usage, please see the startup parameter description.

  ```shell
  # Generate certificate
  ninja genca

  ninja run --pbind 0.0.0.0:8888

  # Set the network on your mobile phone to set your proxy listening address, for example: http://192.168.1.1:8888
  # Then open the browser http://192.168.1.1:8888/preauth/cert, download the certificate, install it and trust it, then open iOS ChatGPT and you can play happily
   ```

#### API documentation

- Platfrom API [doc](https://platform.openai.com/docs/api-reference)
- Backend API [doc](doc/rest.http)

#### Basic services

- ChatGPT WebUI
- Expose `ChatGPT-API`/`OpenAI-API` proxies
- `API` prefix is consistent with the official one
- `ChatGPT` to `API`
- Can access third-party clients
- Can access IP proxy pool to improve concurrency
- Supports obtaining RefreshToken
- Support file feature pooling in HAR format

#### Parameter Description

- `--level`, environment variable `LOG`, log level: default info
- `--bind`, environment variable `BIND`, service listening address: default 0.0.0.0:7999,
- `--tls-cert`, environment variable `TLS_CERT`', TLS certificate public key. Supported format: EC/PKCS8/RSA
- `--tls-key`, environment variable `TLS_KEY`, TLS certificate private key
- `--proxies`, proxy, supports proxy pool, multiple proxies are separated by `,`, format: protocol://user:pass@ip:port
  - Advanced usage
  > Used by built-in protocols and types of agents, built-in protocols: `all/api/auth/arkose`, where `all` is for all clients, `api` is for all `OpenAI API`, `auth` is for authorization/login, `arkose` is for ArkoseLabs; the type of proxy: `interface/proxy/ipv6_subnet`, where `interface` represents the bound export `IP` address, `proxy` represents the upstream proxy protocol: `http/https/socks5`, `ipv6_subnet` Indicates that a random IP address in the Ipv6 network segment is used as a proxy. Example: `all|socks5://192.168.1.1:1080, api|10.0.0.1, auth|2001:db8::/32, http://192.168.1.1:1081`, without built-in protocol `all/api /auth/arkose`, the default is `all`. `ipv6_subnet` is used by default when present.
- `--enable-direct`, enable direct connection, add the IP bound to the `interface` export to the proxy pool
- `--workers`, worker threads: default 1
- `--disable-webui`, if you don’t want to use the default built-in WebUI, use this parameter to turn it off
- `--enable-file-proxy`, environment variable `ENABLE_FILE_PROXY`, turns on the file upload and download API proxy

[...](https://github.com/gngpp/ninja/blob/main/README.md#command-manual)

### Install

- #### Ubuntu(Other Linux)

Making [Releases](https://github.com/gngpp/ninja/releases/latest) has a precompiled deb package, binaries, in Ubuntu, for example:

```shell
wget https://github.com/gngpp/ninja/releases/download/v0.8.6/ninja-0.8.6-x86_64-unknown-linux-musl.tar.gz
tar -xf ninja-0.8.6-x86_64-unknown-linux-musl.tar.gz
./ninja run
```

- #### OpenWrt

There are pre-compiled ipk files in GitHub [Releases](https://github.com/gngpp/ninja/releases/latest), which currently provide versions of aarch64/x86_64 and other architectures. After downloading, use opkg to install, and use nanopi r4s as example:

```shell
wget https://github.com/gngpp/ninja/releases/download/v0.8.6/ninja_0.8.6_aarch64_generic.ipk
wget https://github.com/gngpp/ninja/releases/download/v0.8.6/luci-app-ninja_1.1.6-1_all.ipk
wget https://github.com/gngpp/ninja/releases/download/v0.8.6/luci-i18n-ninja-zh-cn_1.1.6-1_all.ipk

opkg install ninja_0.8.6_aarch64_generic.ipk
opkg install luci-app-ninja_1.1.6-1_all.ipk
opkg install luci-i18n-ninja-zh-cn_1.1.6-1_all.ipk
```

- #### Docker

> Mirror source supports `gngpp/ninja:latest`/`ghcr.io/gngpp/ninja:latest`

```shell
docker run --rm -it -p 7999:7999 --name=ninja \
  -e WORKERS=1 \
  -e LOG=info \
  ghcr.io/gngpp/ninja:latest run
```

- Docker Compose

> `CloudFlare Warp` is not supported in your region (China), please delete it, or if your `VPS` IP can be directly connected to `OpenAI`, you can also delete it

```yaml
version: '3'

services:
  ninja:
    image: gngpp/ninja:latest
    container_name: ninja
    restart: unless-stopped
    environment:
      - TZ=Asia/Shanghai
      - PROXIES=socks5://warp:10000
    command: run
    ports:
      - "8080:7999"
    depends_on:
      - warp

  warp:
    container_name: warp
    image: ghcr.io/gngpp/warp:latest
    restart: unless-stopped

  watchtower:
    container_name: watchtower
    image: containrrr/watchtower
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: --interval 3600 --cleanup
    restart: unless-stopped

```

### Command Manual

```shell
$ ninja --help
Reverse engineered ChatGPT proxy

Usage: ninja [COMMAND]

Commands:
  run      Run the HTTP server
  stop     Stop the HTTP server daemon
  start    Start the HTTP server daemon
  restart  Restart the HTTP server daemon
  status   Status of the Http server daemon process
  log      Show the Http server daemon log
  genca    Generate MITM CA certificate
  gt       Generate config template file (toml format file)
  update   Update the application
  help     Print this message or the help of the given subcommand(s)

Options:
  -h, --help     Print help
  -V, --version  Print version

$ ninja run --help
Run the HTTP server

Usage: ninja run [OPTIONS]

Options:
  -L, --level <LEVEL>
          Log level (info/debug/warn/trace/error) [env: LOG=] [default: info]
  -C, --config <CONFIG>
          Configuration file path (toml format file) [env: CONFIG=]
  -b, --bind <BIND>
          Server bind address [env: BIND=] [default: 0.0.0.0:7999]
  -W, --workers <WORKERS>
          Server worker-pool size (Recommended number of CPU cores) [default: 1]
      --concurrent-limit <CONCURRENT_LIMIT>
          Enforces a limit on the concurrent number of requests the underlying [default: 1024]
      --enable-direct
          Enable direct connection [env: ENABLE_DIRECT=]
  -x, --proxies <PROXIES>
          Request client proxy, support multiple proxy, use ',' to separate
          Format: proto|type
          Proto: all/api/auth/arkose, default: all
          Type: interface/proxy/ipv6 subnet，proxy type only support: socks5/http/https
          Example: all|socks5://192.168.1.1:1080, api|10.0.0.1, auth|2001:db8::/32, http://192.168.1.1:1081 [env: PROXIES=]
      --cookie-store
          Enabled Cookie Store [env: COOKIE_STORE=]
      --timeout <TIMEOUT>
          Client timeout (seconds) [default: 360]
      --connect-timeout <CONNECT_TIMEOUT>
          Client connect timeout (seconds) [default: 20]
      --tcp-keepalive <TCP_KEEPALIVE>
          TCP keepalive (seconds) [default: 60]
      --pool-idle-timeout <POOL_IDLE_TIMEOUT>
          Set an optional timeout for idle sockets being kept-alive [default: 90]
      --tls-cert <TLS_CERT>
          TLS certificate file path [env: TLS_CERT=]
      --tls-key <TLS_KEY>
          TLS private key file path (EC/PKCS8/RSA) [env: TLS_KEY=]
      --cf-site-key <CF_SITE_KEY>
          Cloudflare turnstile captcha site key [env: CF_SECRET_KEY=]
      --cf-secret-key <CF_SECRET_KEY>
          Cloudflare turnstile captcha secret key [env: CF_SITE_KEY=]
  -A, --auth-key <AUTH_KEY>
          Login Authentication Key [env: AUTH_KEY=]
  -D, --disable-webui
          Disable WebUI [env: DISABLE_WEBUI=]
  -F, --enable-file-proxy
          Enable file proxy [env: ENABLE_FILE_PROXY=]
      --arkose-endpoint <ARKOSE_ENDPOINT>
          Arkose endpoint, Example: https://client-api.arkoselabs.com
  -E, --arkose-gpt3-experiment
          Enable Arkose GPT-3.5 experiment
      --arkose-gpt3-har-dir <ARKOSE_GPT3_HAR_DIR>
          About the browser HAR directory path requested by ChatGPT GPT-3.5 ArkoseLabs
      --arkose-gpt4-har-dir <ARKOSE_GPT4_HAR_DIR>
          About the browser HAR directory path requested by ChatGPT GPT-4 ArkoseLabs
      --arkose-auth-har-dir <ARKOSE_AUTH_HAR_DIR>
          About the browser HAR directory path requested by Auth ArkoseLabs
      --arkose-platform-har-dir <ARKOSE_PLATFORM_HAR_DIR>
          About the browser HAR directory path requested by Platform ArkoseLabs
  -K, --arkose-har-upload-key <ARKOSE_HAR_UPLOAD_KEY>
          HAR file upload authenticate key
  -s, --arkose-solver <ARKOSE_SOLVER>
          About ArkoseLabs solver platform [default: yescaptcha]
  -k, --arkose-solver-key <ARKOSE_SOLVER_KEY>
          About the solver client key by ArkoseLabs
  -T, --tb-enable
          Enable token bucket flow limitation
      --tb-store-strategy <TB_STORE_STRATEGY>
          Token bucket store strategy (mem/redis) [default: mem]
      --tb-redis-url <TB_REDIS_URL>
          Token bucket redis connection url [default: redis://127.0.0.1:6379]
      --tb-capacity <TB_CAPACITY>
          Token bucket capacity [default: 60]
      --tb-fill-rate <TB_FILL_RATE>
          Token bucket fill rate [default: 1]
      --tb-expired <TB_EXPIRED>
          Token bucket expired (seconds) [default: 86400]
  -B, --pbind <PBIND>
          Preauth MITM server bind address [env: PREAUTH_BIND=]
  -X, --pupstream <PUPSTREAM>
          Preauth MITM server upstream proxy, Only support http/https/socks5 protocol [env: PREAUTH_UPSTREAM=]
      --pcert <PCERT>
          Preauth MITM server CA certificate file path [default: ca/cert.crt]
      --pkey <PKEY>
          Preauth MITM server CA private key file path [default: ca/key.pem]
  -h, --help
          Print help
```

### Platform Support

- Linux
  - `x86_64-unknown-linux-musl`
  - `aarch64-unknown-linux-musl`
  - `armv7-unknown-linux-musleabi`
  - `armv7-unknown-linux-musleabihf`
  - `arm-unknown-linux-musleabi`
  - `arm-unknown-linux-musleabihf`
  - `armv5te-unknown-linux-musleabi`
- Windows
  - `x86_64-pc-windows-msvc`
- MacOS
  - `x86_64-apple-darwin`
  - `aarch64-apple-darwin`

### Compile

- Linux compile, Ubuntu machine for example:

```shell
apt install build-essential
apt install cmake
apt install libclang-dev

git clone https://github.com/gngpp/ninja.git && cd ninja
cargo build --release
```

- OpenWrt Compile

```shell
cd package
svn co https://github.com/gngpp/ninja/trunk/openwrt
cd -
make menuconfig # choose LUCI->Applications->luci-app-ninja  
make V=s
```

### Instructions

- Open source projects can be modified, but please keep the original author information to avoid losing technical support.
- Project is standing on the shoulders of other giants, thanks!
- Submit an issue if there are errors, bugs, etc., and I will fix them.

### Preview

![img0](./doc/img/img0.png)
![img1](./doc/img/img1.png)