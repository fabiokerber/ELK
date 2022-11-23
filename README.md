# ELK

**Links**<br>
https://www.youtube.com/watch?v=z4zU5BoMixY&ab_channel=TechnologyCentral<br>
https://www.youtube.com/watch?v=NUk9kExOlAg&ab_channel=CodeCloud%26Data<br>
https://www.tutorialkart.com/bash-shell-scripting/bash-date-format-options-examples/<br>
https://www.elastic.co/guide/en/elasticsearch/reference/current/ip-filtering.html<br>
https://aravind.dev/elastic-data-stream/<br>
https://dattell.com/data-architecture-blog/elasticsearch-shards-definitions-sizes-optimizations-and-more/<br>
https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html<br>
https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-shards.html<br>
https://www.elastic.co/guide/en/elasticsearch/reference/current/size-your-shards.html
<br>
https://discuss.elastic.co/t/import-ca-cert-as-privatekeyentry-to-http-keystore-solve-unable-to-create-enrollment-token-error/313780<br>
https://www.elastic.co/guide/en/elasticsearch/reference/master/security-basic-setup.html<br>
https://www.baeldung.com/ops/docker-compose-multiple-commands<br>
https://stackoverflow.com/questions/67108012/where-does-elasticsearch-certificates-located<br>
https://github.com/elastic/elasticsearch/issues/32531<br>
https://www.ibm.com/docs/en/sle/10.2.0?topic=elasticsearch-enabling-https<br>

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

**API**
```json
GET _ilm/policy/enterprise-logs-policy
GET _component_template/enterprise-logs-component-template
GET _index_template/enterprise-logs
GET enterprise-survey-000001
GET entreprise-survey-*/_ilm/explain
GET _cat/shards/filebeat-k8s-sapo-conteudos-lifestylesapopt-prod-fe*?v=true
GET _cat/indices/filebeat-k8s-sapo-conteudos-lifestylesapopt-prod-fe*?v=true&s=index

GET _ilm/policy/logs-policy
GET _component_template/logs-component-template
GET _index_template/logs

DELETE _data_stream/log-stream
DELETE _index_template/logs-index-template
DELETE _component_template/logs-component-template
DELETE _ilm/policy/logs-policy
```

# TEMPLATE

## Spaces ▶︎ `Management > Spaces`<br>

