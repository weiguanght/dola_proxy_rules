# Dola Clash 远程规则

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
