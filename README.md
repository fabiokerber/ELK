# ELK

**Links**
https://www.youtube.com/watch?v=z4zU5BoMixY&ab_channel=TechnologyCentral<br>
https://www.youtube.com/watch?v=NUk9kExOlAg&ab_channel=CodeCloud%26Data<br>
https://www.tutorialkart.com/bash-shell-scripting/bash-date-format-options-examples/<br>

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
```yml
PUT _ilm/policy/enterprise-logs-policy
{
    "policy": {
      "phases" : {
        "hot" : {
          "min_age" : "0ms",
          "actions" : {
            "rollover" : {
              "max_primary_shard_size" : "200kb"
            }
          }
        },
        "warm" : {
          "min_age" : "5m",
          "actions" : {
            "shrink" : {
              "number_of_shards" : 1
            }
          }
        },
        "cold" : {
          "min_age" : "10m",
          "actions" : { 
            "freeze": { }
          }
        },
        "delete" : {
          "min_age" : "15m",
          "actions" : {
            "delete" : {
              "delete_searchable_snapshot" : true
            }
          }
        }
      }
    }
}

PUT _component_template/enterprise-logs-component-template
{
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0,
      "index.lifecycle.name": "enterprise-logs-policy",
      "index.lifecycle.rollover_alias": "enterprise-logs"
    }
  }
}

PUT _index_template/enterprise-logs
{
  "index_patterns": [
    "enterprise-logs-*"
  ],
  "composed_of": [
    "enterprise-logs-component-template"
  ],
  "template": {
    "settings": {
      "index.lifecycle.name": "enterprise-logs-policy"
    }
  }
}

PUT enterprise-logs-000001
{
    "aliases": {
        "enterprise-logs": {
            "is_write_index": true
        }
    }
}

POST enterprise-logs/_doc
{
  "timestamp": "'"${NOW}"'",
  "info": "some infos",
  "environment": "test"
}

CREATE INDEX PATTERN
Name: enterprise-logs
Index pattern: enterprise-logs-*

GET _ilm/policy/enterprise-logs-policy
GET _component_template/enterprise-logs-component-template
GET _index_template/enterprise-logs
GET enterprise-survey-000001
GET entreprise-survey-*/_ilm/explain

GET _ilm/policy/logs-policy
GET _component_template/logs-component-template
GET _index_template/logs

DELETE _data_stream/log-stream
DELETE _index_template/logs-index-template
DELETE _component_template/logs-component-template
DELETE _ilm/policy/logs-policy
```

```
PUT _component_template/enterprise-logs-new-component-template
{
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0,
      "index.lifecycle.name": "enterprise-logs-policy",
      "index.lifecycle.rollover_alias": "enterprise-logs-new"
    }
  }
}

PUT _index_template/enterprise-logs
{
  "index_patterns": [
    "enterprise-logs-new-*"
  ],
  "composed_of": [
    "enterprise-logs-component-template"
  ],
  "template": {
    "settings": {
      "index.lifecycle.name": "enterprise-logs-new-policy"
    }
  }
}

PUT enterprise-logs-new-000001
{
    "aliases": {
        "enterprise-logs-new": {
            "is_write_index": true
        }
    }
}
```

**Bash**
```
#!/bin/bash
NOW=$(date '+%b %d, %Y @ %H:%M:%S')
for i in {1..100}
do
curl -H "Content-Type: application/json" -X POST "http://192.168.56.185:9200/enterprise-logs/_doc" -d '{"timestamp": "'"${NOW}"'","info": "some infos","environment": "test"}'
done

$ chmod +x loop.sh
$ bash loop.sh
```