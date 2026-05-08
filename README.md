# carrot-nas

carrot-nas 是一个面向家庭场景的自建 NAS 私有云控制层，目标是把普通 x86 主机、Linux、Docker 和成熟开源应用组合成可管理的家庭私有云节点。

阶段一聚焦本地 Debian 服务器：接入已有媒体资源，统一部署并管理 Immich、Jellyfin、FileBrowser 等服务，提供设备状态、资源来源状态、应用状态和访问入口。

## 当前状态

项目处于 V0.1 需求设计阶段。

## 文档

- [Roadmap](roadmap.md)
- [V0.1 阶段一详细需求](docs/phase-1-requirements.md)

## V0.1 方向

```text
Debian NAS 服务器
  -> Windows SMB 只读验证资源来源
  -> Immich / Jellyfin / FileBrowser
  -> NAS Agent
  -> 本地 Web 管理台
```

