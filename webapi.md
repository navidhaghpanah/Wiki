# WebAPI 文档

最后修改：2023.01.19

路由前缀为 `/mod_mu/`， 所有请求必须带 key 参数（即 config 中的 muKey）并且面板中启用了 WebAPI 功能。除非特别说明，所有返回值均使用 json 编码。

## User

路由 | 方式 | 参数 | 返回值 |描述
-----|------|-----|-------|----
`/users` | GET | node_id | 见下表 | 获取当前请求节点（根据 node_id 参数判断）可连接的用户

节点类型 | 下发的数据
--------|--------
Shadowsocks | method, node_speedlimit, node_iplimit, port, passwd, node_connector, alive_ip
Vmess/Vless  | node_speedlimit, node_iplimit, node_connector, uuid, alive_ip
Trojan | node_speedlimit, node_iplimit, node_connector, uuid, alive_ip

---
路由 | 方式 | 参数 | 描述
-----|------|-----|-------
`/users/traffic` | POST | node_id, u, d, user_id | 上报用户的流量使用情况

---
路由 | 方式 | 参数 | 描述
-----|------|-----|-------
`/users/aliveip` | POST | node_id, ip, user_id | 上报用户的当前在线IP

---
路由 | 方式 | 参数 | 描述
-----|------|-----|-------
`/users/detectlog` | POST | node_id, list_id, user_id | 上报用户碰撞的审计规则id

## Node

路由 | 方式 | 参数 | 返回值 |描述
-----|------|-----|-------|----
`/nodes/{id}/info` | GET | node_id | node_group, node_class, node_speedlimit, traffic_rate, mu_only, sort, server, disconnect_time, type | 获取当前请求节点的节点设置

## Func

路由 | 方式 | 参数 | 返回值 |描述
-----|------|-----|-------|----
`/func/detect_rules` | GET | N/A | rules | 获取当前的审计规则

---
路由 | 方式 | 参数 | 返回值 |描述
-----|------|-----|-------|----
`/func/ping` | GET | N/A | N/A | Ping? Pong!

## Media

路由 | 方式 | 参数 | 返回值 |描述
-----|------|-----|-------|----
`/media/save_report` | POST | node_id, content | N/A | 上报节点流媒体解锁报告
