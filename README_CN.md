[EN](README.md) | 中文

# voxl-releases

[voxl](https://github.com/tembo/voxl) 的 Linux x86_64 编译版。直接下载二进制，
不用装 Rust、不用从源码编译。

voxl 是一个快速远程桌面流工具：把 X11 显示器以 H.264 通过一条 WebSocket
推到浏览器观看，不用装客户端。上游项目（MIT）：
https://github.com/tembo/voxl

## 下载

去 [Releases](https://github.com/popeico/voxl-releases/releases) 下载对应版本：

- `voxl-linux-x86_64` —— 二进制文件（可改名为 `voxl`）
- `voxl-linux-x86_64.sha256` —— 校验文件
- `voxl-linux-x86_64-musl` —— 动态链接版，用于 musl 系统（Alpine
  等），需要装运行库（见下）
- `voxl-linux-x86_64-musl.sha256` —— musl 版校验文件
- `build-info.txt` / `build-info-musl.txt` —— 两个版本各自的上游
  commit、版本号、构建日期、sha256

校验下载：

```sh
sha256sum -c voxl-linux-x86_64.sha256
chmod +x voxl-linux-x86_64
./voxl-linux-x86_64 --version
```

musl 版校验方法相同（`sha256sum -c voxl-linux-x86_64-musl.sha256`，
先装好下面的运行库再运行）。

## 运行环境

Linux x86_64，需要一个要推流的 X11 显示器。glibc 版二进制动态链接
音视频库，先装好（Debian / Ubuntu）：

```sh
sudo apt-get install -y libx264-dev libyuv-dev libopus-dev libpulse-dev \
  libxcb1-dev libxcb-shm0-dev libxcb-randr0-dev libxcb-xfixes0-dev \
  libxcb-shape0-dev
```

`voxl-linux-x86_64-musl` 是针对 musl libc 动态链接的版本，目标系统
上同样需要装这些库（Alpine）：

```sh
apk add opus pulseaudio x264 libyuv libxcb
```

为什么不是全静态？Alpine 没有提供静态版的 libopus / libpulse——
既没有 opus-static / pulseaudio-static 子包，opus-dev /
pulseaudio-dev 里也没有 .a（已对照 v3.22 APKINDEX 核实）。全静态
的 musl 版需要从源码为 musl 编译 opus、pulseaudio 及其依赖，这里
没有做。musl 版的 `file` / `ldd` 输出记录在 `build-info-musl.txt`
中。

不需要 Rust、Node，也不用编译——编译已经在这里完成。

## 快速开始

在要推流的机器上（这里显示器是 `:1`）：

```sh
DISPLAY=:1 ./voxl-linux-x86_64 --port=6901
```

它会打印一个带登录 token 的链接，用浏览器打开。URL 后加 `?stats`
可以看到帧率 / 码率。全部参数见上游
[README](https://github.com/tembo/voxl#readme)
（`--title`、`--framerate`、`--enable-basic-auth`、剪贴板、录制等）。

通过公网域名（反代 / 隧道）提供服务时，两个参数缺一不可，否则浏览器
WebSocket 会被拒绝：

```sh
DISPLAY=:1 ./voxl-linux-x86_64 --port=6901 \
  --allowed-hosts=<你的域名> --allowed-origins=https://<你的域名>
```

## 故障排查

- **流一直起不来 / 浏览器 WS 被拒绝：** 缺了 `--allowed-origins`
  （见上）。公网域名下只给 `--allowed-hosts` 不够。
- **启动时编码器退出（`height not divisible by 2`）：** 显示器分辨率
  宽或高是奇数。改成偶数分辨率，例如 1280x720 或 1920x1080。
- **日志反复出现 `pulse connect failed`：** 会话里没有 PulseAudio，
  所以没有声音，视频不受影响。
- **`error while loading shared libraries`：** 上面列表里的某个运行库
  没装，装好再试。

## 关于这些构建

- 源码：[tembo/voxl](https://github.com/tembo/voxl)（MIT 协议）。
  每个 release 都注明构建所用的上游 commit。
- 构建定义在私有仓库 `popeico/build-factory`（`voxl/` 源码镜像 +
  `.github/workflows/voxl.yml`）。本仓库只放 release 文件，没有源码镜像。
- 上游不提供预编译二进制（只有 `.crate` 源码包），所以才有这些构建。
  上游一旦开始发布预编译版，这些构建就会停止。
