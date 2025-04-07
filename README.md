 Node Exporter Automated Setup Script
# Compatible with RHEL 8 / CentOS / Rocky Linux.

# create a file named with "install_node_exporter.sh"
# copy the content below in the file with using " nano or vi " & save and exit.
------------------------------------------------------------------------------------------------------------------------
#!/bin/bash

# Node Exporter Installation Script
# Tested on RHEL 8 (minor adjustments may be needed for Ubuntu)

set -e

echo "🔧 Creating node_exporter user..."
useradd -rs /bin/false node_exporter || echo "User may already exist"

echo "⬇️ Downloading Node Exporter..."
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz -P /tmp/

echo "📦 Extracting package..."
tar -xzf /tmp/node_exporter-1.7.0.linux-amd64.tar.gz -C /tmp

echo "🚚 Moving binary to /usr/local/bin..."
mv /tmp/node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/

echo "🔥 Opening firewall port 9100..."
firewall-cmd --permanent --add-port=9100/tcp
firewall-cmd --reload

echo "📝 Creating systemd service file..."
cat <<EOF > /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF

echo "🔄 Reloading systemd..."
systemctl daemon-reload

echo "🚀 Enabling and starting node_exporter service..."
systemctl enable --now node_exporter

echo "🔐 Setting SELinux context (if enabled)..."
/sbin/restorecon -v /usr/local/bin/node_exporter || echo "SELinux may not be enforced"

echo "✅ Restarting service to apply everything..."
systemctl restart node_exporter

echo "📊 Verifying metrics endpoint..."
curl -s http://localhost:9100/metrics | head -n 10

echo "🎉 Node Exporter installed and running on port 9100!"

------------------------------------------------------------------------------------------------------------------

# execute the commands below :

# run with sudo permissions :

  sudo chmod +x install_node_exporter.sh

  sudo ./install_node_exporter.sh
