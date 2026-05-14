# 2026-05-15 Mac NAS Lab Jellyfin 运行记录

记录时间：2026-05-15
时区：Asia/Shanghai
对应文档：`docs/v0.1-mac-nas-prepare.md`

---

## 1. 目标

本次事务继续 V0.1 prepare 的 Mac NAS Lab：

```text
Mac 本机
  -> Colima Linux VM
  -> Docker / Docker Compose
  -> Jellyfin
  -> 浏览器访问 http://localhost:8096
```

目标是让 Jellyfin 读取本机实验视频目录，并继续验证媒体资源目录和应用数据目录的边界。

---

## 2. Compose 模板

项目内新增模板：

```text
compose/mac-lab/jellyfin.compose.yml
```

挂载关系：

```text
${HOME}/carrot-nas-lab/appdata/jellyfin/config -> /config
${HOME}/carrot-nas-lab/appdata/jellyfin/cache  -> /cache
${HOME}/carrot-nas-lab/media/videos            -> /media/videos:ro
```

端口：

```text
localhost:8096 -> container:8096
```

设计取舍：

```text
media/videos 只读挂载，模拟 V0.1 资源来源只读策略
appdata/jellyfin/config 保存 Jellyfin 配置
appdata/jellyfin/cache 保存 Jellyfin 缓存和运行期生成内容
用户原始视频和应用数据分离
```

---

## 3. 启动命令

```bash
colima start
docker compose -f compose/mac-lab/jellyfin.compose.yml up -d
```

验证命令：

```bash
docker ps --filter name=carrot-jellyfin
curl -I http://localhost:8096
```

---

## 4. Jellyfin 初始设置建议

首次打开：

```text
http://localhost:8096
```

媒体库建议：

```text
类型：Movies 或 Mixed Movies and Shows
路径：/media/videos
```

这个路径对应 Mac 侧：

```text
${HOME}/carrot-nas-lab/media/videos
```

---

## 5. 2026-05-15：Docker Hub 拉取异常与 GHCR 切换

首次使用 Docker Hub 镜像：

```text
jellyfin/jellyfin:latest
```

遇到两类网络错误：

```text
TLS handshake timeout
unexpected EOF
```

排查结论：

```text
Colima VM 的用户 shell 有代理环境变量
Docker daemon 初始 Environment 为空
宿主机和 Colima VM shell 均可访问 registry
Docker daemon 拉取 Docker Hub 镜像时网络链路不稳定
```

处理：

```text
给 Colima VM 内 Docker daemon 增加 systemd proxy drop-in
重启 Docker daemon
改用 Jellyfin 官方 GHCR 镜像 ghcr.io/jellyfin/jellyfin:latest
```

GHCR 镜像拉取成功：

```text
Digest: sha256:93227545077893cc9516f28b3adb733b67bc4691f41b6167428a2a0e3220b81c
```

---

## 6. 2026-05-15：启动与验收

启动成功：

```text
container: carrot-jellyfin
image: ghcr.io/jellyfin/jellyfin:latest
status: running / healthy
port: 0.0.0.0:8096->8096/tcp
```

HTTP 验证：

```text
http://localhost:8096      -> 302 Location: web/
http://localhost:8096/web/ -> 200 OK
```

Jellyfin 日志确认：

```text
Kestrel is listening on 0.0.0.0
FFmpeg: /usr/lib/jellyfin-ffmpeg/ffmpeg
Core startup complete
```

应用数据目录已生成：

```text
${HOME}/carrot-nas-lab/appdata/jellyfin/config
${HOME}/carrot-nas-lab/appdata/jellyfin/cache
```

测试视频：

```text
${HOME}/carrot-nas-lab/media/videos/mac-lab-test-video.mp4
```

该文件由 Jellyfin 镜像内自带 ffmpeg 生成，用于首次创建 `/media/videos` 媒体库后验证播放。

---

## 7. 今日事务总结

本次完成了 Mac NAS Lab 的 Jellyfin prepare：

```text
新增 Jellyfin Compose 模板
启动 Jellyfin 并完成 HTTP 入口验证
生成一个测试视频用于媒体库播放验证
确认 /media/videos 是只读媒体资源目录
确认 /config 和 /cache 写入 appdata/jellyfin
确认首次使用时播放入口在首页媒体库，不在 Dashboard 配置入口
```

今天沉淀的项目文件：

```text
compose/mac-lab/jellyfin.compose.yml
docs/records/2026-05-15-mac-nas-lab-jellyfin.md
README.md
```

当前本机实验数据保留：

```text
${HOME}/carrot-nas-lab/appdata/jellyfin/config
${HOME}/carrot-nas-lab/appdata/jellyfin/cache
${HOME}/carrot-nas-lab/media/videos/mac-lab-test-video.mp4
```

结束时已恢复环境：

```text
docker compose -f compose/mac-lab/jellyfin.compose.yml down
docker compose -f compose/mac-lab/filebrowser.compose.yml down
colima stop
```

验证结果：

```text
无运行中的 Docker 容器
Colima：not running
```
