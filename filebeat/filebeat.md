# Filebeat

## General

**Commands**
```bash
sudo filebeat modules list
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