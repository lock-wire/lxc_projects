# LXC Projects
Personal LXC projects

# Prepare Ubuntu host for LXC

## UFW Prep
Establish route for outbound traffic from lxc bridge
`sudo ufw route allow out on lxcbr0`

Establish route for inbound traffic to lxc bridge
`sudo ufw route allow in on lxdbr0`
`sudo ufw allow in on lxdbr0`

## Create DHCP Reservation



