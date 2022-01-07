### Add new service on agent

Agent must be [installed](install_client.md)

For example there are two REST API services: 'api1' and 'api2' on one host/container. First service binds on port 8090, second on port 8091. 

Create two config files:

```bas 
touch /etc/consul.d/api1.json
touch /etc/consul.d/api2.json
chown -R consul:consul /etc/consul.d/api1.json
chown -R consul:consul /etc/consul.d/api2.json
```

Fill the first file:
```json
{
    "service": {
        "name": "api",
        "id": "api1",
        "port": 8090,
        "tags": ["ts", "api", "golang"],
        "check": {
            "http": "http://localhost:8090/check",
            "method": "GET",
            "header": {"Content-Type": ["application/json"]},
            "interval": "10s",
            "timeout": "1s"
        }
    }
}
```


Fill the second file:
```json
{
    "service": {
        "name": "api",
        "id": "api2",
        "port": 8090,
        "tags": ["ts", "api", "golang"],
        "check": {
            "http": "http://localhost:8091/check",
            "method": "GET",
            "header": {"Content-Type": ["application/json"]},
            "interval": "10s",
            "timeout": "1s"
        }
    }
}
```

Explanations :
* `"name": "api"` - service's name, all instances of one service should have the same 'name'
* `"id": "api1"` - id of instance (could be uniq or not)
* `"port": 8091` - port of the service, can be using in autodiscovery
* `"check"` - monitoring
* `"http": "http://localhost:8091/check"` - URL to check HTTP services
* `"method": "GET"` - method to make HTTP request


[More info about monitoring](https://www.consul.io/docs/discovery/checks)

[check tutorial](https://learn.hashicorp.com/tutorials/consul/service-registration-health-checks)