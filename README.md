# Grafana + Prometheus + Node Exporter Monitoring Setup

This guide provides step-by-step instructions to install **Grafana**, **Prometheus**, and **Node Exporter** on CentOS 9 for monitoring system performance (disk, memory, load average, etc.). The guide assume selinux is not set to **enforcing**.

```
setenforce 0
make sure that in /etc/selinux/config SELINUX is set to permissive as shown below

vim /etc/selinux/config
SELINUX=permissive
```
---
##  Install Grafana

```bash
cat <<EOF | tee /etc/yum.repos.d/grafana.repo
[grafana]
name=Grafana
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
EOF

dnf install -y grafana

systemctl enable --now grafana-server
systemctl status grafana

firewall-cmd --permanent --add-port=3000/tcp
firewall-cmd --reload

``` 
On browser 
http://192.168.108.19:3000/
## 🚀 Install Node Exporter
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz

tar -xvf node_exporter-1.8.2.linux-amd64.tar.gz
mv node_exporter-1.8.2.linux-amd64/node_exporter /usr/local/bin/node_exporter

useradd --no-create-home --shell /bin/false nodeusr
chown nodeusr:nodeusr /usr/local/bin/node_exporter
```
Create systemd service
vim /etc/systemd/system/node_exporter.service
```
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=nodeusr
Group=nodeusr
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```
Reload the daemon and start node_exporter service
```
systemctl daemon-reload
systemctl start node_exporter
systemctl enable node_exporter
systemctl status node_exporter
firewall-cmd --permanent --add-port=9090/tcp
firewall-cmd --permanent --add-port=9100/tcp
firewall-cmd --reload
```
On browser
http://192.168.108.19:9100/metrics

