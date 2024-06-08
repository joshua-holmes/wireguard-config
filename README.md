# WireGuard Config

This repo is to house a default layout of a server-client WireGuard setup. The purpose is really for my own note taking, but if you stumbled upon this for help, here is a quick summary of how I would set this up on my own machines.

1. Set up a server instance with your preferred provider (mine is AWS) so that you can SSH into it. When I tested 500Mbps down and up, I was using about 40% of my CPU (my EC2 instance had 2 vCPUs) and only around 250MB of total RAM used, including the Debian OS that was running on it. RAM is not a concern, this is mostly a CPU-bound task.
2. Allow ipv4 packet forwarding by adding `net.ipv4.ip_forward = 1` to `/etc/sysctl.conf`, which will persist after reboot. Then, either reboot, or run `sudo sysctl -w net-ipv4.ip_forward=1`. To verify that it is enabled, run `sudo sysctl -a | grep 'net.ipv4.ip_forward ='.` It should be set to 1.
3. Install wireguard, it will vary depending on the OS, so I'll let you look it up. The common commands it comes with are `wg`, which is the main WireGuard command, and `wg-quick` which allows you to spin up configured instances quickly after you have already set up the config files.
4. Each file would be placed on each machine like so: `/etc/wireguard/wg0.conf`.
5. Customize those config files to your needs. Use the documentation if you need help
6. Use `wg-quick up wg0` on each machine to start the instance. A systemd service is automatically set up if you want to automate. It works like this:
```
# systemctl enable wg-quick@wg0.service
```
7. If you want more configs, they can be setup like `/etc/wireguard/wg1.conf` and then you just enable them with the new name, which is `wg1`.

This is not supposed to be a tutorial, but in short, the "Instance" on each machine refers to info about itself and it's identity. The "Peer" on the server side refers to a specific client. Each client you want to connect to the VPN will need a new "Peer" section in the server-side config. The "Peer" on the client side refers to the server and it's info. See [this tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-wireguard-on-ubuntu-22-04#step-4-%E2%80%94-adjusting-the-wireguard-server-39-s-network-configuration) if you want a really solid walk-through. Good luck.
