# Filebeat

## General

**Commands**
```bash
sudo filebeat modules list
docker rm -f filebeat && docker compose -f /vagrant/files/filebeat-docker-compose.yml up -d && docker logs -f filebeat
```

**Logs Policy**
```json
PUT _ilm/policy/filebeat-7.17.3-srv01-policy
{
    "policy": {
      "phases" : {
        "hot" : {
          "min_age" : "0ms",
          "actions" : {
            "rollover" : {
              "max_docs" : 10000
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

**+ Create index pattern**
  - **Name:** `new_alias`
  - **Timestamp field:** @timestamp

**Loop Bash**
```bash
#!/bin/bash
for i in {1..25000}
do
        echo 'Sequence:' $i
        echo 'Lorem ipsum dolor sit amet, consectetur adipiscing elit' | sudo tee -a /var/log/server.log
        echo 'Ut enim ad minim veniam' | sudo tee -a /var/log/server.log
        echo 'Nemo enim ipsam voluptatem quia voluptas sit aspernatur aut odit aut fugit' | sudo tee -a /var/log/server.log
        echo 'Quis autem vel eum iure reprehenderit qui in ea voluptate velit' | sudo tee -a /var/log/network.log
        echo 'Ut enim ad minim veniam' | sudo tee -a /var/log/network.log
        echo 'Nam libero tempore, cum soluta nobis est eligendi optio cumque' | sudo tee -a /var/log/network.log
        echo '--------------'
done
```

## filebeat.yml
```bash
$ ssh fabio@mgm-manager01.manager.bk.sapo.pt
$ kubectl -n logging --context conteudos get cm filebeat-filebeat-daemonset-config -o yaml
```

**Links**<br>
https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-reference-yml.html<br>
https://www.elastic.co/guide/en/beats/filebeat/current/configuration-template.html<br>
https://logit.io/sources/configure/ubuntu/<br>
https://doc.hcs.huawei.com/usermanual/mrs/mrs_01_1501.html<br>
https://logz.io/blog/filebeat-tutorial/<br>
https://github.com/shazChaudhry/docker-elastic/blob/master/filebeat-docker-compose.yml<br>
https://gigi.nullneuron.net/gigilabs/filebeat-elasticsearch-and-kibana-with-docker-compose/<br>
https://linuxhint.com/bash-for-loop-1-to-10/<br>
https://discuss.elastic.co/t/filebeat-output-two-conditions/135130/2<br>
https://stackoverflow.com/questions/72188167/filebeat-how-to-create-new-field-from-the-path<br>
https://appliedaiconsulting.com/what-is-filebeat-and-why-is-it-imperative/<br>