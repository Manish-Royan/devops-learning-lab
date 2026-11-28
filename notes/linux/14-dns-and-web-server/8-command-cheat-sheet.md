# 8. Command Cheat Sheet

All commands used in this chapter, grouped by task. Commands marked **(risky)** are explained, with a safer option, in [7-cleanup-and-safety.md](7-cleanup-and-safety.md).

## Packages

```bash
rpm -q httpd                          # exact installed version of a package, or "not installed"
dnf list --installed httpd            # same check through dnf
rpm -qa | grep bind                   # list installed packages containing "bind" (also matches rpcbind)
rpm -q bind bind-utils                # exact check for the two BIND packages
sudo dnf install -y httpd             # install Apache
sudo dnf install -y bind bind-utils   # install BIND and the lookup tools (dig, nslookup)
```

## Services

```bash
systemctl status httpd                # state, logs, enabled or disabled
systemctl is-active httpd             # active or inactive
sudo systemctl start httpd            # start now
sudo systemctl stop nginx             # stop now
sudo systemctl enable httpd           # start at boot
sudo systemctl disable nginx          # (risky) do not start at boot
sudo systemctl enable --now named     # enable and start in one step
sudo systemctl restart named          # reload the configuration and zone files
```

## Ports and addresses

```bash
sudo ss -tulpn 'sport = :80'          # which process holds port 80
sudo ss -tulpn | grep :80             # shorter form of the same check
sudo fuser -v 80/tcp                  # alternative if ss is unavailable
ip a                                  # IP addresses of all interfaces
ifconfig                              # same information in the older format
```

## Web server files and tests

```bash
cat /var/www/html/index.html          # the page Apache serves
echo "<h1>Hello</h1>" | sudo tee /var/www/html/index.html
curl http://localhost                 # request the page from the machine itself
curl -I http://192.168.110.86         # headers only, from the LAN address
```

## Firewall (`firewalld`)

```bash
sudo firewall-cmd --add-service=http --permanent       # allow port 80 (risky on a shared network)
sudo firewall-cmd --add-service=dns --permanent        # allow port 53 (risky on a shared network)
sudo firewall-cmd --reload                             # apply saved rules
sudo firewall-cmd --list-services                      # show allowed services
sudo firewall-cmd --remove-service=http --permanent    # close port 80 again
sudo firewall-cmd --remove-service=dns --permanent     # close port 53 again
```

## BIND configuration and validation

```bash
sudo cp /etc/named.conf /etc/named.conf.bak            # backup before editing
sudo nano /etc/named.conf                              # edit (Ctrl+O, Enter, Ctrl+X)
sudo named-checkconf /etc/named.conf                   # syntax check of the main config
sudo chgrp named /var/named/manishdevops.local.forwardzone
sudo restorecon -v /var/named/manishdevops.local.forwardzone
sudo named-checkzone manishdevops.local /var/named/manishdevops.local.forwardzone
```

## DNS tests

```bash
dig @192.168.110.86 www.manishdevops.local +short                  # ask the BIND server directly, short answer
nslookup www.manishdevops.local 192.168.110.86                     # same, with nslookup
nslookup www.manishdevops.local                                    # uses the default resolver
curl -s --resolve www.manishdevops.local:80:192.168.110.86 http://www.manishdevops.local | head -n 10
getent hosts www.manishdevops.local                                # how applications resolve the name
grep '^hosts:' /etc/nsswitch.conf                                  # order of lookup methods
```

## Resolver settings

```bash
ls -la /etc/resolv.conf                                            # symlink or plain file?
sudo cp -L /etc/resolv.conf ~/resolv.conf.backup                   # backup of the real content
sudo rm -f /etc/resolv.conf                                        # (risky) remove the link
nmcli -t -f NAME,DEVICE connection show --active                   # active connection name
sudo nmcli connection modify "<wifi-connection-name>" ipv4.dns "192.168.110.86" ipv4.ignore-auto-dns yes
sudo nmcli connection up "<wifi-connection-name>"
resolvectl status wlo1                                             # DNS servers used by the Wi-Fi link
```

## Revert

```bash
sudo ln -sf ../run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
sudo systemctl restart systemd-resolved
sudo nmcli connection modify "<wifi-connection-name>" ipv4.dns "" ipv4.ignore-auto-dns no ipv6.dns "" ipv6.ignore-auto-dns no ipv6.method auto
sudo systemctl stop named httpd
sudo systemctl disable named httpd
```
