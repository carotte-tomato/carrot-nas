# carrot-nas

carrot-nas 是一个面向家庭场景的自建 NAS 私有云控制层，目标是把普通 x86 主机、Linux、Docker 和成熟开源应用组合成可管理的家庭私有云节点。

阶段一聚焦本地 Debian 服务器：接入已有媒体资源，统一部署并管理 Immich、Jellyfin、FileBrowser 等服务，提供设备状态、资源来源状态、应用状态和访问入口。

## 当前状态

项目处于 V0.1 需求设计与准备阶段。

当前除了正式 Debian NAS 节点方案外，已开始用当前 Mac 搭建本机 NAS Lab，用于提前熟悉 Docker、Compose、目录挂载、应用数据目录和 FileBrowser 等基础服务。

## 文档

- [Roadmap](roadmap.md)
- [V0.1 阶段一详细需求](docs/phase-1-requirements.md)
- [V0.1 Prepare：Mac 本机 NAS 服务器熟悉路径](docs/v0.1-mac-nas-prepare.md)
- [2026-05-14 Mac NAS Lab FileBrowser 运行记录](docs/records/2026-05-14-mac-nas-lab-filebrowser.md)

## V0.1 方向

```text
Debian NAS 服务器
  -> Windows SMB 只读验证资源来源
  -> Immich / Jellyfin / FileBrowser
  -> NAS Agent
  -> 本地 Web 管理台
```

## Mac NAS Lab

Mac NAS Lab 是 V0.1 prepare 的本机实验环境，不是最终部署形态。当前已沉淀 FileBrowser 的 Compose 模板：

```text
compose/mac-lab/filebrowser.compose.yml
```

恢复实验环境：

```bash
colima start
docker compose -f compose/mac-lab/filebrowser.compose.yml up -d
```

访问入口：

```text
http://localhost:8081
```

停止实验环境：

```bash
docker compose -f compose/mac-lab/filebrowser.compose.yml down
colima stop
```
