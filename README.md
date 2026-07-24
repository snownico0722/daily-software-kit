# daily-software-kit · 日用合集

个人日用软件配置搜集库，按主题分目录。

## 目录

```text
clash/   Clash · Mihomo 全局扩展 + ZeroOmega
```

---

## clash/

用全局扩展脚本覆写机场规则：AI 单独走代理，Steam 社区代理、下载/游戏直连，GFW 走主代理，其余默认直连。浏览器补漏用 ZeroOmega（7898 强制代理 / 7899 强制直连）。

| 文件 | 说明 |
|------|------|
| `clash全局扩展脚本-v1.1.txt` | 扩展脚本 |
| `clash全局扩展覆写配置示例.txt` | 前置 / 后置 / 删除规则示例 |
| `clash脚本说明.txt` | 简短备注 |
| `ZeroOmegaOptions-v1.bak` | ZeroOmega 备份 |

### 用法

1. 客户端打开「全局扩展脚本」，粘贴 `clash全局扩展脚本-v1.1.txt`
2. 需要时按示例写前置 / 后置 / 删除规则
3. ZeroOmega 导入 `.bak`；端口：`7897` 常规 · `7898` 强代 · `7899` 强直

### 策略组

`默认代理服务` · `AI定向代理` · `Cloudflare定向代理` · `Steam社区代理` · `浏览器转发代理` · `最快节点` · `故障转移` · `未命中流量代理`（MATCH 默认直连）

### 规则顺序

```text
前置规则 → 内置规则 → 后置规则 → MATCH
```

规则集：`ai.mrs` / `gfw.mrs`（[DustinWin/ruleset_geodata](https://github.com/DustinWin/ruleset_geodata)）

> 脚本不继承机场策略组与 rule-providers。主端口若非 7897，同步改脚本 `IN-PORT` 与 ZeroOmega。
