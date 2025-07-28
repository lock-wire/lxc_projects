# Project 1: Ollama with GPU support

Create ollama lxc
`$sudo lxc-create --name ollama --template download -- --dist ubuntu --release noble --arch amd64 --variant cloud`

`$sudo lxc-info ollama`
`lxc-ls -f`

If no dhcp from cloud-init
`lxc-attach ollama`
`cloud-init clean`
`reboot`
`lxc-ls -f`
## Hardware acceleration for Nvidia GPU
Reference (https://hostbor.com/gpu-passthrough-in-lxc-containers/)

Might need to install nvidia container toolkit
(http://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)


`$sudo vi /var/lib/lxc/ollama/config`
`lxc.cgroup2.devices.allow = c 195:* rwm
lxc.cgroup2.devices.allow = c 506:* rwm
lxc.mount.entry = /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry = /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry = /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry = /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file`

`sudo apt install nvidia-driver-570-open`
`nvidia-smi -L`

Install ollama
`apt install curl`
`curl -fsSL https://ollama.com/install.sh | sh`
