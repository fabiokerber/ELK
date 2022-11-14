# ELK

```
> Vagrantfile
  elk.vm.provision 'shell', inline: 'dockerd --max-concurrent-downloads 2 &>/dev/null' ("Weak Network")

> vagrant up elk

> http://LOG_IP:3000
  admin
  admin

> http://LOG_IP:5601

> http://LOG_IP:9200
```

**LAB 1**
```json
PUT _ilm/policy/logs-policy
{
    "policy": {
      "phases" : {
        "hot" : {
          "min_age" : "0ms",
          "actions" : {
            "rollover" : {
              "max_primary_shard_size" : "1mb"
            },
            "forcemerge": {
              "max_num_segments": 1
            }
          }
        },
        "cold" : {
          "min_age" : "1d",
          "actions" : { 
            "freeze": { }
          }
        },
        "delete" : {
          "min_age" : "2d",
          "actions" : {
            "delete" : {
              "delete_searchable_snapshot" : true
            }
          }
        }
      }
    }
}

PUT _component_template/logs-component-template
{
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0,
      "index.lifecycle.name": "logs-policy"
    }
  }
}

PUT _index_template/logs-index-template
{
  "index_patterns": [
    "logs-stream*"
  ],
  "data_stream": {},
  "composed_of": [
    "logs-component-template"
  ]
}

PUT _data_stream/logs-stream
GET _data_stream/logs-stream
PUT _data_stream/logs-stream-0000001
GET _data_stream/logs-stream-0000001

DELETE _data_stream/log-stream
DELETE _index_template/logs-index-template
DELETE _component_template/logs-component-template
DELETE _ilm/policy/logs-policy
```