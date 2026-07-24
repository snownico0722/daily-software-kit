# daily-software-kit · 日用合集

个人日常使用的软件配置与脚本搜集库。后续会按主题分目录补充。

---

## 目录结构

```text
daily-software-kit/
├── README.md
└── clash/                    # Clash / Mihomo + ZeroOmega
    ├── clash全局扩展脚本-v1.1.txt
    ├── clash全局扩展覆写配置示例.txt
    ├── clash脚本说明.txt
    └── ZeroOmegaOptions-v1.bak
```

---

## clash/ · Clash / Mihomo 全局扩展 + ZeroOmega

自用代理分流方案：用 **Clash / Mihomo 全局扩展脚本**覆写机场下发规则，再用 **ZeroOmega** 处理浏览器里匹配不到的站点。

机场默认往往「全站 Steam 走代理 + 不认识的链接也全走代理」，既慢又浪费。这里用一套更克制的规则：需要代理的才代理，其余尽量直连。

| 文件 | 说明 |
|------|------|
| [`clash/clash全局扩展脚本-v1.1.txt`](clash/clash全局扩展脚本-v1.1.txt) | Mihomo 全局扩展脚本（核心） |
| [`clash/clash全局扩展覆写配置示例.txt`](clash/clash全局扩展覆写配置示例.txt) | 前置 / 后置 / 删除规则的配置示例 |
| [`clash/clash脚本说明.txt`](clash/clash脚本说明.txt) | 设计思路与使用备注 |
| [`clash/ZeroOmegaOptions-v1.bak`](clash/ZeroOmegaOptions-v1.bak) | ZeroOmega 配置备份（可导入） |

### 设计思路

1. **AI 流量** → `AI定向代理`（可单独切美国等节点）
2. **其他需代理域名（GFW 列表）** → `默认代理服务`
3. **浏览器经 7897 进来的 Cloudflare（ASN 13335）** → `Cloudflare定向代理`（方便过 CF 盾）
4. **Steam 社区网页 / 头像等** → `Steam社区代理`；下载、更新、游戏进程默认 **直连**
5. **最终未命中** → `未命中流量代理`（默认 `DIRECT`）
6. **浏览器仍打不开的站点** → 用 ZeroOmega 扔到 **7898 强制代理**（或 7899 强制直连）

浏览器侧也可挂 GFW 判断，方便在扩展图标上看出当前站点走哪条策略（非必须，偏可视化）。

### 策略组一览

脚本会**清空机场原有策略组**，重建为：

| 策略组 | 类型 | 作用 |
|--------|------|------|
| 默认代理服务 | select | 主代理入口 |
| AI定向代理 | select | AI 相关规则集 |
| Cloudflare定向代理 | select | 浏览器 CF 流量 |
| Steam社区代理 | select | Steam 社区页 / 头像等 |
| 浏览器转发代理 | select | ZeroOmega 7898 强制代理 |
| 最快节点 | url-test | 自动测速 |
| 故障转移 | fallback | 节点挂了自动切 |
| 未命中流量代理 | select | `MATCH` 兜底（默认直连） |

节点来源同时支持：

- 配置里的 `proxies`
- `proxy-providers`（会尽量保留并启用健康检查）

### 内置规则摘要

**Steam 社区（走代理）**

- `steamcommunity.com` 及社区 CDN / 头像 / 用户图等

**Steam 下载与进程（直连）**

- `steamcontent.com`、`steampowered.com`、`steamstatic.com` 等下载相关域名
- `steam.exe`、`steamwebhelper.exe` 等进程
- `steamapps\common` 下游戏路径（进程路径正则）

**分流**

- `RULE-SET,ai` → AI 定向
- `AND,((IN-PORT,7897),(IP-ASN,13335))` → Cloudflare 定向（仅浏览器 7897 入口）
- `RULE-SET,gfw` → 默认代理
- `MATCH` → 未命中流量代理

规则集来自 [DustinWin/ruleset_geodata](https://github.com/DustinWin/ruleset_geodata) 的 `ai.mrs` / `gfw.mrs`。

### 端口约定（ZeroOmega 联动）

| 端口 | 监听 | 用途 |
|------|------|------|
| **7897** | 由客户端主配置决定 | 浏览器常规代理入口（脚本里按此端口识别 CF） |
| **7898** | `127.0.0.1` mixed | 强制代理 → `浏览器转发代理` |
| **7899** | `127.0.0.1` mixed | 强制直连 → `DIRECT` |

脚本会保留原有 `listeners`，并重建名为 `zeroomega-proxy` / `zeroomega-direct` 的两个入口。

> 若你的主 HTTP/SOCKS 不是 7897，请改脚本中 `IN-PORT,7897` 与 ZeroOmega 里「代理判断」配置，保持一致。

### 使用方法

#### 1. Clash / Mihomo 客户端

适用于支持 **全局扩展脚本** 的 Mihomo 系客户端（如 Clash Verge Rev、Mihomo Party 等，名称因客户端而异）。

1. 打开订阅 / 配置的 **全局扩展脚本**（或 Script Override）
2. 将 [`clash/clash全局扩展脚本-v1.1.txt`](clash/clash全局扩展脚本-v1.1.txt) 全文粘贴进去并保存
3. 确认订阅能正常更新，规则集可下载（需能访问 GitHub 或已有代理）
4. 在策略组里按需选择节点（AI 可单独指定美国等）

Windows 下脚本会设置 `find-process-mode: always`，以便进程规则生效。

#### 2. 自定义前置 / 后置 / 删除规则

在全局扩展**覆写配置**（YAML）中可写中文字段，脚本会读取后从最终配置里删掉，避免传给内核：

```yaml
profile:
  store-selected: true

前置规则:
  - DOMAIN-SUFFIX,example.com,DIRECT

后置规则:
  - DOMAIN-SUFFIX,example.com,DIRECT

删除规则:
  - DOMAIN-SUFFIX,example.com,DIRECT
```

完整示例见 [`clash/clash全局扩展覆写配置示例.txt`](clash/clash全局扩展覆写配置示例.txt)。

**规则合并顺序：**

```text
前置规则 → 内置规则（可被「删除规则」精确删掉） → 后置规则 → MATCH
```

「删除规则」按**完整规则字符串**精确匹配删除内置规则。

#### 3. ZeroOmega

1. 安装 [ZeroOmega](https://github.com/zero-peak/ZeroOmega)（SwitchyOmega 继任）
2. 导入 [`clash/ZeroOmegaOptions-v1.bak`](clash/ZeroOmegaOptions-v1.bak)
3. 按本机实际端口核对：
   - **代理判断** → `127.0.0.1:7897`（或你的主端口）
   - **强制代理** → `127.0.0.1:7898`
   - **强制直连** → `127.0.0.1:7899`

匹配不到、又打不开的网页：在扩展里切到「强制代理」即可，不必改 Clash 规则。

### 注意事项

- 本脚本**不继承**机场的 `proxy-groups` 与 `rule-providers`，只保留节点与你自定义的扩展字段逻辑。
- 健康检查默认使用 `https://www.gstatic.com/generate_204`，间隔约 600 秒。
- 规则集每日更新间隔 `86400`；首次加载需能下载到 `./ruleset/*.mrs`。
- 这是个人日用方案，规则不追求「大而全」；有更好的分流思路欢迎提建议。

### 版本

- 全局扩展脚本：`v1.1`
- ZeroOmega 备份：`v1`

---

## License

个人自用合集，按需自取。使用代理服务请遵守当地法律法规与机场条款。
