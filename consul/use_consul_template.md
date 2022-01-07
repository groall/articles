### Using consul-template

For example, there is a config file for some API `/opt/projects/api/config.yaml`. Need to update  it and restart API's instance after that.

Create config file and fill it:

```bash
touch /etc/consul-template.d/templates.hcl
chown -R consul:consul /etc/consul-template.d/templates.hcl
mcedit /etc/consul-template.d/templates.hcl
```

```hcl
template {
  source = "/opt/projects/api/config.yaml.ctmpl"
  destination = "/opt/projects/api/config.yaml"
  command = "sudo /bin/systemctl restart api.service"
  command_timeout = "10s"
  error_on_missing_key = false
  backup = true
  wait {
    min = "2s"
    max = "10s"
  }
}
```

User 'consul' can't manage service 'api', to do it:

Create a file `/etc/sudoers.d/consul` and put there these lines:
```
%consul ALL=NOPASSWD: /bin/systemctl start api.service
%consul ALL=NOPASSWD: /bin/systemctl stop api.service
%consul ALL=NOPASSWD: /bin/systemctl restart api.service
```

Create template file and destination file. For example template file:
```yaml
host:
  params: parseTime=true&loc=Local&timeout=5s
  max_idle_conns: {{keyOrDefault "api/host/max_idle_conns" "5"}}
  max_open_conns: {{keyOrDefault "api/host/max_open_conns" "50"}}
```

Reload consul-template.

[quick example on a project's page](https://github.com/hashicorp/consul-template#quick-example)

[more docs](https://gitee.com/ucasrichard/consul-template)
