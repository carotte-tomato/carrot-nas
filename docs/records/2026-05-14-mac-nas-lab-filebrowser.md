# 2026-05-14 Mac NAS Lab FileBrowser 运行记录

记录时间：2026-05-14
时区：Asia/Shanghai
对应文档：`docs/v0.1-mac-nas-prepare.md`

---

## 1. 目标

本次事务目标是先不进入正式 V0.1，实现一个 Mac 本机 NAS Lab 的最小闭环：

```text
Mac 本机
  -> Colima Linux VM
  -> Docker / Docker Compose
  -> FileBrowser
  -> 浏览器访问 http://localhost:8081
```

本次暂不继续 Jellyfin、Immich、Debian VM 和 Windows SMB。

---

## 2. 2026-05-13：Prepare 文档归档

新增准备阶段文档：

```text
docs/v0.1-mac-nas-prepare.md
```

文档结论：

```text
先走 Mac Docker 本机 NAS Lab
第一闭环优先 FileBrowser + Jellyfin
Immich 暂缓
后续再迁移到 Debian VM 和正式 V0.1 路径
```

---

## 3. 2026-05-13：Homebrew 修复

检查结果：

```text
Mac 架构：arm64
macOS：26.4.1
Docker：未安装
Docker Desktop / OrbStack / Colima：均未安装
Homebrew：4.0.20，无法识别 macOS 26.4.1
```

执行动作：

```bash
brew update
```

结果：

```text
Homebrew 更新到 5.1.11
Homebrew 可以识别 macOS 26.4.1
旧的 homebrew/core 和 homebrew/cask tap 缓存被 Homebrew 自动清理
```

---

## 4. 2026-05-14：安装命令行 Docker 路线

执行动作：

```bash
brew install docker docker-compose colima
```

安装结果：

```text
docker：29.4.3
docker-compose：5.1.3
colima：0.10.1
lima：2.1.1
```

Compose 插件初始问题：

```text
docker: unknown command: docker compose
```

原因：

```text
Homebrew 安装的 docker-compose 是 Docker CLI 插件
Docker CLI 需要知道插件目录
```

修复动作：

```text
创建 ~/.docker/config.json
写入 cliPluginsExtraDirs = /opt/homebrew/lib/docker/cli-plugins
```

验证结果：

```text
Docker Compose version 5.1.3
```

---

## 5. 2026-05-14 00:05：启动 Colima 首次失败

第一次启动命令：

```bash
colima start --cpu 2 --memory 4 --disk 30 --vm-type vz --mount-type virtiofs
```

配置含义：

```text
CPU：2
内存：4GB
磁盘：30GB
虚拟化：macOS Virtualization.Framework
挂载：virtiofs
```

失败原因：

```text
下载 Colima 基础镜像 ubuntu-24.04-minimal-cloudimg-arm64-docker.qcow2 时 unexpected EOF
```

处理动作：

```text
确认 Colima 未运行
确认 Docker daemon 未运行
确认 ~/.colima 实际占用约 20K
清理残留 daemon 进程
```

---

## 6. 2026-05-14 00:11：代理后启动 Colima 成功

再次启动时，Colima 日志显示检测到代理环境：

```text
http_proxy=http://127.0.0.1:7890
https_proxy=http://127.0.0.1:7890
```

Colima 自动替换为 VM 内可访问地址：

```text
http://192.168.5.2:7890
https://192.168.5.2:7890
```

启动结果：

```text
Colima：running
架构：aarch64
runtime：docker
mountType：virtiofs
docker socket：unix://${HOME}/.colima/default/docker.sock
Docker context：colima
```

Docker 验证：

```text
Docker CLI：29.4.3 darwin/arm64
Docker Server：29.2.1 linux/arm64
Docker Compose：5.1.3
```

---

## 7. 2026-05-14：创建 Mac NAS Lab 目录

创建目录：

```text
${HOME}/carrot-nas-lab/
  media/
    photos/
    videos/
    files/
  appdata/
    filebrowser/
    jellyfin/
    immich/
  compose/
```

