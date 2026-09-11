# Порядок применения правил

Для отправлленого icmp пакета действия применяются в следующем порядке:
1) Применение правила из iptables (цепочка OUTPUT)
2) Применение правил маршрутизации из ip route
3) отправка на интерфейс (ip addr show)

# Разбор команд
## iptables
```console
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

Представлена таблица Filter c тремя цепочками (chain), в которых принимается первая подходящая политика. Каждая политика состоит из действия (ACEEPT, DROP, REJECT, LOG и т.д) и условий его применения (протокол, источник, направление и т.д.)

Так как мы отправляем пакет (цепочка OUTPUT) и он icmp, то применится либо DROP, либо ACCEPT. 
Праивло DROP применяется для сети 192.168.48.0.20 => 192.168.48.0 - 192.168.63.255, т.е. максимальный адрес сети меньше нашей цели ->
работает правило ACCEPT.
## ip route
```console
default via 192.168.32.2 dev eth-x proto static
192.168.12.0/23     dev eth-x   proto kernel scope link src 192.168.32.51 -> 
192.168.40.0/20     dev eth-i2  proto kernel scope link src 192.168.41.52
192.168.0.0/16      dev eth-x   proto kernel scope link src 192.168.32.7
192.168.32.0/24     dev eth-i2  proto kernel scope link src 192.168.32.178
192.168.32.65/27    dev eth-x   proto kernel scope link src 192.168.32.70
192.168.48.0/20     dev eth-i2  proto kernel scope link src 192.168.49.94
192.168.80.0/20     dev eth-i1  proto kernel scope link src 192.168.83.52
192.168.51.0/19     dev eth-i3  proto kernel scope link src 192.168.61.145
```

В таблице маршрутизации выбирается самый специфический маршрут (то есть с наибольщим совпадающим префиксом).
Сети, которые могут подходить:
1) 192.168.40.0/20 => 192.168.32.0 - 192.168.47.255 - максимальный адрес меньше требуемого
2) 192.168.0.0/16 - точно подходит
3) 192.168.48.0/20 => 192.168.48.0 - 192.168.63.255 - максимальный адрес меньше требуемого
4) 192.168.51.0/19 => 192.168.32.0 - 192.168.63.255 - максимальный адрес меньше требуемого

Значит мы отправялем в сеть 192.168.0.0 с source ip = 192.168.32.7 через интерфейс eth-x, так как не указывали свой специфический source ip.

## ip addr show
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
    inet 192.168.32.51/24 brd 192.168.32.255 scope global eth-x // 192.168.32.0 - 192.168.32.255
       valid_lft forever preferred_lft forever
    inet 192.168.32.7/27 brd 192.168.32.94 scope global eth-x:1 // 192.168.32.0 - 192.168.32.31
       valid_lft forever preferred_lft forever
    inet 192.168.32.70/27 brd 192.168.32.94 scope global eth-x:2 // 192.168.32.64 - 192.168.32.95
       valid_lft forever preferred_lft forever

3: eth-i2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 11:22:33:44:55:66 brd ff:ff:ff:ff:ff:ff
    inet 192.168.41.52/20 brd 192.168.47.255 scope global eth-i2 // 192.168.32.0 - 192.168.47.255
       valid_lft forever preferred_lft forever
    inet 192.168.49.94/20 brd 192.168.55.255 scope global eth-i2:1 // 192.168.48.0 - 192.168.63.255
       valid_lft forever preferred_lft forever

4: eth-i1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 77:88:99:aa:bb:cc brd ff:ff:ff:ff:ff:ff
    inet 192.168.83.52/20 brd 192.168.95.255 scope global eth-i1 // 192.168.80.0 - 192.168.95.255
       valid_lft forever preferred_lft forever
    inet 145.172.10.10/20 brd 145.172.11.255 scope global eth-i1:1 // 145.172.0.0 - 145.172.15.255
       valid_lft forever preferred_lft forever

5: eth-i3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1300 qdisc mq state UP group default qlen 1000
    link/ether 11:22:33:44:55:66 brd ff:ff:ff:ff:ff:ff
    inet 192.168.61.145/19 brd 192.168.71.255 scope global eth-i3 // 192.168.32.0 - 192.168.63.255
       valid_lft forever preferred_lft forever
```
На самом деле, разбор этой команды не имеет смысла, так как ранее уже был указан source ip и интерфейс.
Псевдонимы (eth-x:1, eth-x:2 и т.д.) не учавствуют в маршрутизации.

# Ответ: Через интерфейс еth-х и через 192.168.32.7