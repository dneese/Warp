Как использовать Cloudflare WARP на OpenWrt 

На свой компьютер загрузите соответствующий двоичный выпуск wgcf с Github https://github.com/ViRb3/wgcf .

Вы получите файл wgcf-profile.conf , который вам понадобится для настройки Wireguard на вашем маршрутизаторе OpenWrt. Файл должен выглядеть так:
> [Interface]\
PrivateKey = XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\
Address = 100.16.0.2/32\
Address = fddd:5ca1:ab1e:8daf:209d:9414:d1e0:5d2c/128\
DNS = 1.1.1.1\
MTU = 1280\
[Peer]\
PublicKey = XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\
AllowedIPs = 0.0.0.0/0\
AllowedIPs = ::/0\
Endpoint = engage.cloudflareclient.com:2408
>

Теперь на вашем маршрутизаторе OpenWrt выполните: **opkg update && opkg install wireguard wireguard-tools luci-proto-wireguard**

Отредактируйте файл /etc/config/network и добавьте следующие строки. Убедитесь, что личные_ключи {PRIVATEKEY} {PUBLICKEY}. совпадают с имеющимся у вас файлом wgcf-profile.conf :

затем выполните 

opkg update
opkg install luci-proto-wireguard
# network backup
cp /etc/config/network /etc/config/network.cloudflare.bak
# script
NET_IF="lan"
. /lib/functions/network.sh
network_flush_cache
network_get_ipaddr NET_ADDR "${NET_IF}"
# delete
uci -q delete dhcp.lan.dhcp_option
uci -q delete dhcp.lan.dns
# interface\
uci set network.Cloudflare=interface
uci set network.Cloudflare.proto='wireguard'
uci set network.Cloudflare.private_key=${PRIVATEKEY}
uci set network.Cloudflare.mtu='1280'
uci add_list network.Cloudflare.addresses='172.16.0.2/32' довільний
uci add_list network.Cloudflare.addresses='2606:4700:110:876a:4734:8b50:27b8:203b/128' довільний
uci add_list network.Cloudflare.dns='1.1.1.1'
uci add_list network.Cloudflare.dns='1.0.0.1'
uci add_list network.Cloudflare.dns='2606:4700:4700::1111'
uci add_list network.Cloudflare.dns='2606:4700:4700::1001'
# 
uci set network.wireguard_Cloudflare=wireguard_Cloudflare
uci set network.wireguard_Cloudflare.public_key=${PUBLICKEY}
uci set network.wireguard_Cloudflare.endpoint_host='engage.cloudflareclient.com'
uci set network.wireguard_Cloudflare.endpoint_port='2408'
uci set network.wireguard_Cloudflare.route_allowed_ips='1'
uci add_list network.wireguard_Cloudflare.allowed_ips='0.0.0.0/0'
uci add_list network.wireguard_Cloudflare.allowed_ips='::/0'
#
uci set network.route_wireguard=route
uci set network.route_wireguard.interface='Cloudflare'
uci set network.route_wireguard.target='0.0.0.0/0'
uci set network.route_wireguard.gateway=${NET_ADDR}
uci set network.route_wireguard.metric='1024'
# FW
ZOON_NO='1'
uci add_list firewall.@zone[${ZOON_NO}].network='Cloudflare'
# set 
uci commit
# restart
/etc/init.d/network restart
