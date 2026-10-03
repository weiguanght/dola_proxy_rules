# Dola 分流规则（Clash / Loon）

保留原有 Clash YAML，另提供 Loon 插件及远程规则列表。

## Clash

远程规则文件：

```text
https://raw.githubusercontent.com/weiguanght/dola_proxy_rules/main/dola.yaml
```

在 Clash Verge 的配置中引用：

```yaml
rule-providers:
  dola:
    type: http
    behavior: classical
    format: yaml
    url: https://raw.githubusercontent.com/weiguanght/dola_proxy_rules/main/dola.yaml
    path: ./ruleset/dola.yaml
    interval: 86400

rules:
  - RULE-SET,dola
```

规则按文件顺序匹配。`wss-normal-i18n.dola.com` 第一条是 `REJECT`，因此后面的同域名代理规则不会生效；该域名最终会被屏蔽。`🚀 节点选择` 必须是本地配置中已有的代理组名称。

## Loon

以下两种方式任选其一，不要重复添加。Loon 版本保留原 YAML 的有效匹配结果：1 个域名拦截、5 个域名走代理；省略已被前置拦截覆盖的同域名代理规则。所有规则均为精确域名匹配，不扩大到整个域名后缀。

### 方式一：远程规则订阅（推荐）

分别添加两个远程规则订阅：

| 文件 | 策略 | 用途 |
| --- | --- | --- |
| [dola_reject.list](https://raw.githubusercontent.com/weiguanght/dola_proxy_rules/main/dola_reject.list) | `REJECT` | 拦截 `wss-normal-i18n.dola.com` |
| [dola.list](https://raw.githubusercontent.com/weiguanght/dola_proxy_rules/main/dola.list) | `Available` | 其余 5 个域名走代理 |

将以下两行合并到现有配置的 `[Remote Rule]` 段；请确保 Loon 中已存在名为 `Available` 的策略组：

```ini
[Remote Rule]
https://raw.githubusercontent.com/weiguanght/dola_proxy_rules/main/dola_reject.list,policy=REJECT,tag=Dola-Reject,enabled=true
https://raw.githubusercontent.com/weiguanght/dola_proxy_rules/main/dola.list,policy=Available,tag=Dola-Proxy,enabled=true
```

`.list` 文件不内置策略，策略在订阅时指定。仅添加 `dola.list` 不会拦截 WebSocket 域名，必须同时添加拦截列表才能保持原规则行为。

如果已经添加过远程规则订阅，请将 `Dola-Proxy` 的策略改为 `Available`；仅刷新 `.list` 不会修改本地订阅的策略。采用此方式无需启用 Dola 插件。

### 方式二：插件（可选，单链接）

在 Loon 的插件管理中通过 URL 添加：

```text
https://raw.githubusercontent.com/weiguanght/dola_proxy_rules/main/dola.plugin
```

启用插件，并在插件的代理策略设置中选择 `Available` 策略组。插件中的 `PROXY` 会映射到 `Available`，`REJECT` 保持拦截。不需要脚本、重写或 MITM 证书。

也可以将下面一行合并到现有配置的 `[Plugin]` 段；同样需要已存在 `Available` 策略组：

```ini
[Plugin]
https://raw.githubusercontent.com/weiguanght/dola_proxy_rules/main/dola.plugin,tag=Dola,policy=Available,enabled=true
```

### 注意事项

- 以上配置片段不是完整的 Loon 配置，不要直接替换整个配置文件。
- 使用规则分流模式；检查规则排序，避免被更高优先级的同域名规则或宽泛规则提前匹配。
- WebSocket 域名被拦截可能影响实时连接，这是原规则的既有行为，不是 Loon 转换新增的限制。
