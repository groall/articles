### Auto scraping services list in prometheus

[Install consul agent](install_client.md) on host with Prometheus. Set node_name = 'prometheus'

Create token with the rules:
```hcl
node "prometheus" {
  policy = "write"
}
agent_prefix "" {
  policy = "read"
}
node_prefix "" {
  policy = "read"
}
service_prefix "" {
  policy = "read"
}
```

You can restrict these rules to fewer rights if you like.

Add job in Prometheus config:
```yml
    # abcp-api
    - job_name: api
      consul_sd_configs:
      - server: "localhost:8500"
        token: <your token here>
        services:
          - "api"
      relabel_configs:
        - source_labels: [__meta_sd_consul_tags]
          separator: ","
          regex: label:([^=]+)=([^,]+)
          target_label: ${1}
          replacement: ${2}
```
