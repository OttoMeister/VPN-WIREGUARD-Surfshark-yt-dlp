# Surfshark mit WIREGUARD
## https://www.surfshark.com
```shell
sudo apt install wireguard wireguard-tools
```
## Download conf file from website to /etc/wiregard and disable DNS
## https://my.surfshark.com/vpn/manual-setup/main/wireguard
```shell
sudo sed -i 's/^DNS/#DNS/' /etc/wireguard/*.conf # ohne DNS
sudo sed -i 's/^DNS = .*/DNS = 1.1.1.1, 8.8.8.8/' /etc/wireguard/*.conf # mit generik DNS
```
## Add private Key to the config
```shell
sudo sed -i 's|<insert_your_private_key_here>|##################|' /etc/wireguard/*.conf
```
## Show all configs
```shell
ls /etc/wireguard/*.conf | xargs -n 1 basename -s .conf | tr '\n' ' '; echo
```
## Start - status - stop
```shell
sudo wg-quick up ch-zur
sudo wg
sudo ip a show ch-zur
sudo wg-quick down ch-zur
```
## debugg
```shell
journalctl -xe
curl ifconfig.me; echo
https://browserleaks.com/ip
```
