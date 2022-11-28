# ELK

## API - Dev Tools [🔗cURL](https://curl.se/docs/manpage.html)<br>

**User**
```yml
GET _security/user
GET _security/user/<user>
```

**Role**
```yml
GET _security/role
GET _security/role/<role>
```

**Index**
```yml
GET <index>
GET _cat/indices/<index>*?v=true&s=index
GET _cat/indices/<index>*?v=true&s=index
GET <index>/_settings?filter_path=*.settings.index.lifecycle
GET <index>*/_settings?filter_path=*.settings.index.lifecycle
GET _cat/indices/<index>*?v=true&s=index&h=index,store.size

GET _component_template
GET _component_template/<component_template>

GET _index_template
GET _index_template/<index_template>
```

**Aliases**
```yml
GET _aliases
GET _aliases/<alias>
```

**ILM**
```yml
GET _ilm/policy
GET _ilm/policy/<policy>
GET <index>/_ilm/explain?human
GET <index>*/_ilm/explain?human
GET /_ilm/status
```

**Shards**
```yml
GET _cat/shards/?v=true
GET _cat/shards/<index>?v=true&s=prirep
GET _cat/shards/<index>*?v=true&s=prirep
```

**Elastic System**
```yml
*HEALTH*
GET _cat/health?v&pretty

*ILM*
GET _cluster/settings?include_defaults=true&filter_path=*.indices.lifecycle*,*.xpack.ilm*

*MASTER NODE*
GET _cat/master?v&pretty

*ALL NODES*
GET _cat/nodes?v&pretty

*TASKS*
GET _cat/tasks?v&pretty

*DISK USAGE*
GET _cat/allocation?v&pretty
```

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

# TEMPLATE

## Spaces ▶︎ `Management > Spaces`<br>

