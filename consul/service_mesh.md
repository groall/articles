### Service mesh

#### Enable Connect
Add to config on all agents:
```hcl
connect {
  enabled = true
}
```

And restart agents.

#### Install Envoy and sidecar proxy

How to install and use Envoy you can read [here](https://learn.hashicorp.com/tutorials/consul/service-mesh-with-envoy-proxy).

Be sure to check your environment variables after installation.

#### Create Envoy service's config

Create env config:
```bash
touch /etc/sysconfig/envoy
mcedit /etc/sysconfig/envoy
```

```
CONSUL_HTTP_SSL=true
CONSUL_HTTP_ADDR=127.0.0.1:8501
CONSUL_CACERT=/etc/ssl/certs/consul-ca.pem
#CONSUL_CLIENT_CERT=/etc/consul/client1.hetzner.consul.pem
#CONSUL_CLIENT_KEY=/etc/consul/client1.hetzner.consul.key
#CONSUL_GRPC_ADDR=https://127.0.0.1:8502
```
or without SSL
```
CONSUL_HTTP_SSL=false
CONSUL_HTTP_ADDR=127.0.0.1:8500
```
or you can use cli params like that:
```bash
consul connect envoy -sidecar-for my_api -http-addr http://localhost:8500 -grpc-addr localhost:8502 
```


Create service config:
```bash
touch /usr/lib/systemd/system/envoy-my_api.service
mcedit /usr/lib/systemd/system/envoy-my_api.service
```

Add to the file:
```ini
[Unit]
Description=Start envoy proxy
Documentation=https://learn.hashicorp.com/tutorials/consul/service-mesh-with-envoy-proxy
Requires=local-fs.target
After=local-fs.target

[Service]
Type=simple
ExecStart=/usr/local/bin/consul connect envoy --sidecar-for my_api
EnvironmentFile=-/etc/sysconfig/envoy
KillSignal=SIGINT
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```

Exec:
```bash
systemctl daemon-reload
systemctl start envoy-my_api
systemctl status envoy-my_api
```

If status is 'active', exec to enable autostart new service:

```bash
systemctl enable envoy-my_api
```
