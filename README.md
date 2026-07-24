# daily-software-kit · 日用合集

个人日用软件配置搜集库，按主题分目录。

## 目录

```text
clash/   Clash · Mihomo 全局扩展 + ZeroOmega
```

---

## clash/

用全局扩展脚本覆写机场规则：AI 单独走代理，Steam 社区走代理、下载和游戏直连，GFW 域名走主代理，其余默认直连。浏览器里还是打不开的站点，用 ZeroOmega 手动切强制代理。

| 文件 | 说明 |
|------|------|
| `clash全局扩展脚本-v1.1.txt` | 扩展脚本 |
| `clash全局扩展覆写配置示例.txt` | 前置 / 后置 / 删除规则示例 |
| `clash脚本说明.txt` | 简短备注 |
| `ZeroOmegaOptions-v1.bak` | ZeroOmega 配置备份 |

### 用法

1. 在客户端打开「全局扩展脚本」，把 `clash全局扩展脚本-v1.1.txt` 全文贴进去。
2. 需要自定义时，按示例写前置、后置、删除规则。
3. ZeroOmega 导入 `.bak`。端口约定：
   - **7897**：浏览器常规代理入口
   - **7898**：强制代理
   - **7899**：强制直连

### 策略组

`默认代理服务` · `AI定向代理` · `Cloudflare定向代理` · `Steam社区代理` · `浏览器转发代理` · `最快节点` · `故障转移` · `未命中流量代理`（最终兜底，默认直连）

### 规则顺序

```text
前置规则 → 内置规则 → 后置规则 → MATCH
```

规则集用 `ai.mrs` / `gfw.mrs`（[DustinWin/ruleset_geodata](https://github.com/DustinWin/ruleset_geodata)）。

脚本不会继承机场原来的策略组和 rule-providers。如果你的主端口不是 7897，记得同时改脚本里的 `IN-PORT` 和 ZeroOmega 配置。
