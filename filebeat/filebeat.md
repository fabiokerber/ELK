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

## ~/.kube/config
```bash
apiVersion: v1
kind: Config
clusters:
- name: "conteudos"
  cluster:
    server: "https://rancher2.bk.sapo.pt/k8s/clusters/c-vv8bx"
    certificate-authority-data: "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURlekNDQ\
      W1PZ0F3SUJBZ0lJUm1HVFZUME9VWjh3RFFZSktvWklodmNOQVFFTEJRQXdTekVMTUFrR0ExVUUKQ\
      mhNQ1VGUXhHekFaQmdOVkJBb01FbEJVSUVOdmJYVnVhV05oWTI5bGN5QlRRVEVOTUFzR0ExVUVDd\
      3dFVTBGUQpUekVRTUE0R0ExVUVBd3dIVTBGUVR5QkRRVEFlRncweU1UQXpNRGd4TWpBek1EbGFGd\
      zB6TVRBek1EWXhNakF6Ck1EbGFNRXN4Q3pBSkJnTlZCQVlUQWxCVU1Sc3dHUVlEVlFRS0RCSlFWQ\
      0JEYjIxMWJtbGpZV052WlhNZ1UwRXgKRFRBTEJnTlZCQXNNQkZOQlVFOHhFREFPQmdOVkJBTU1CM\
      U5CVUU4Z1EwRXdnZ0VpTUEwR0NTcUdTSWIzRFFFQgpBUVVBQTRJQkR3QXdnZ0VLQW9JQkFRRGE0b\
      nFBckxPQ3BvL2lWdm96VjZvSnYzQ3NhM21aSXNPdG9Wek9NTXA3CkR0d1pJYlpWcG5YTHkxK2VHM\
      lBsdXQ2YTBEWHl3cnd0eDdJT1JTTW1YQUdLRTdiVmJBV1hwamZjRkkzR0pmK1oKdTBQWTZWdkdzV\
      nMyRXpHUTR6b2wwcmx2aWJ6T0dGTkNyczRYdzNCL1c5dkorUGw1Mm1HbjZpYXBtbURMVmt5QwpFd\
      zRlYnRUVHM5ZXBZRzdTbDRkVnQvdnhXejNGTGxYNVV6b0dVc1E5cEJQUVo4ajlDN2NudVhKVnpYR\
      TBOUlhUCnRhOVN4WHJZV0kyMnZlaWxTZG5PclNQeWljN1ZGakhYbVdxR1g3djhvcTg1MzFPVHNKW\
      m5yWmFjRzVTU3lRTUQKWmRvQzIrb0YzdmUySWY0N0FPR3BzZFNHcVpzN0ZLMkVqRm5RMXhxR04yY\
      2pBZ01CQUFHall6QmhNQjBHQTFVZApEZ1FXQkJTSk5aN0tWZHlTTXBzUlhBMXVzVnJqeklxUFZ6Q\
      VBCZ05WSFJNQkFmOEVCVEFEQVFIL01COEdBMVVkCkl3UVlNQmFBRklrMW5zcFYzSkl5bXhGY0RXN\
      nhXdVBNaW85WE1BNEdBMVVkRHdFQi93UUVBd0lCaGpBTkJna3EKaGtpRzl3MEJBUXNGQUFPQ0FRR\
      UFNOVJQb1hTN041ZnlpZDdvK2RvUlArVkt2TG10bUNyazdNemFIRFhtVE9jZApFMEFsL2RIc0MrT\
      2VOakt5V2lLRjk3NnU2NnJmNlZyeVJCRkFuVXpJM01seGhSd0hLVDZuQ3NyZWd5b0pJeDlxCklKS\
      mc0emoza1NoSFpJV3p0aWRueWhXNUh1VTVRREU4cWZ1c2xkK01PSHF4R3JSdU0rdmdGVVBxNFdiS\
      1ZpMFcKc1RoN2lnR3JNN2EyQWlOMDBhTjFDU2xoNGRKQXkzYnlpb0xkeDNOVmNTRnBzeEdvdElIT\
      2ZkektOeitLamtFaQpqTmdzTVVOKzdTRHhZdDZHdFdsaStWcGtMWXZFNEJ0NC9yN1hPNnd2ckpyV\
      2hTdDlxNGx2MUIrWkFGaHZXUEZxCjJoYzVFNGtWcmJBSldjWHNKZUhDNjZGQUZqajhBQkk5VWtGM\
      0RaeUZhUT09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0="

users:
- name: "conteudos"
  user:
    token: "kubeconfig-u-d77f8.c-vv8bx:xcdstkt4k6z7vshjvnhvj6t69tzhb6dvx6jfgmlc9tr54xwc2mxn29"

contexts:
- name: "conteudos"
  context:
    user: "conteudos"
    cluster: "conteudos"
```

## ConfigMap
```bash
$ kubectl -n logging --context conteudos get cm filebeat-filebeat-daemonset-config -o yaml
$ kubectl -n logging --context conteudos edit cm filebeat-filebeat-daemonset-config -o yaml
$ kubectl -n logging --context conteudos rollout restart daemonset/filebeat-filebeat
$ watch kubectl -n logging --context conteudos get pods -l app=filebeat-filebeat
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
https://www.metricfire.com/blog/kubernetes-logging-with-filebeat-and-elasticsearch-part-2/<br>