# Surfshark with WIREGUARD
## https://www.surfshark.com
```shell
sudo apt install wireguard wireguard-tools
```
## Download conf file from website to /etc/wiregard
## https://my.surfshark.com/vpn/manual-setup/main/wireguard

![Alternativer Text](https://github.com/OttoMeister/VPN-WIREGUARD-Surfshark-yt-dlp/blob/62822c09982620b92e2aeb328bba3f3d2f9a1f62/DownloadConfig.png)

## Change all DNS in config files
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
# YT Downloader einrichten
```shell
# mkdir -p ~/.local/bin && echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc
wget https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp -O ~/.local/bin/yt-dlp
wget https://github.com/denoland/deno/releases/latest/download/deno-x86_64-unknown-linux-gnu.zip -qO- | funzip >~/.local/bin/deno
chmod a+rx ~/.local/bin/yt-dlp ~/.local/bin/deno
yt-dlp --update-to nightly
```


