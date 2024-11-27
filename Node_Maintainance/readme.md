# Node Maintainance

Maintaining nodes in an Elasticsearch, Logstash, and Kibana (ELK) cluster is critical for ensuring smooth operations, high availability, and performance. Here's a step-by-step guide for safely performing node maintenance in an ELK cluster.
1. Pre-Maintenance Preparation

    Understand the Node's Role:
        Check if the node is a master, data, or client (coordinating) node.
        Nodes have different responsibilities, so identify the impact of taking it offline.

    Check Cluster Health:
        Run:

        curl -X GET "http://<NODE_IP>:9200/_cluster/health?pretty"

        Ensure the cluster status is green. A yellow or red status requires attention before proceeding.

    Backup Configuration:
        Backup important configuration files like elasticsearch.yml, logstash.yml, and kibana.yml.

    Inform Stakeholders:
        Notify your team or stakeholders about potential impacts during maintenance.

2. Safely Removing a Node from the Cluster

    Exclude Node from Allocation (Data Nodes Only):
        Prevent the cluster from allocating shards to the node:

    curl -X PUT "http://<NODE_IP>:9200/_cluster/settings" -H 'Content-Type: application/json' -d '{
      "transient": {
        "cluster.routing.allocation.exclude._name": "<NODE_NAME>"
      }
    }'

Wait for Rebalancing to Complete:

    Monitor shard relocation using:

    curl -X GET "http://<NODE_IP>:9200/_cluster/health?pretty"

    Ensure no shards remain on the node before proceeding.

Drain the Node (Optional):

    If Logstash is running on the node, stop ingesting data temporarily.

Stop Node Services:

    Use system commands to stop Elasticsearch, Logstash, or Kibana:

    sudo systemctl stop elasticsearch
    sudo systemctl stop logstash
    sudo systemctl stop kibana

Verify Node Removal:

    Check the cluster state to ensure the node is no longer part of it:

        curl -X GET "http://<NODE_IP>:9200/_cat/nodes?v"

3. Perform Maintenance on the Node

    Update software packages, configurations, or perform hardware repairs.
    Make sure that all maintenance steps are documented.

4. Re-Adding the Node to the Cluster

    Re-enable Allocation:
        Remove allocation exclusion rules:

    curl -X PUT "http://<NODE_IP>:9200/_cluster/settings" -H 'Content-Type: application/json' -d '{
      "transient": {
        "cluster.routing.allocation.exclude._name": ""
      }
    }'

Start Elasticsearch Service:

sudo systemctl start elasticsearch

Verify Node Reconnection:

    Confirm that the node is rejoining the cluster:

    curl -X GET "http://<NODE_IP>:9200/_cat/nodes?v"

Monitor Shard Relocation:

    Ensure shards are balanced across the cluster:

    curl -X GET "http://<NODE_IP>:9200/_cluster/health?pretty"

Start Other Services:

    Restart Logstash or Kibana if needed:

        sudo systemctl start logstash
        sudo systemctl start kibana

5. Post-Maintenance Checks

    Verify Cluster Health:
        Confirm the cluster health is back to green.

    Validate Node Functionality:
        Check logs (/var/log/elasticsearch/) for any errors.
        Verify data ingestion and query functionality.

    Re-Enable Monitoring and Alerts:
        If any monitoring or alerting was disabled, re-enable them.

    Inform Stakeholders:
        Notify stakeholders that maintenance is complete and services are operational.