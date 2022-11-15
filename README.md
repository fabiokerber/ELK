# ELK

**Links**
https://www.youtube.com/watch?v=z4zU5BoMixY&ab_channel=TechnologyCentral<br>
https://www.youtube.com/watch?v=NUk9kExOlAg&ab_channel=CodeCloud%26Data<br>
https://www.tutorialkart.com/bash-shell-scripting/bash-date-format-options-examples/<br>
https://www.elastic.co/guide/en/elasticsearch/reference/current/ip-filtering.html<br>

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

PUT _component_template/enterprise-logs-component-template
{
  "template": {
    "settings": {
      "number_of_shards": 10,
      "number_of_replicas": 1,
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

CREATE INDEX PATTERN
Name: enterprise-logs
Index pattern: enterprise-logs-*

```

```yml
PUT _component_template/enterprise-logs-new-component-template
{
  "template": {
    "settings": {
      "number_of_shards": 10,
      "number_of_replicas": 1,
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
    "enterprise-logs-new-component-template"
  ],
  "template": {
    "settings": {
      "index.lifecycle.name": "enterprise-logs-policy"
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

CREATE INDEX PATTERN
Name: enterprise-logs-new
Index pattern: enterprise-logs-new-*
```

**Send Logs**
```
#!/bin/bash
for i in {1..10000}
do
        NOW=$(date '+%b %d, %Y @ %H:%M:%S')
        curl -H "Content-Type: application/json" -X POST "http://192.168.56.185:9200/enterprise-logs-new/_doc" -d '{"timestamp": "'"${NOW}"'","info": "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Vestibulum pulvinar risus et justo consequat, laoreet mattis quam aliquet. Sed lacinia maximus urna, ut rhoncus metus tincidunt eu. Aenean sem urna, convallis eget pharetra eu, molestie id tortor. Suspendisse potenti. Nunc egestas lorem tellus, a consequat leo gravida id.","environment": "test"}'
        echo "" 
done

$ chmod +x loop.sh
$ bash loop.sh
```

**Deny Logs - API**

Example:
```
PUT /_cluster/settings
{
  "persistent" : {
    "xpack.security.transport.filter.allow" : "172.16.0.0/24"
  }
}
```
```
PUT /_cluster/settings
{
  "persistent" : {
    "xpack.security.transport.filter.enabled" : false
  }
}
```
You configure IP filtering by specifying the *xpack.security.transport.filter.allow* and *xpack.security.transport.filter.deny* settings in elasticsearch.yml.<br>
Allow rules take precedence over the deny rules.<br>
```
xpack.security.transport.filter.allow: "192.168.0.1"
xpack.security.transport.filter.deny: "192.168.0.0/24"
```

The *_all* keyword can be used to *deny all connections* that are not *explicitly* allowed.
```
xpack.security.transport.filter.allow: [ "192.168.0.1", "192.168.0.2", "192.168.0.3", "192.168.0.4" ]
xpack.security.transport.filter.deny: _all
```

IP filtering configuration also support *IPv6 addresses*.
```
xpack.security.transport.filter.allow: "2001:0db8:1234::/48"
xpack.security.transport.filter.deny: "1234:0db8:85a3:0000:0000:8a2e:0370:7334"
```

You can also filter by hostnames when *DNS lookups* are available.
```
xpack.security.transport.filter.allow: localhost
xpack.security.transport.filter.deny: '*.google.com'
```

*Disabling IP filtering* can slightly improve performance under some conditions.<br>
To disable IP filtering entirely:
```
xpack.security.transport.filter.enabled: false
```

You can also disable IP filtering for the transport protocol but enable it for HTTP only.
```
xpack.security.transport.filter.enabled: false
xpack.security.http.filter.enabled: true
```