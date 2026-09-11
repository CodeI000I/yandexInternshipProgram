# 1.Несколько маршрутов
У нас есть лог из консоли:
```console
# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever

2: eth-x: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UNKNOWN group default qlen 1000
    link/ether aa:bb:cc:dd:ee:ff brd ff:ff:ff:ff:ff:ff
    inet 192.168.32.51/24 brd 192.168.32.255 scope global eth-x
       valid_lft forever preferred_lft forever
    inet 192.168.32.7/27 brd 192.168.32.94 scope global eth-x:1
       valid_lft forever preferred_lft forever
    inet 192.168.32.70/27 brd 192.168.32.94 scope global eth-x:2
       valid_lft forever preferred_lft forever

3: eth-i2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 11:22:33:44:55:66 brd ff:ff:ff:ff:ff:ff
    inet 192.168.41.52/20 brd 192.168.47.255 scope global eth-i2
       valid_lft forever preferred_lft forever
    inet 192.168.49.94/20 brd 192.168.55.255 scope global eth-i2:1
       valid_lft forever preferred_lft forever

4: eth-i1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 77:88:99:aa:bb:cc brd ff:ff:ff:ff:ff:ff
    inet 192.168.83.52/20 brd 192.168.95.255 scope global eth-i1
       valid_lft forever preferred_lft forever
    inet 145.172.10.10/20 brd 145.172.11.255 scope global eth-i1:1
       valid_lft forever preferred_lft forever

5: eth-i3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1300 qdisc mq state UP group default qlen 1000
    link/ether 11:22:33:44:55:66 brd ff:ff:ff:ff:ff:ff
    inet 192.168.61.145/19 brd 192.168.71.255 scope global eth-i3
       valid_lft forever preferred_lft forever

# ip route
default via 192.168.32.2 dev eth-x proto static
192.168.12.0/23     dev eth-x   proto kernel scope link src 192.168.32.51
192.168.40.0/20     dev eth-i2  proto kernel scope link src 192.168.41.52
192.168.0.0/16      dev eth-x   proto kernel scope link src 192.168.32.7
192.168.32.0/24     dev eth-i2  proto kernel scope link src 192.168.32.178
192.168.32.65/27    dev eth-x   proto kernel scope link src 192.168.32.70
192.168.48.0/20     dev eth-i2  proto kernel scope link src 192.168.49.94
192.168.80.0/20     dev eth-i1  proto kernel scope link src 192.168.83.52
192.168.51.0/19     dev eth-i3  proto kernel scope link src 192.168.61.145

# iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
DROP       icmp --  192.168.32.7         192.168.48.0/20        icmp echo-request echo-reply
ACCEPT     all  --  192.168.32.0/24      anywhere               any

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
REJECT     udp  --  192.168.40.0/24      anywhere

Chain OUTPUT (policy DROP)
target     prot opt source               destination
DROP       icmp --  anywhere             192.168.48.0/20      icmp echo-request
ACCEPT     icmp --  anywhere             192.168.32.0/16      icmp echo-request
REJECT     all  --  192.168.28.0/23      192.168.40.0/24      ADDRTYPE match dst-type BROADCAST
```

Администратор набирает команду ping 192.168.65.255. Как полетят пакеты?

## Варианты ответа:
- Никак, потому что это адрес сети, а не адрес узла
- Через интерфейс еth-i1 напрямую
- Через интерфейс еth-х и через 192.168.32.7 но входящий пакет дропнется
- Через интерфейс еth-i3 и 192.168.61.145
- Через интерфейс еth-х и через 192.168.32.7
- Через интерфейс еth-х напрямую
- Никак, потому что заблокированы широковещательные адреса
- Через интерфейс еth-i2 напрямую