# OrayBox X1 构建说明

本仓库的 `Build OrayBox X1` 工作流为 MediaTek MT7628 / MIPS 24KEc 编译独立版本：32 位小端、O32 ABI、软浮点、静态 musl 链接，使用 `-Os` 减小体积。

## 在 GitHub Actions 编译

1. 打开仓库的 **Actions → Build OrayBox X1**。
2. 点击 **Run workflow**，选择所需分支（默认 `main`），运行。
3. 成功后在该次运行的 **Artifacts** 下载 `hev-socks5-tunnel-oray-x1`。

工作流仅手动触发，不创建 Release 或推送 Docker 镜像。产物保留 30 天。编译器固定到 `cross-tools/musl-cross` 的 `20261001` 版本，并校验下载文件的 SHA-256。

产物包含二进制 `hev-socks5-tunnel-oray-x1`、校验和、上游许可证、源码提交与构建信息，以及 ELF 架构和版本检查报告。

构建会检查 ELF32、小端、MIPS、软浮点和无动态加载器，并用 QEMU 的 `24KEc` CPU 执行 `--version`。这验证程序可加载和启动，不替代真实设备上的 TUN、路由和代理连通性测试。

## 在设备上验证与安装

下载解压后，先检查完整性，再上传到 `/tmp` 测试，不直接覆盖运行中的程序：

```sh
sha256sum -c SHA256SUMS
scp hev-socks5-tunnel-oray-x1 oray:/tmp/hev-socks5-tunnel-oray-x1
ssh oray
chmod 700 /tmp/hev-socks5-tunnel-oray-x1
/tmp/hev-socks5-tunnel-oray-x1 --version
```

上游程序打印版本后返回 `255`，这是当前源码的设计，不代表启动失败。

确认可执行后，在设备上停止原代理并备份，再替换程序：

```sh
/root/hev-manager.sh stop
cp -p /root/hev-socks5-tunnel /root/hev-socks5-tunnel.previous
cp /tmp/hev-socks5-tunnel-oray-x1 /root/hev-socks5-tunnel
chmod 700 /root/hev-socks5-tunnel
/root/hev-manager.sh start
```

保留现有 `/root/hev.yml`，代理地址和认证信息不会随二进制构建或替换而改变。真实代理服务不可用时，更换二进制不一定能恢复上网。

如果新版本出现问题，停止后复制备份恢复并重新启动。替换或重启使用映射 DNS 的 HEV 后，客户端可能需要重新连接 Wi-Fi 或刷新 DNS 缓存。

其他 Oray 型号使用该产物前应先确认 CPU、字节序、ABI、内核和 TUN 支持，不能仅凭品牌相同判断兼容性。
