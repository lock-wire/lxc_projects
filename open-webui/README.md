# Project 2: Ollama with GPU support

Create ollama lxc
```
$ sudo lxc-create --name openwebui --template download -- --dist ubuntu --release noble --arch amd64 --variant cloud
```
```
$ sudo lxc-info openwebui
```
```
lxc-ls -f
```
If no dhcp from cloud-init
```
# lxc-attach ollama
# cloud-init clean
# cat <<EOF>/etc/netplan/50_cloud_init.yaml
#cloud-config
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
EOF
# reboot
```

```
lxc-ls -f
```

## Install Open-WebUI from Docker
```
apt install -y ca-certificates curl 
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-releasesudo
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

docker pull ghcr.io/open-webui/open-webui:main
docker run -d -p 3000:8080 -e OLLAMA_BASE_URL=<ollama_ip>:11434 -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```

# --- Work in progresss---
# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update


## Install Open-WebUI
Referenced proxmox helper script (https://github.com/community-scripts/ProxmoxVE/blob/main/install/openwebui-install.sh)
I did this manually by command. Might replicate the process in a script for my needs later.

msg_info "Installing Dependencies"
```
apt install -y git ffmpeg
```
msg_ok "Installed Dependencies"

msg_info "Setup Python3"
```
apt install -y --no-install-recommends python3 python3-pip python3-pipx
```
msg_ok "Setup Python3"

msg_info "Setup Node.js"
```
sudo apt install -y ca-certificates curl gnupg
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_22.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list
apt update
sudo apt install -y nodejs
```
msg_ok "Setup Node.js"

msg_info "Installing Open WebUI (Patience)"
```
git clone https://github.com/open-webui/open-webui.git /opt/open-webui
cd /opt/open-webui/backend
pipx ensurepath
pipx install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu --include-deps
pipx install -r requirements.txt -U
cd /opt/open-webui
cp .env.example .env
cat <<EOF >/opt/open-webui/.env
ENV=prod
ENABLE_OLLAMA_API=false
OLLAMA_BASE_URL=http://10.0.3.10:11434
EOF
npm install --force --legacy-peer-deps
export NODE_OPTIONS="--max-old-space-size=3584"
npm run build
```
msg_ok "Installed Open WebUI"

msg_info "Creating Service"
```
cat <<EOF >/etc/systemd/system/open-webui.service
[Unit]
Description=Open WebUI Service
After=network.target

[Service]
Type=exec
WorkingDirectory=/opt/open-webui
EnvironmentFile=/opt/open-webui/.env
ExecStart=/opt/open-webui/backend/start.sh

[Install]
WantedBy=multi-user.target
EOF
systemctl enable -q --now open-webui
msg_ok "Created Service"

motd_ssh
customize
```
msg_info "Cleaning up"
```
$STD apt-get -y autoremove
$STD apt-get -y autoclean
```
msg_ok "Cleaned"
