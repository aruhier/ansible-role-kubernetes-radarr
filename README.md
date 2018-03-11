Ansible Role: Radarr for Kubernetes
===================================

Ansible role to install [Radarr](https://radarr.video/) on Kubernetes.

Role Variables
--------------

```yaml
# Image used
kubernetes_radarr_image: "linuxserver/radarr:latest"

# Namespace
kubernetes_radarr_namespace: "default"
# App name (used as selector)
kubernetes_radarr_app: "radarr"
# Deployment name
kubernetes_radarr_deployment: "radarr-deployment"
# Service name
kubernetes_radarr_service: "radarr"

# Number of replicas
kubernetes_radarr_replicas: 1
kubernetes_radarr_revision_history: 1

# Node selector
kubernetes_radarr_node_selector: {}

# Add custom labels in the deployment metadata section
kubernetes_radarr_deployment_labels: {}
# Add custom annotations in the deployment metadata section
kubernetes_radarr_deployment_annotations: {}

kubernetes_radarr_resources:
  limits:
    memory: "1Gi"
  requests:
    memory: "256Mi"

# Setup ingress for radarr
kubernetes_radarr_setup_ingress: false
kubernetes_radarr_ingress:
  name: "radarr-ingress"
  host: "radarr.example.com"
  annotations:
  tls:

### Volumes ###

# For each volume, `definition` and `subPath` can be used. Their content is
# exactly what would be in a Kkubernetes pod manifest.

# Downloads volumes. Mounted in /downloads/ (see examples for more details)
kubernetes_radarr_downloads_volumes: {}
# Movies volumes. Mounted in /movies/ (see examples for more details)
kubernetes_radarr_movies_volumes: {}
# Watch directories. Useful when blackhole is used. Mounted in /watchdirs/ (see
# examples for more details)
kubernetes_radarr_watchdirs_volumes: {}

# radarr config volume. Contains the database, arts cache and config.
kubernetes_radarr_config_volume:
  definition:
```

Dependencies
------------

Kubectl needs to be installed on the host targeted by the role.


Example Playbook
----------------

```yaml

- hosts: kube-master
  run_once: true
  vars:
    kubernetes_radarr_downloads_volumes:
      # This volume will be mounted as /downloads/completed
      completed:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-torrents
            readOnly: false
        subPath: "completed"

    kubernetes_radarr_movies_volumes:
      # This volume will be mounted as /movies/global
      global:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-movies
            readOnly: false

    # Directories watched by our torrents client
    kubernetes_radarr_watchdirs_volumes:
      # This volume will be mounted as /watchdirs/providers
      providers:
        definition:
          glusterfs:
            endpoints: gluster-example-cluster
            path: volume-watchdirs
            readOnly: false
        subPath: "example/providers"

    kubernetes_radarr_config_volume:
      definition:
        glusterfs:
          endpoints: gluster-example-cluster
          path: volume-radarr
          readOnly: false
        subPath: "config"

    kubernetes_radarr_setup_ingress: true
    kubernetes_radarr_ingress:
      name: "radarr-ingress"
      host: "radarr.example.com"
      annotations:
        kubernetes.io/tls-acme: "true"
      tls:
        - secretName: "radarr-ingress-tls"
          hosts:
            - "radarr.example.com"
  roles:
    - role: Anthony25.kubernetes-radarr
```

Use `run_once` to run the role on only one available master in the cluster.
If the radarr web interface is not reachable, please check that it is
listening on `0.0.0.0:7878`, and not only `localhost:7878` as the default is.

License
-------

Tool under the BSD license. Do not hesitate to report bugs, ask me some
questions or do some pull request if you want to!
