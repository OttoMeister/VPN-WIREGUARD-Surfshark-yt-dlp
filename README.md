# Download YouTube with VPN and yt-dlp:

## Routes yt-dlp downloads through an isolated WireGuard network namespace keeping the rest of the system's traffic unaffected.
Usage:  
vpn-yt-dlp rand https://www.youtube.com/watch?v=1nnatyEvxQU #rand is random VPN conection  
vpn-yt-dlp us-slc https://www.youtube.com/watch?v=1nnatyEvxQU  
vpn-yt-dlp rand 1nnatyEvxQU #rand is random VPN conection  
vpn-yt-dlp us-nyc 1nnatyEvxQU  
```shell
#!/usr/bin/env bash
set -euo pipefail
WG_DIR="/etc/wireguard"
NS="vpn-yt-$$" WG_IF="wg-yt-$$"
[[ $EUID -ne 0 ]] && exec sudo --preserve-env=HOME "$0" "$@"
REAL_USER="${SUDO_USER:-$USER}"
REAL_HOME=$(getent passwd "$REAL_USER" | cut -d: -f6)
PATH_YTDLP="$REAL_HOME/.local/bin/yt-dlp"
PATH_DENO="$REAL_HOME/.local/bin/deno"
YTDLP_DEFAULTS=(-f "bestvideo[ext=mp4]+bestaudio[ext=m4a]/best[ext=mp4]/best"
  --merge-output-format mp4 --js-runtimes "deno:$PATH_DENO")
die()   { echo "Error: $*"; exit 1; }
usage() {
  echo "Usage: $(basename "$0") <vpn|rand> <URL|ID> [yt-dlp-options]"
  echo -n "VPNs: "; ls "$WG_DIR"/*.conf | xargs -n1 basename -s .conf | tr '\n' ' '; echo
  exit 1
}
[[ $# -lt 2 ]] && usage
AVAILABLE=($(ls "$WG_DIR"/*.conf | xargs -n1 basename -s .conf))
[[ "$1" == "rand" ]] && VPN="${AVAILABLE[$RANDOM % ${#AVAILABLE[@]}]}" \
  || { [[ -f "$WG_DIR/$1.conf" ]] && VPN="$1" || die "'$1' unknown."; }
shift
URL="$1"; shift
[[ "$URL" != http* ]] && URL="https://www.youtube.com/watch?v=$URL"
trap 'ip netns exec "$NS" ip link delete "$WG_IF" 2>/dev/null||true
      ip netns delete "$NS" 2>/dev/null||true; rm -rf "/etc/netns/$NS"' EXIT INT TERM
S=$(sed 's/#.*//;/^[[:space:]]*$/d' "$WG_DIR/$VPN.conf")
gi() { echo "$S" | grep -m1 "^$1" | cut -d= -f2- | sed 's/^ *//;s/ *$//'; }
gp() { echo "$S" | grep -A5 '^\[Peer\]' | grep -m1 "^$1" | cut -d= -f2- | sed 's/^ *//;s/ *$//'; }
PK=$(gi PrivateKey); ADDR=$(gi Address); DNS=$(gi DNS)
PUBK=$(gp PublicKey); EP=$(gp Endpoint); AIPS=$(gp AllowedIPs); PSK=$(gp PresharedKey||true)
EP_IP=$(getent hosts "${EP%:*}" | awk '{print $1;exit}')
[[ -z "$EP_IP" ]] && die "DNS lookup failed for ${EP%:*}."
echo "[vpn-yt-dlp] VPN=$VPN EP=$EP_IP URL=$URL"
ip netns add "$NS"
ip netns exec "$NS" ip link set lo up
ip link add "$WG_IF" type wireguard
ip link set "$WG_IF" netns "$NS"
ip netns exec "$NS" wg set "$WG_IF" \
  private-key <(echo "$PK") listen-port 0 \
  peer "$PUBK" ${PSK:+preshared-key <(echo "$PSK")} \
  endpoint "$EP_IP:${EP##*:}" allowed-ips "$AIPS" persistent-keepalive 25
echo "$ADDR" | tr ',' '\n' | sed 's/^ *//;s/ *$//' | xargs -r -I{} ip netns exec "$NS" ip addr add {} dev "$WG_IF"
ip netns exec "$NS" ip link set "$WG_IF" up
ip netns exec "$NS" ip route add default dev "$WG_IF"
mkdir -p "/etc/netns/$NS"
echo "nameserver ${DNS%%,*}" > "/etc/netns/$NS/resolv.conf"
echo -n "[vpn-yt-dlp] External IP: "
ip netns exec "$NS" curl -s --max-time 5 https://api.ipify.org || echo "unknown"; echo
ip netns exec "$NS" sudo -u "$REAL_USER" "$PATH_YTDLP" "${YTDLP_DEFAULTS[@]}" "$@" "$URL"
```
Place it in ~/.local/bin/vpn-yt-dlp and change execution:
```shell
chmod +x ~/.local/bin/vpn-yt-dlp
```
# Config Surfshark with WIREGUARD
## https://www.surfshark.com
```shell
sudo apt install wireguard wireguard-tools
```
### Download conf file from website to /etc/wireguard
https://my.surfshark.com/vpn/manual-setup/main/wireguard

![Alternativer Text](https://github.com/OttoMeister/VPN-WIREGUARD-Surfshark-yt-dlp/blob/62822c09982620b92e2aeb328bba3f3d2f9a1f62/DownloadConfig.png)

## Change all DNS in config files
```shell
sed -i 's/^#\?DNS = .*/DNS = 1.1.1.1, 8.8.8.8/' ~/Downloads/*.conf
```
## Add private Key to the config as YOUR_PRIVATE_KEY_HERE
```shell
sed -i 's|<insert_your_private_key_here>|YOUR_PRIVATE_KEY_HERE|' ~/Downloads/*.conf
```
## Move to etc and make root and chmod
```shell
sudo mv ~/Downloads/*.conf /etc/wireguard/
sudo chown root:root /etc/wireguard/*.conf
sudo chmod 600 /etc/wireguard/*.conf
```
## Show all configs
```shell
sudo sh -c "ls /etc/wireguard/*.conf | xargs -n 1 basename -s .conf | tr '\n' ' '" ; echo
```
## Start - status - stop
```shell
sudo wg-quick up ch-zur
sudo wg
sudo ip a show ch-zur
sudo wg-quick down ch-zur
```
## debug
```shell
journalctl -xe
curl ifconfig.me; echo
https://browserleaks.com/ip
```
# Config YT Downloader  
```shell
# mkdir -p ~/.local/bin && echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc
wget https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp -O ~/.local/bin/yt-dlp
wget https://github.com/denoland/deno/releases/latest/download/deno-x86_64-unknown-linux-gnu.zip -qO- | funzip >~/.local/bin/deno
chmod a+rx ~/.local/bin/yt-dlp ~/.local/bin/deno
yt-dlp --update-to nightly
```


