### Secure gossip communications (UDP)

Agents communicate with each other not only over TCP, but also over UDP.

For this reason, agents can communicate with others after TLS is enabled. You must secure your gossip to create a secure network.

Generate a key:

```bash
consul keygen
````

Add a key in a config (on servers and clients): 

`encrypt = "your key here"`

Restart agent (on all servers and clients):
```bash
systemctl restart consul
systemctl status consul
```

(official guide)[https://learn.hashicorp.com/tutorials/consul/gossip-encryption-secure]
