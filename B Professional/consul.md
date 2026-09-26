
```
wget https://releases.hashicorp.com/consul/1.15.0/consul_1.15.0_linux_amd64.zip
unzip consul_1.15.0_linux_amd64.zip
ls
mv consul /usr/local/bin/
consul --version
useradd --system --create-home --shell /bin/false consul
mkdir -p /etc/consul.d
mkdir -p /opt/consul
chown -R consul:consul /usr/local/bin/consul
chown -R consul:consul /etc/consul.d
chown -R consul:consul /opt/consul
chmod 750 /etc/consul.d
nano /etc/consul.d/consul.hcl
nano /etc/systemd/system/consul.service
systemctl daemon-reload
systemctl start consul
systemctl enable consul
systemctl status consul
```

```
nano /etc/consul.d/consul.hcl

data_dir = "/opt/consul"
client_addr = "0.0.0.0"
bind_addr = "127.0.0.1"
advertise_addr = "127.0.0.1"
server = true
bootstrap_expect = 1
ui = true
```

- **`server = true`**: This tells this specific instance to run the core Raft engine and act as a decision-maker, rather than just a basic data-forwarding agent.
- **`bootstrap_expect = 1`**: This is exactly what we discussed earlier about bootstrapping a cluster! By setting this to `1`, you are telling this server: _"As soon as you turn on, you don't need to wait for any peers. Calculate your initial quorum requirement as 1, and immediately elect yourself leader."_ This forms a single-node testing cluster.

```
nano /etc/systemd/system/consul.service

[Unit]
Description=Consul
Documentation=https://consul.io
After=network.target

[Service]
ExecStart=/usr/local/bin/consul agent -bind=127.0.0.1 -config-dir=/etc/consul.d
Restart=on-failure
User=consul
Group=consul
LimitNOFILE=65536
LimitNPROC=65536

[Install]
WantedBy=multi-user.target

```

