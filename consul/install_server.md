### Install server's agent (ubuntu/debian)

#### install
Install software-properties-common:
```bash
sudo apt-get update -y
sudo apt-get install -y software-properties-common
```

Add repo with consul:
```consul
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

Add node name, `server1` - does not affect anything but for usability

`node_name = "server1"`

Set address to bind:

`client_addr = "0.0.0.0"`

Enable UI:
```
ui_config{
  enabled = true
}
```

Enable server mode:

`server = true`

Set number of nodes for running cluster. You should use minimum 2 servers to provide no failure tolerance.

`bootstrap_expect=2`

Save the config.

#### start

Start a service:

`systemctl start consul`

Check status:

`systemctl status consul`

Enable autostart for a service:

`systemctl enable consul`

Check consul UI - new node and new 'consul' service have popped up there.
