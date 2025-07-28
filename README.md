# LXC Projects
Personal LXC projects

# Prepare Ubuntu host for LXC

## LXC Network DHCP pool
Edit or create `/etc/lxc/dnsmasq.conf`
```
$sudo vi /etc/lxc/dnsmasq.conf
```
Add the following:
```
# DHCP Range
dhcp-range=lxcbr0,10.0.3.100,10.0.3.254,24h
```

## LXC Network DHCP reservations
Edit or create `/etc/lxc/dnsmasq.conf`
```
$sudo vi /etc/lxc/dnsmasq.conf
```
Add the following:
```
# DHCP Reservations
dhcp-host=<container_name>,10.0.3.10
```

## UFW Prep
Establish route for outbound traffic from lxc bridge
```
sudo ufw route allow out on lxcbr0`
```
Establish route for inbound traffic to lxc bridge
```
sudo ufw route allow in on lxdbr0
sudo ufw allow in on lxdbr0
```

