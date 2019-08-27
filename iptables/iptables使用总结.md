# iptables/netfilter架构

# 概念

## chain(链)

就是关卡, 链上可以配置规则
## rules(规则)

规则包括两部分:
- 匹配条件
    - 基本匹配条件
        - 源IP, 目的IP
        - 协议
        - 源网络接口, 目的网络接口
    - 扩展匹配条件(扩展模块: -m)
        - tcp: dport, sport, tcp-flags
        - udp
        - iprange
        - comment
        - conntrack
        - string
        - time
- 处理目标(-j target)
    - 基本动作: accept, drop, reject, SNAT, DNAT, redirect, log
    - 自定义链

### target

```
iptable -t tableName -I/A chainName <匹配条件> -j <target> [option]
```

target分为两种:

- 动作(动作也有选项)
    - 基础动作: ACCEPT, DROP
    - 扩展动作: REJECT, DNAT, SNAT等等
- 自定义链

# 使用总结

- 规则顺序很重要
    > 如果报文被前面的规则匹配上, 将会触发规则对应的动作, 将不会继续匹配后序规则. (前面的规则动作为LOG时除外)
- 当规则中存在多个匹配条件时, 条件默认之间存在`与`关系
- 应该讲更容易匹配到的规则放到最前面
- 在配置iptables白名单时, 要将链的默认规则的动作设置为ACCEPT, 通过在链的最后增加一条REJECT规则.