


## ip 

## ip route相关

查看路由，以及相关的目标IP从哪出去，
```bash
ip route show
# 或
ip route get <dst-ip>
```

查看目标IP从哪出去
```bash
ip route get 192.168.164.138
192.168.164.138 via 10.10.10.12 dev eth1 src 10.10.10.11 uid 0 
    cache 
```