**Create space `new_space`[🔗Spaces](https://www.elastic.co/guide/en/kibana/master/xpack-spaces.html)**<br>
- **+ Create space**
  - **Name:** `new_space`
  - **Background color:** `background_color`
  - **Analytics**
    - Discover
    - Dashboard
    - Canvas
    - Maps
    - Machine Learning
    - Visualize Library
  - **Observability**
    - Logs
    - Metrics
  - **Management**
    - Dev Tools
    - Index Pattern Management

## User ▶︎ `Dev Tools`<br>

**Create `new_user` with default role `run_as`[🔗Configure security in Kibana API](https://www.elastic.co/guide/en/kibana/7.17/using-kibana-with-security.html)**<br>
```json
POST _security/user/<new_user>
{
  "password" : "${openssl rand -hex 32}",
  "roles" : ["run_as"],
  "full_name" : "<full_name>",
  "email" : "<email>"
}
```

**Add `new_user` to role `run_as` [🔗Create or update roles API](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/security-api-put-role.html)**<br>
```json
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
```

## Role ▶︎ `Dev Tools`<br>

**Create role `new_role`, including `new_space` and `new_user` [🔗Create or update roles API](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/security-api-put-role.html)**<br>
```json
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
```

**Create `new_user` and set `new_role` [🔗Configure security in Kibana API](https://www.elastic.co/guide/en/kibana/7.17/using-kibana-with-security.html)**<br>
```json
POST _security/user/<new_user>
{
  "password" : "${openssl rand -hex 32}",
  "roles" : ["run_as","<new_role>"],
  "full_name" : "<full_name>",
  "email" : "<email>"
}
```

## ILM ▶︎ `Dev Tools`<br>

**Create `new_policy` [🔗Create or update lifecycle policy API](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/ilm-put-lifecycle.html)**<br>
```json
POST _ilm/policy/<new_policy>
{
    "policy": {
      "phases" : {
        "hot" : {
          "min_age" : "0ms",
          "actions" : {
            "rollover" : {
              "max_docs" : 40000000
            }
          }
        },
        "warm" : {
          "min_age" : "1d",
          "actions" : {
            "allocate" : {
              "number_of_replicas" : 0,
              "include" : { },
              "exclude" : { },
              "require" : { }
            },
            "shrink" : {
              "number_of_shards" : 1
            }
          }
        },
        "cold" : {
          "min_age" : "2d",
          "actions" : { }
        },
        "delete" : {
          "min_age" : "7d",
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

## Index ▶︎ `Dev Tools`<br>

**Create `new_component_template` [🔗Create or update component template API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-component-template.html)**<br>
```json
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
```

**Create `new_index_template` [🔗Create or update index template API](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/indices-templates-v1.html)**<br>
```json
PUT _index_template/<new_index_template>
{
  "index_patterns": [
    "<new_index>*"
  ],
  "composed_of": [
    "<new_component_template>"
  ]
}
```

**Create `new_index` [🔗Add an alias at index creation](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/aliases.html#add-alias-at-creation)**<br>
```json
PUT <new_index>-000001
{
  "aliases": {
    "<new_alias>": {
      "is_write_index": true
    }
  }
}
```

**Allow access `new_index` to all users belong to `new_role` [🔗Create or update roles API](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/security-api-put-role.html)**<br>
```json
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
```

## Index Pattern ▶︎ `Management > Stack Management > Kibana > Index Patterns`<br>

**Create `new_index_pattern` after receiving the first log in `new_index` [🔗Create an index pattern](https://www.elastic.co/guide/en/kibana/7.17/index-patterns.html#settings-create-pattern)**<br>
- **+ Create index pattern**
  - **Name:** `new_index`
  - **Timestamp field:** @timestamp

## SEND LOGS
```bash
#!/bin/bash
for i in {1..25000}
do
        NOW=$(date '+%Y-%m-%dT%H:%M:%S.%3NZ')
        curl -H "Content-Type: application/json" -X POST "http://<ip>:9200/server-logs/_doc" -d '{"@timestamp": "'"${NOW}"'","info": "Lorem ipsum dolor sit amet, consectetur adipiscing elit.","environment": "stg"}'
        echo "" 
done

$ chmod +x loop.sh
$ bash loop.sh
```

## Allow/Deny Logs - API ▶︎ `Dev Tools`<br>
```json
PUT /_cluster/settings
{
  "persistent" : {
    "xpack.security.transport.filter.allow" : "172.16.0.0/24"
  }
}
```
```json
PUT /_cluster/settings
{
  "persistent" : {
    "xpack.security.transport.filter.enabled" : false
  }
}
```
```json
PUT /_cluster/settings
{
  "persistent" : {
    "xpack.security.transport.filter.allow": "192.168.56.1",
    "xpack.security.transport.filter.deny": "192.168.56.0/24"
  }
}
```

You configure IP filtering by specifying the *xpack.security.transport.filter.allow* and *xpack.security.transport.filter.deny*.<br>
Allow rules take precedence over the deny rules<br>
```yml
xpack.security.transport.filter.allow: "192.168.0.1"
xpack.security.transport.filter.deny: "192.168.0.0/24"
```

The *_all* keyword can be used to *deny all connections* that are not *explicitly* allowed<br>
```yml
xpack.security.transport.filter.allow: [ "192.168.0.1", "192.168.0.2", "192.168.0.3", "192.168.0.4" ]
xpack.security.transport.filter.deny: _all
```

IP filtering configuration also support *IPv6 addresses*<br>
```yml
xpack.security.transport.filter.allow: "2001:0db8:1234::/48"
xpack.security.transport.filter.deny: "1234:0db8:85a3:0000:0000:8a2e:0370:7334"
```

You can also filter by hostnames when *DNS lookups* are available<br>
```yml
xpack.security.transport.filter.allow: localhost
xpack.security.transport.filter.deny: '*.google.com'
```

*Disabling IP filtering* can slightly improve performance under some conditions<br>
To disable IP filtering entirely:
```yml
xpack.security.transport.filter.enabled: false
```

You can also disable IP filtering for the transport protocol but enable it for HTTP only<br>
```yml
xpack.security.transport.filter.enabled: false
xpack.security.http.filter.enabled: true
```

## filebeat.yml
```bash
$ ssh fabio@mgm-manager01.manager.bk.sapo.pt
$ kubectl -n logging --context conteudos get cm filebeat-filebeat-daemonset-config -o yaml
```

**Links**<br>
https://www.elastic.co/guide/en/elasticsearch/reference/current/scalability.html<br>
https://dattell.com/data-architecture-blog/elasticsearch-shards-definitions-sizes-optimizations-and-more/<br>
<br />
https://www.youtube.com/watch?v=z4zU5BoMixY&ab_channel=TechnologyCentral<br>
https://www.youtube.com/watch?v=NUk9kExOlAg&ab_channel=CodeCloud%26Data<br>
https://www.tutorialkart.com/bash-shell-scripting/bash-date-format-options-examples/<br>
https://www.elastic.co/guide/en/elasticsearch/reference/current/ip-filtering.html<br>
https://aravind.dev/elastic-data-stream/<br>
https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html<br>
https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-shards.html<br>
https://www.elastic.co/guide/en/elasticsearch/reference/current/size-your-shards.html
<br />
https://discuss.elastic.co/t/import-ca-cert-as-privatekeyentry-to-http-keystore-solve-unable-to-create-enrollment-token-error/313780<br>
https://www.elastic.co/guide/en/elasticsearch/reference/master/security-basic-setup.html<br>
https://www.baeldung.com/ops/docker-compose-multiple-commands<br>
https://stackoverflow.com/questions/67108012/where-does-elasticsearch-certificates-located<br>
https://github.com/elastic/elasticsearch/issues/32531<br>
https://www.ibm.com/docs/en/sle/10.2.0?topic=elasticsearch-enabling-https<br>
<br />
https://stackoverflow.com/questions/66236879/get-the-filtered-response-in-elasticsearch-cat-apis<br>
https://linuxhint.com/elasticsearch-shard-list/<br>
https://keepgrowing.in/tools/how-to-find-and-diagnose-unassigned-elasticsearch-shards/<br>
https://www.alibabacloud.com/blog/597074<br>