# Ansible Role: ansible_elasticsearch_docker

=========

This Ansible role installs and configures Elasticsearch 9.x+ inside Docker containers using either single-node (docker_container) or multi-node cluster (docker compose) setup — without Kibana.

Features

⸻

- Deploys Elasticsearch from official Docker images (elasticsearch or elasticsearch-wolfi)
- Supports single-node mode (standalone container with discovery.type=single-node)
- Supports multi-node cluster mode (3+ nodes via docker compose, secure by default with TLS & password)
- Automatically creates Docker network and persistent data volumes
- Handles initial security (bootstrap password, CA certificates)
- Configurable JVM heap size and container memory
- Optionally generates hardened containers via Wolfi images

Requirements
Target host must have:

- Docker >= 24
- Docker Compose plugin (v2+)
- Python 3 and Ansible >= 2.15 on the control node
- Collection community.docker
- At least 4 GB RAM for multi-node cluster (per Elastic recommendation)

Role Variables

You can override the following defaults in defaults/main.yml:

```yaml
# Image and version
es_version: "9.1.3"
es_image_repo: "docker.elastic.co/elasticsearch/elasticsearch"
es_use_wolfi: false

# Mode: "single" or "multi"
es_mode: "single"

# Common settings
es_network: "elastic"
es_data_root: "/var/lib/elasticsearch"
es_container_memory: "2g"
es_bind_host: "0.0.0.0"

# Security
es_enable_security: true
es_elastic_password: "ChangeMe123"

# Single-node
es_container_name: "es01"
es_http_port: 9200
es_heap: "1g"
es_single_extra_env: {}

# Multi-node
es_cluster_name: "es-docker"
es_nodes:
  - name: es01
    http_port: 9200
    roles: ["master","data","ingest","ml","transform"]
    heap: "1g"
  - name: es02
    roles: ["master","data","ingest"]
    heap: "1g"
  - name: es03
    roles: ["master","data","ingest"]
    heap: "1g"
```

Dependencies

This role requires the community.docker Ansible collection.
Install it with:

```bash
ansible-galaxy collection install community.docker
```

Example Playbooks

Single-node Elasticsearch

```yaml
- name: Deploy single-node Elasticsearch
  hosts: es_host
  become: yes
  roles:
    - role: es_docker
      es_mode: single
      es_elastic_password: "StrongAlphaNum123"
      es_http_port: 9200
      es_heap: "1g"
```

Verify:

```bash
curl --cacert /var/lib/elasticsearch/http_ca.crt -u elastic:StrongAlphaNum123 https://localhost:9200
```

⸻

Multi-node Elasticsearch cluster

```yaml
- name: Deploy Elasticsearch cluster (3 nodes)
  hosts: es_host
  become: yes
  roles:
    - role: es_docker
      es_mode: multi
      es_elastic_password: "StrongAlphaNum123"
      es_nodes:
        - { name: es01, http_port: 9200, heap: "2g", roles: ["master","data","ingest"] }
        - { name: es02, heap: "2g", roles: ["master","data","ingest"] }
        - { name: es03, heap: "2g", roles: ["master","data","ingest"] }
```

Verify cluster health:

```bash
curl --cacert /opt/es-compose/certs/http_ca.crt -u elastic:StrongAlphaNum123 https://localhost:9200/_cluster/health
curl --cacert /opt/es-compose/certs/http_ca.crt -u elastic:StrongAlphaNum123 https://localhost:9200/_cat/nodes
```

License

Apache 2.0

Author Information

Morgan Miller
GitHub
Rework by [Pavel Xolog](https://github.com/Xolog)
