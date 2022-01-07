### Install client's agent (ubuntu/debian)

#### install
Install software-properties-common:
```bash
sudo apt-get update -y
sudo apt-get install -y software-properties-common
```

Add repo with consul:
```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install consul
```

#### configure
Open config's file for edit:
```bash
mcedit /etc/consul.d/consul.hcl
```

Set datacenter's name. All agents on a cluster should have the same 'datacenter' value:

`datacenter = "Hetzner"`

Set server's address:

`retry_join = ["consul.foo-company.net"]`

Save the config.

#### start

Start a service:
```bash
systemctl start consul`
```

Check status:
```bash
systemctl status consul
```

Enable autostart for a service:
```bash
systemctl enable consul
```

Check consul UI - our new node have popped up there.
