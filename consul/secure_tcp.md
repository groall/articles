### Set TLS/SSL encryption in consul
#### Generate keys

Create certificate authorities (CA):
```bash
consul tls ca create -days=3650 -domain consul
```

Further, using the example of three consul servers in cluster.

Servers have these domains: consul01.foo-company.net, consul02.foo-company.net и consul03.foo-company.net. 

Common servers' domain 'consul.foo-company.net'. Inside consul network they would have domain 'server.hetzner.consul', where 'hetzner' is datacenter's name and 'consul' - common domain in consul inner network. Both params could be changed in config file.

On the server (where CA created) create keys for all three servers:

First of all, generate a Certificate Signing Requests (CSR):
```bash
openssl req -new -newkey rsa:2048 -nodes -keyout server1.hetzner.consul.key -out server1.hetzner.consul.csr -CAcreateserial -addext 'subjectAltName=DNS:consul.foo-company.net,DNS:consul01.foo-company.net,DNS:server.hetzner.consul,DNS:localhost,IP:127.0.0.1'
openssl req -new -newkey rsa:2048 -nodes -keyout server2.hetzner.consul.key -out server2.hetzner.consul.csr -CAcreateserial -addext 'subjectAltName=DNS:consul.foo-company.net,DNS:consul02.foo-company.net,DNS:server.hetzner.consul,DNS:localhost,IP:127.0.0.1'
openssl req -new -newkey rsa:2048 -nodes -keyout server3.hetzner.consul.key -out server3.hetzner.consul.csr -CAcreateserial -addext 'subjectAltName=DNS:consul.foo-company.net,DNS:consul03.foo-company.net,DNS:server.hetzner.consul,DNS:localhost,IP:127.0.0.1'
```
Note, we use SAN (subjectAltName).

Check first CSR:
```bash
openssl req -noout -text -noout -in server1.hetzner.consul.csr
```

Sign CSR with our CA:
```bash
openssl x509 -req -days 3650 -in server1.hetzner.consul.csr -CA consul-agent-ca.pem -CAkey consul-agent-ca-key.pem -out server1.hetzner.consul.crt \
  -extensions SAN \
  -extfile <(cat /etc/ssl/openssl.cnf \
    <(printf "\n[SAN]\nsubjectAltName=DNS:consul.foo-company.net,DNS:consul01.foo-company.net,DNS:server.hetzner.consul,DNS:localhost,IP:127.0.0.1")) 

openssl x509 -req -days 3650 -in server2.hetzner.consul.csr -CA consul-agent-ca.pem -CAkey consul-agent-ca-key.pem -CAcreateserial -out server2.hetzner.consul.crt \
  -extensions SAN \
  -extfile <(cat /etc/ssl/openssl.cnf \
    <(printf "\n[SAN]\nsubjectAltName=DNS:consul.foo-company.net,DNS:consul02.foo-company.net,DNS:server.hetzner.consul,DNS:localhost,IP:127.0.0.1")) 

openssl x509 -req -days 3650 -in server3.hetzner.consul.csr -CA consul-agent-ca.pem -CAkey consul-agent-ca-key.pem -CAcreateserial -out server3.hetzner.consul.crt \
  -extensions SAN \
  -extfile <(cat /etc/ssl/openssl.cnf \
    <(printf "\n[SAN]\nsubjectAltName=DNS:consul.foo-company.net,DNS:consul03.foo-company.net,DNS:server.hetzner.consul,DNS:localhost,IP:127.0.0.1")) 
```

Check first key:
```bash
openssl x509 -text -noout -in server1.hetzner.consul.crt
```

Common name (CN) shouldn't be set. But if you would like, it can be done by param `-subj '/CN=server.hetzner.consul`

Instead of openssl you can use consul to generate keys:

```bash
consul tls ca create -name-constraint=true -additional-name-constraint=consul.foo-company.net -cluster-id test -common-name "Consul Agent CA" -days=365 -domain consul
consul tls cert create -server -dc hetzner -additional-dnsname=consul.foo-company.net
```

#### Deploys keys

Copy generated keys on three servers. `server1.hetzner.consul.crt` and `server1.hetzner.consul.key` to first server, `server2.hetzner.consul.crt`, `server2.hetzner.consul.key` to second and `server3.hetzner.consul.crt`,`server3.hetzner.consul.key` to third.
Copy `consul-agent-ca.pem` on all servers.

Make `consul:consul` as the owner and set chmod `640`.

#### Configure servers

Add to config each server, just change keys paths:
```hcl
verify_incoming = false
verify_incoming_rpc = true
verify_outgoing = true
verify_server_hostname = true

ca_file = "/etc/consul.d/consul-agent-ca.pem"
cert_file = "/etc/consul.d/server2.hetzner.consul.crt"
key_file = "/etc/consul.d/server2.hetzner.consul.key"

auto_encrypt {
  allow_tls = true
}

ports {
  http = -1
  https = 8501
}

connect {
  enabled = true
}
```

Restart agents:
```bash
systemctl restart consul
systemctl status consul
```

Now the consul works only over HTTPS and only over secure RPC. But the HTTPS client have not being verified. To check incoming http request set `verify_incoming = true` and generate client's keys.

#### Configure clients
Add to config on each client:

```hcl
verify_incoming = false
verify_outgoing = true
verify_server_hostname = true
ca_file = "/etc/consul.d/consul-agent-ca.pem"
auto_encrypt = {
  tls = true
}

connect {
  enabled = true
}

ports {
  grpc = 8502
}

```

Copy `consul-agent-ca.pem` on all clients.

Restart agents:
```bash
systemctl restart consul
systemctl status consul
```

#### Access to CLI

For example use additional params:
```bash
consul members -http-addr="https://consul.foo-company.net:8501" -ca-file="consul-agent-ca.pem"
```

Or set environment vars:
```bash
export CONSUL_HTTP_SSL=true
export CONSUL_HTTP_ADDR=localhost:8501
export CONSUL_CACERT=/etc/consul.d/consul-agent-ca.pem
```

Official tutorials:
* [https://learn.hashicorp.com/tutorials/consul/tls-encryption-openssl-secure?in=consul/security-operations настройка SSL с помощью openssl]
* [https://learn.hashicorp.com/tutorials/consul/tls-encryption-secure-existing-datacenter как настроить SSL в существующем ДЦ]
* [https://learn.hashicorp.com/tutorials/consul/tls-encryption-secure гайд по безопасности]
