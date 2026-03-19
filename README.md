# Surfshark mit WIREGUARD
# https://my.surfshark.com/vpn/manual-setup/main/wireguard
sudo apt install wireguard wireguard-tools
# Download conf file from website to /etc/wiregard and disable DNS
# DNS = 162.252.172.57, 149.154.159.92
sudo sed -i 's/^DNS/#DNS/' /etc/wireguard/*.conf # ohne DNS
sudo sed -i 's/^DNS = .*/DNS = 1.1.1.1, 8.8.8.8/' /etc/wireguard/*.conf # mit generik DNS
# Add private Key to the config
sudo sed -i 's|<insert_your_private_key_here>|##################|' /etc/wireguard/*.conf
# show all configs
ls /etc/wireguard/*.conf | xargs -n 1 basename -s .conf | tr '\n' ' '; echo
ar-bua at-vie ch-zur co-bog de-ber de-fra ec-uio es-mad fr-par ie-dub it-mil jp-tok kr-seo mx-qro pa-pac pe-lim pt-lis py-asu se-sto tr-ist tw-tai us-bos us-chi us-dal us-mia us-nyc us-slc ve-car
# start - status - stop
sudo wg-quick up ch-zur
sudo wg
sudo ip a show ch-zur
sudo wg-quick down ch-zur
# debugg
journalctl -xe
curl ifconfig.me; echo
ip a | sed -n '/scope global/ { /-/ s/.* //p }'
https://browserleaks.com/ip