创建测试文件：

```text
${HOME}/carrot-nas-lab/media/files/hello.txt
```

测试文件内容：

```text
hello from carrot-nas Mac NAS Lab
```

---

## 8. 2026-05-14：FileBrowser Compose

最初临时 Compose 文件放在：

```text
${HOME}/carrot-nas-lab/compose/filebrowser.compose.yml
```

后续按项目约定归档到仓库内：

```text
compose/mac-lab/filebrowser.compose.yml
```

归档后，home 目录里的临时 Compose 文件已删除，后续以项目内文件为准。

挂载关系：

```text
${HOME}/carrot-nas-lab/media/files      -> /srv:ro
${HOME}/carrot-nas-lab/appdata/filebrowser -> /database
```

设计取舍：

```text
media/files 只读挂载，模拟 V0.1 资源来源只读策略
appdata/filebrowser 可写，用于保存 FileBrowser 数据库
用户文件和应用数据分离
```

---

## 9. 2026-05-14 00:20：FileBrowser 首次拉取失败

启动命令：

```bash
docker compose -f ${HOME}/carrot-nas-lab/compose/filebrowser.compose.yml up -d
```

失败原因：

```text
访问 Docker Hub registry-1.docker.io 时 TLS handshake timeout
```

判断：

```text
Colima VM 已有代理环境变量
Docker daemon 自身 systemd Environment 为空
但本次先按网络波动处理，没有修改 Docker daemon 代理配置
```

---

## 10. 2026-05-14 00:27：FileBrowser 重试成功

再次执行：

```bash
docker compose -f ${HOME}/carrot-nas-lab/compose/filebrowser.compose.yml up -d
```

结果：

```text
镜像 filebrowser/filebrowser:latest 拉取成功
容器 carrot-filebrowser 创建成功
容器 carrot-filebrowser 启动成功
```

容器状态：

```text
carrot-filebrowser
Up / healthy
0.0.0.0:8081->80/tcp
```

页面验证：

```text
curl http://localhost:8081 -> HTTP 200
```

FileBrowser 版本：

```text
File Browser v2.63.3
```

首次登录信息：

```text
username: admin
password: X2ToyyVP2DloK6mV
```

---

## 11. 当前成果

已完成：

```text
命令行 Docker 路线安装完成
Colima 可运行 Docker daemon
Docker Compose 可用
Mac NAS Lab 目录创建完成
FileBrowser 最小服务跑通
FileBrowser 可通过 http://localhost:8081 访问
项目内已归档 Mac Lab FileBrowser Compose 文件
```

尚未继续：

```text
Jellyfin
Immich
Docker daemon 代理持久化配置
Docker Hub registry mirror
Debian VM
Windows SMB
NAS Agent
Web 管理台
```

---

## 12. 后续恢复方式

如果 Colima 已停止，先启动：

```bash
colima start
```

再启动 FileBrowser：

```bash
docker compose -f compose/mac-lab/filebrowser.compose.yml up -d
```

访问：

```text
http://localhost:8081
```

停止 FileBrowser：

```bash
docker compose -f compose/mac-lab/filebrowser.compose.yml down
```

停止 Colima 虚机：

```bash
colima stop
```

---

## 13. 2026-05-14 00:33：暂停事务并停止虚机

按暂停要求执行：

```bash
docker compose -f compose/mac-lab/filebrowser.compose.yml down
colima stop
```

随后验证：

```text
colima is not running
```

当前保留：

```text
项目内 Compose 模板：compose/mac-lab/filebrowser.compose.yml
Mac NAS Lab 数据目录：${HOME}/carrot-nas-lab
Colima 配置和已下载镜像缓存
```

补充清理：

```text
由于最早的 FileBrowser 容器由 home 目录临时 compose 创建，
后续短暂启动 Colima，执行 docker rm -f carrot-filebrowser 删除旧容器，
再执行 colima stop 停回虚机。
```

当前未继续：

```text
Jellyfin
Immich
Debian VM
Windows SMB
```
