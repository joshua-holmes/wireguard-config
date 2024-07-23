# WireGuard Config

This repo is to house a default layout of a server-client WireGuard setup. The purpose is really for my own note taking, but if you stumbled upon this for help, here is a quick summary of how I would set this up on my own machines.

1. Set up a server instance with your preferred provider (mine is AWS) so that you can SSH into it. When I tested 500Mbps down and up, I was using about 40% of my CPU (my EC2 instance had 2 vCPUs) and only around 250MB of total RAM used, including the Debian OS that was running on it. RAM is not a concern, this is mostly a CPU-bound task.
2. Allow ipv4 packet forwarding by adding `net.ipv4.ip_forward = 1` to `/etc/sysctl.conf`, which will persist after reboot. Then, either reboot, or run `sudo sysctl -w net.ipv4.ip_forward=1`. To verify that it is enabled, run `sudo sysctl -a | grep 'net.ipv4.ip_forward ='`. It should be set to 1.
3. Setup routing on the server. Run these commands, one line at a time. Obviously don't run the comments, it won't do anything. `iptables` may not be installed on every system by default, but I will let you consult the googs for that if it's not installed. Try running `sudo iptables` to see if it's installed. `sudo` is necessary here.
```
# Flush existing routing configuration for a clean slate
sudo iptables -F
sudo iptables -X
sudo iptables -t nat -F
sudo iptables -t nat -X
sudo iptables -t mangle -F
sudo iptables -t mangle -X
sudo iptables -P INPUT ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -P FORWARD ACCEPT

# Allow port forwarding
sudo iptables -A FORWARD -j ACCEPT

# Allow NAT to masquerade all traffic as the server's public IP
sudo iptables -t nat -A POSTROUTING -j MASQUERADE

# iptables is not persistent by default, so this package will handle that.
sudo apt install iptables-persistent

# If you install this before running the previous commands, or make changes
# to iptables after installing, you must use the `iptables-save` command to
# persist those new changes. Look up documentation on how to do that. If you
# follow the order I presented, installing the package will setup initial
# config files so you don't need to go through the manual save steps.
```
4. Install wireguard, it will vary depending on the OS, so I'll let you look it up. The common commands it comes with are `wg`, which is the main WireGuard command, and `wg-quick` which allows you to spin up configured instances quickly after you have already set up the config files.
5. Each file would be placed on each machine like so: `/etc/wireguard/wg0.conf`.
6. Customize those config files to your needs. Use the documentation if you need help
7. Use `wg-quick up wg0` on each machine to start the instance. A systemd service is automatically set up if you want to automate. It works like this:
```
# systemctl enable wg-quick@wg0.service
```
7. If you want more configs, they can be setup like `/etc/wireguard/wg1.conf` and then you just enable them with the new name, which is `wg1`.

This is not supposed to be a tutorial, but in short, the "Instance" on each machine refers to info about itself and it's identity. The "Peer" on the server side refers to a specific client. Each client you want to connect to the VPN will need a new "Peer" section in the server-side config. The "Peer" on the client side refers to the server and it's info. See [this tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-wireguard-on-ubuntu-22-04#step-4-%E2%80%94-adjusting-the-wireguard-server-39-s-network-configuration) if you want a really solid walk-through. However, for server configuration, I found [this article](https://www.linode.com/docs/guides/linux-router-and-ip-forwarding/?tabs=iptables) about configuring a server to function as a router to be more helpful. Good luck.
