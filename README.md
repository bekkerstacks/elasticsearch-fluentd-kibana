# elasticsearch-fluentd-kibana
Elasticsearch FluentD and Kibana

## Usage

```
$ docker network create --driver overlay private
```

Deploy the elasticsearch-fluentd-kibana stack:

```
$ docker stack deploy -c docker-compose-eflk.yml efk
Creating config efk_fluent-elasticsearch-conf.v1
Creating service efk_fluentd-elasticsearch
Creating service efk_kibana
Creating service efk_elasticsearch
```

Deploy the sample application that logs info to stdout every 5 seconds:

```
$ docker stack deploy -c docker-compose-eflk-app.yml efk
Creating service efk_demo-logger
```

View the elasticsearch nodes:

```
$ curl http://localhost:9200/_cat/nodes?v
ip       heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
10.0.0.7           34          95   7    0.42    0.43     0.26 mdi       *      es-node.1.docker-desktop
```

View the elasticsearch indices:

```
$ curl http://localhost:9200/_cat/indices?v
health status index              uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .kibana_1          Kxq95NMkR-6m9AMWWfzEvg   1   0          0            0       230b           230b
yellow open   fluentd-2019.10.09 fnxKL5I2Ty6vneP1bLWnhA   5   1          3            0       21kb           21kb
```

View the documents from the fluentd index:

```
$ > curl http://localhost:9200/fluentd-2019.10.09/_search?pretty
{
  "took" : 108,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 6,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "fluentd-2019.10.09",
        "_type" : "access_log",
        "_id" : "CToQsW0BdVrjXZ_xF5Hd",
        "_score" : 1.0,
        "_source" : {
          "source" : "stdout",
          "log" : "The date is: 1570633394",
          "container_id" : "de7883816f60d768f0c743c39eeb351c67a8eeefde867f3d013b628a1b1cd575",
          "container_name" : "/efk_demo-logger.1.tt85yi5sv2yosj9w1n5jaaqoj",
          "hostname" : "0ce8d311101c",
          "tag" : "docker.efk.demo-logger",
          "stack_name" : "efk",
          "service_name" : "demo-logger",
          "fluentd_hostname" : "docker-desktop",
          "@timestamp" : "2019-10-09T15:05:19.000000000+00:00",
          "@log_name" : "docker.efk.demo-logger"
        }
      },
...
}
```

Head over to kibana on `http://localhost:5601/`, select "Management" and "Index Patterns", provide the pattern "fluentd-" as shown below:

<img width="1229" alt="image" src="https://user-images.githubusercontent.com/30043398/66495502-99b2ec00-eab9-11e9-83de-bff4ef22d790.png">

Then select "@timestamp" as the "Time Filter field name" and click "Create Index Pattern"

Click on "Discover" and you will see the log events as shown below:

<img width="1231" alt="image" src="https://user-images.githubusercontent.com/30043398/66495659-e5fe2c00-eab9-11e9-91b9-94a4f6a9b634.png">

