### 背景
- 因业务流量增长，需要进行上联扩容80G-->160G
- 网络拓扑  
![image](https://github.com/XuYBup/Network/assets/111575435/b32ccf57-c67e-45b2-8bcb-316d59034d7c)
- 路由介绍：ASW--CSW通过EBGP三层对接，CSW从ASW学到的BGP路由做PBR/路由策略重定向下一跳到AR； CSW--AR静态路由三层对接

### 问题原因
- PBR/路由策略不能直接定向两个下一跳地址；扩容后需重定向的路由不会ECMP，只跑原有线路

### 解决办法
- 可通过新增一个重定向IP地址，配置静态路由联动bfd echo，将路由牵引到两条线路上
- bfd echo配置
```
bfd number 1 peer-ip X.X.X.X interface XXX source-ip X.X.X.X one-arm-echo
 discriminator local 1

bfd number 2 peer-ip X.X.X.X interface XXX source-ip X.X.X.X one-arm-echo
 discriminator local 2
```
- 静态路由配置，需新增一个重定向IP，eg: 1.1.1.1
```
ip route-static 1.1.1.1 32 上联接口 下一跳 track bfd-session bfd 1 description X
ip route-static 1.1.1.1 32 上联接口 下一跳 track bfd-session bfd 2 description X
```
- 修改PBR/路由策略
```
route-policy X permit node 500
 apply ip-address next-hop 1.1.1.1
 
traffic behavior X
 redirect nexthop 1.1.1.1
```