**Create space `new_space`[🔗Spaces](https://www.elastic.co/guide/en/kibana/master/xpack-spaces.html)**<br>
>__"+ Create space"__
  - **Name:** `new_space`
  - **Background color:** `background_color`
  - '''Analytics'''
    - Discover
    - Dashboard
    - Canvas
    - Maps
    - Machine Learning
    - Visualize Library
  - '''Observability'''
    - Logs
    - Metrics
  - '''Management'''
    - Dev Tools
    - Index Pattern Management


## User ▶︎ `Dev Tools`<br>

**Create `new_user` with default role `run_as`**<br>
[🔗Configure security in Kibana API](https://www.elastic.co/guide/en/kibana/7.17/using-kibana-with-security.html)

```json
POST _security/user/<new_user>
{
  "password" : "${openssl rand -hex 32}",
  "roles" : ["run_as"],
  "full_name" : "<full_name>",
  "email" : "<email>"
}
```

'''*''' Add `new_user` to role `run_as` ^([https://www.elastic.co/guide/en/elasticsearch/reference/7.17/security-api-put-role.html Create or update roles API])^
{{{#!json
POST _security/role/run_as
{
  "cluster": [],
  "indices": [],
  "run_as": [ 
    "<new_user>"],
  "metadata" : {
    "version" : 1
  }
}
}}}


= Role ^([https://elk.bk.sapo.pt/s/default/app/dev_tools Management > Dev Tools)]^ =

'''*''' Create role `new_role`, including `new_space` and `new_user` ^([https://www.elastic.co/guide/en/elasticsearch/reference/7.17/security-api-put-role.html Create or update roles API])^
{{{#!json
POST _security/role/<new_role>
{
  "cluster" : [ ],
  "indices" : [
    {
      "names" : [ ],
      "privileges" : [
        "all"
      ],
      "allow_restricted_indices" : false
    }
  ],
  "applications" : [
    {
      "application" : "kibana-.kibana",
      "privileges" : [
          "feature_discover.all",
          "feature_dashboard.all",
          "feature_canvas.all",
          "feature_maps.all",
          "feature_ml.all",
          "feature_visualize.all",
          "feature_logs.all",
          "feature_dev_tools.all",
          "feature_indexPatterns.all",
          "feature_infrastructure.all"
      ],
      "resources" : [
        "<new_space>"
      ]
    }
  ],
  "run_as" : [
    "<new_user>"
  ],
  "metadata" : {
    "version" : 1
  },
  "transient_metadata" : {
    "enabled" : true
    }
}
}}}

'''*''' Create `new_user` and set `new_role` ^([https://www.elastic.co/guide/en/kibana/7.17/using-kibana-with-security.html Configure security in Kibana API])^
{{{#!json
POST _security/user/<new_user>
{
  "password" : "${openssl rand -hex 32}",
  "roles" : ["run_as","<new_role>"],
  "full_name" : "<full_name>",
  "email" : "<email>"
}
}}}


= ILM ^([https://elk.bk.sapo.pt/s/default/app/dev_tools Management > Dev Tools)]^ =

'''*''' Create `new_policy` ^([https://www.elastic.co/guide/en/elasticsearch/reference/7.17/ilm-put-lifecycle.html Create or update lifecycle policy API])^
{{{#!json
POST _ilm/policy/<new_policy>
{
    "policy": {
      "phases" : {
        "hot" : {
          "min_age" : "0ms",
          "actions" : {
            "rollover" : {
              "max_age" : "7d"
            }
          }
        },
        "warm" : {
          "min_age" : "1d",
          "actions" : {
            "shrink" : {
              "number_of_shards" : 1
            }
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
}}}


= Index ^([https://elk.bk.sapo.pt/s/default/app/dev_tools Management > Dev Tools)]^ =

'''*''' Create `new_component_template` ^([https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-component-template.html Create or update component template API])^
{{{#!json
PUT _component_template/<new_component_template>
{
  "template": {
    "settings": {
      "number_of_shards": 3,
      "number_of_replicas": 1,
      "index.lifecycle.name": "<new_policy>",
      "index.lifecycle.rollover_alias": "<new_alias>"
    },
    "mappings": {
      "properties": {
        "created_at": {
          "type": "date",
          "format": "EEE dd MMM yyyy HH:mm:ss Z"
        }
      }
    }
  }
}
}}}

'''*''' Create `new_index_template` ^([https://www.elastic.co/guide/en/elasticsearch/reference/7.17/indices-templates-v1.html Create or update index template API])^
{{{#!json
PUT _index_template/<new_index_template>
{
  "index_patterns": [
    "<new_index>*"
  ],
  "composed_of": [
    "<new_component_template>"
  ]
}
}}}

'''*''' Create `new_index` ^([https://www.elastic.co/guide/en/elasticsearch/reference/7.17/aliases.html#add-alias-at-creation Add an alias at index creation])^
{{{#!json
PUT <new_index>-000001
{
    "aliases": {
        "<new_alias>": {
            "is_write_index": true
        }
    }
}
}}}

'''*''' Allow access `new_index` to all users belong to `new_role` ^([https://www.elastic.co/guide/en/elasticsearch/reference/7.17/security-api-put-role.html Create or update roles API])^
{{{#!json
POST _security/role/<new_role>
{
  "cluster" : [ ],
  "indices" : [
    {
      "names" : [
        "<new_index>*"
      ],
      "privileges" : [
        "all"
      ],
      "allow_restricted_indices" : false
    }
  ],
  "applications" : [
    {
      "application" : "kibana-.kibana",
      "privileges" : [
          "feature_discover.all",
          "feature_dashboard.all",
          "feature_canvas.all",
          "feature_maps.all",
          "feature_ml.all",
          "feature_visualize.all",
          "feature_logs.all",
          "feature_dev_tools.all",
          "feature_indexPatterns.all",
          "feature_infrastructure.all"
      ],
      "resources" : [
        "<new_space>"
      ]
    }
  ],
  "run_as" : [
    "<new_user>"
  ],
  "metadata" : {
    "version" : 1
  },
  "transient_metadata" : {
    "enabled" : true
    }
}
}}}

= Index Pattern ^([https://elk.bk.sapo.pt/s/default/app/dev_tools Management > Dev Tools)]^ =

'''*''' Create `new_index_pattern` after receiving the first log in `new_index` ^([https://www.elastic.co/guide/en/kibana/7.17/index-patterns.html#settings-create-pattern Create an index pattern])^
* '''Go to Space `new_space` > Management > Stack Management > Kibana > Index Patterns'''
  - '''[[span(style=color:  #0032ff, + Create index pattern)]]'''
    - '''Name:''' <new_index>*
    - '''Timestamp field:''' @timestamp

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