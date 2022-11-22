# ELK

```yml
> Vagrantfile
  elk.vm.provision 'shell', inline: 'dockerd --max-concurrent-downloads 2 &>/dev/null' ("Weak Network")

- TOKEN -
$ vagrant up elk
$ vagrant ssh elk
$ sudo docker exec -it elasticsearch /bin/bash
$ /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token --scope kibana

> http://LOG_IP:3000
  admin
  admin

> http://LOG_IP:5601

> http://LOG_IP:9200
```

**COMMANDS**
```yml
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

**POLICY**
```yml
PUT /_ilm/policy/enterprise-logs-policy
{
    "policy": {
      "phases" : {
        "hot" : {
          "min_age" : "0ms",
          "actions" : {
            "rollover" : {
              "max_primary_shard_size" : "500kb"
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
```

**LAB 1 - INDEX**
```yml
PUT /_component_template/server-logs-component-template
{
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0,
      "index.lifecycle.name": "enterprise-logs-policy",
      "index.lifecycle.rollover_alias": "server-logs"
    }
  }
}

PUT /_index_template/server-logs
{
  "index_patterns": [
    "server-logs-*"
  ],
  "composed_of": [
    "server-logs-component-template"
  ],
  "template": {
    "settings": {
      "index.lifecycle.name": "enterprise-logs-policy"
    }
  }
}

PUT /server-logs-000001
{
    "aliases": {
        "server-logs": {
            "is_write_index": true
        }
    }
}
 ```

**CREATE INDEX PATTERN**
Name: server-logs
Index pattern: server-logs-*

```yml
PUT /_component_template/network-logs-component-template
{
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0,
      "index.lifecycle.name": "enterprise-logs-policy",
      "index.lifecycle.rollover_alias": "network-logs"
    }
  }
}

PUT /_index_template/network-logs
{
  "index_patterns": [
    "network-logs-*"
  ],
  "composed_of": [
    "network-logs-component-template"
  ],
  "template": {
    "settings": {
      "index.lifecycle.name": "enterprise-logs-policy"
    }
  }
}

PUT /network-logs-000001
{
    "aliases": {
        "network-logs": {
            "is_write_index": true
        }
    }
}
```

**CREATE INDEX PATTERN**
Name: network-logs
Index pattern: network-logs-*

**Send Logs**
```bash
#!/bin/bash
for i in {1..25000}
do
        NOW=$(date '+%b %d, %Y @ %H:%M:%S.%3N')
        curl -H "Content-Type: application/json" -X POST "http://192.168.56.185:9200/server-logs/_doc" -d '{"@timestamp": "'"${NOW}"'","info": "Lorem ipsum dolor sit amet, consectetur adipiscing elit.","environment": "stg"}'
        echo "" 
done

$ chmod +x loop.sh
$ bash loop.sh
```

**Allow/Deny Logs - API**

Example:
```yml
PUT /_cluster/settings
{
  "persistent" : {
    "xpack.security.transport.filter.allow" : "172.16.0.0/24"
  }
}
```
```yml
PUT /_cluster/settings
{
  "persistent" : {
    "xpack.security.transport.filter.enabled" : false
  }
}
```

Example Lab:<br>
```yml
PUT /_cluster/settings
{
  "persistent" : {
    "xpack.security.transport.filter.allow": "192.168.56.1",
    "xpack.security.transport.filter.deny": "192.168.56.0/24"
  }
}
```
You configure IP filtering by specifying the *xpack.security.transport.filter.allow* and *xpack.security.transport.filter.deny*.<br>
Allow rules take precedence over the deny rules.<br>
```yml
xpack.security.transport.filter.allow: "192.168.0.1"
xpack.security.transport.filter.deny: "192.168.0.0/24"
```

The *_all* keyword can be used to *deny all connections* that are not *explicitly* allowed.
```yml
xpack.security.transport.filter.allow: [ "192.168.0.1", "192.168.0.2", "192.168.0.3", "192.168.0.4" ]
xpack.security.transport.filter.deny: _all
```

IP filtering configuration also support *IPv6 addresses*.
```yml
xpack.security.transport.filter.allow: "2001:0db8:1234::/48"
xpack.security.transport.filter.deny: "1234:0db8:85a3:0000:0000:8a2e:0370:7334"
```

You can also filter by hostnames when *DNS lookups* are available.
```yml
xpack.security.transport.filter.allow: localhost
xpack.security.transport.filter.deny: '*.google.com'
```

*Disabling IP filtering* can slightly improve performance under some conditions.<br>
To disable IP filtering entirely:
```yml
xpack.security.transport.filter.enabled: false
```

You can also disable IP filtering for the transport protocol but enable it for HTTP only.
```yml
xpack.security.transport.filter.enabled: false
xpack.security.http.filter.enabled: true
```