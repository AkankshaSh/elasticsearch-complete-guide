# elasticsearch-complete-guide

Elasticsearch: V7.9.0


In this project we will be using three Elasticsearch nodes in one Elasticsearch cluster. You can query Elasticsearch via Kibana's Dev Management tool.

1. Make sure Docker Engine is allotted at least 4GiB of memory. In Docker Desktop, you configure resource usage on the Advanced tab in Preference (macOS) or Settings (Windows).

2. Run docker-compose to bring up the cluster:
  docker-compose up
  
  
3. Submit a _cat/nodes request to see that the nodes are up and running:
curl -X GET "localhost:9200/_cat/nodes?v&pretty"


References
https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html



