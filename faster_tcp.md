Use google's TCP BBR, it may increase network speed by fixing some TCP's limitations

```bash
sudo tee /etc/sysctl.d/10-bbr-networking.conf <<EOF
# Make networking faster
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr
EOF

# apply changes
sudo sysctl --system
```

To check (should show **bbr**)
```bash
sysctl net.ipv4.tcp_congestion_control
```
