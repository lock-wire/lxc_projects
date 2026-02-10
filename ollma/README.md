# Project 1: Ollama with GPU support

Create ollama lxc
```
$sudo lxc-create --name ollama --template download -- --dist ubuntu --release noble --arch amd64 --variant cloud`
```
```
$sudo lxc-info ollama
```
```
lxc-ls -f
```
If no dhcp from cloud-init
```
#lxc-attach ollama
#cloud-init clean
cat <<EOF>/etc/netplan/50_cloud_init.yaml
#cloud-config
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
EOF
#reboot
```
```
lxc-ls -f
```

Add to container if static ip desired.
```
cat <<EOF>>/etc/netplan/25_static_init.yaml
#cloud-config
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: no
      addresses: [10.0.3.10/24]
      routes:
        - to: default
          via: 10.0.3.1
      nameservers:
        addresses: [localhost]
EOF
```

## Hardware acceleration for Nvidia GPU
Reference (https://hostbor.com/gpu-passthrough-in-lxc-containers/)

Might need to install nvidia container toolkit
(http://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)

Shutdown the container first
```
lxc-stop ollama
```

Edit the container configuration file
```
$sudo vi /var/lib/lxc/ollama/config`
```
Add the following
```
# Container specific configuration
lxc.cgroup2.devices.allow = c 195:* rwm
lxc.cgroup2.devices.allow = c 506:* rwm
lxc.mount.entry = /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry = /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry = /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry = /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file
```

`sudo apt install nvidia-driver-570-open`

`nvidia-smi -L`

Install ollama
Referenced proxmox helper script (https://github.com/community-scripts/ProxmoxVE/blob/main/install/ollama-install.sh)

`apt install curl`

This kept failing to download may try again `curl -fsSL https://ollama.com/install.sh | sh`

Used some code from the proxmox helper script. I did this mannually for now.
```
# RELEASE=$(curl -fsSL https://api.github.com/repos/ollama/ollama/releases/latest | grep "tag_name" | awk -F '"' '{print $4}')
# OLLAMA_INSTALL_DIR="/usr/local/lib/ollama"
# BINDIR="/usr/local/bin"
# mkdir -p $OLLAMA_INSTALL_DIR
# OLLAMA_URL="https://github.com/ollama/ollama/releases/download/${RELEASE}/ollama-linux-amd64.tgz"
# TMP_TAR="/tmp/ollama.tgz"
# curl -fL# -o "$TMP_TAR" "$OLLAMA_URL"
# tar -xzf "$TMP_TAR" -C "$OLLAMA_INSTALL_DIR"
# ln -sf "$OLLAMA_INSTALL_DIR/bin/ollama" "$BINDIR/ollama"
# useradd -r -s /usr/sbin/nologin -U -m -d /usr/share/ollama ollama
# usermod -aG render ollama
# usermod -aG video ollama
# cat <<EOF >/etc/systemd/system/ollama.service
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
Type=exec
ExecStart=/usr/local/bin/ollama serve
Environment=HOME=$HOME
Environment=OLLAMA_HOST=0.0.0.0
Environment=SYCL_CACHE_PERSISTENT=1
Environment=ZES_ENABLE_SYSMAN=1
Environment=OLLAMA_VULKAN=1
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
EOF
```

Check by browsing to (http://<container_ip>:11434)

Choose a model
```
ollama run codellama:13b
```
