[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/awwesome)](https://artifacthub.io/packages/search?repo=awwesome)

# awwesome

This repository contains a Helm chart to run [awweso.me](https://awweso.me) on Kubernetes.

Awwesome is a better ui for the [Awesome Selfhosted](https://github.com/awesome-selfhosted/awesome-selfhosted) list.

## Values

---

### general

| value            | default  | description                                       |
| ---------------- | -------- | ------------------------------------------------- |
| fullnameOverride | awwesome | Specify what to override the workload names with. |

### web (aka nginx pod)

| value                                    | default                 | description                                                                               |
| ---------------------------------------- | ----------------------- | ----------------------------------------------------------------------------------------- |
| web.label                                | empty object            | Specify what to override the workload names with.                                         |
| web.label                                | empty object            | Specify the names of the additional labels for the web workload.                          |
| web.mountPath                            | "/usr/share/nginx/html" |                                                                                           |
| web.replicas                             | 1                       | count of nginx replicas                                                                   |
| web.podAnnotations                       | empty object            | Specify the names of the additional annotations for the web pod                           |
| web.image.repository                     | nginx                   | Define which Docker repository nginx can be found in.                                     |
| web.image.tag                            | alpine                  | Define which Docker tag of nginx should be used.                                          |
| web.resources                            | empty object            | At this point, the resources of the nginx web pod can be defined.                         |
| web.nodeSelector                         | empty object            | At this point, the nodeselector of the nginx web pod can be defined.                      |
| web.affinity                             | empty object            | At this point, the affinity of the nginx web pod can be defined.                          |
| web.tolerations                          | empty object            | At this point, the tolerations of the nginx web pod can be defined.                       |
| web.ingress.enabled                      | false                   | If this value is set to true, an Ingress will be created.                                 |
| web.ingress.annotations                  | empty object            | At this point, the annotations ingress can be defined.                                    |
| web.ingress.className                    | empty string            | At this point, the ingress class to be used can be defined.                               |
| web.ingress.tls[].hosts                  | empty array of strings  | At this point, an array of hosts can be defined.                                          |
| web.ingress.tls[].secretName             | empty string            | At this point, the name can be defined for the secret in which the certificate is stored. |
| web.ingress.hosts[].host                 | empty string            | At this point, a string of the host can be defined.                                       |
| web.ingress.hosts[].paths                | empty array of strings  | At this point, an array of paths can be defined                                           |
| web.ingress.persistence.storageClassName | empty string            | At this point, the storage class can be defined                                           |
| web.ingress.persistence.size             | 10Gi                    | At this point, the storage size can be defined                                            |

### generator

The generator is needed to supply the awwesome site with values once. These values are retrieved through the GraphQL interface of Github, and therefore, a secret with a Github token must already exist in the target namespace.

> You will need a personal access token from Github to be able to use the GraphQL API. More information on this can be found under "Managing your personal access tokens".

> It is recommended to use a fine-grained personal access token without any access rights. This is possible because fine-grained personal access have access to public repos per default.

[Source](https://github.com/mkitzmann/awwesome?tab=readme-ov-file#personal-access-token)

```yaml
apiVersion: v1
data:
  github_token: <token base64encoded>
kind: Secret
metadata:
  name: awwesome
type: Opaque
```

The following values are applied to the `cronjob`, which runs repeatedly, and to the `initContainer`, which is executed once at the start of the `deployment` of the **web** workload.

| value                      | default            | description                                                                          |
| -------------------------- | ------------------ | ------------------------------------------------------------------------------------ |
| generator.label            | empty object       | Specify what to override the workload names with.                                    |
| generator.schedule         | 0 _/12 _ \* \*     | twice a day, look at [crontab.guru](https://crontab.guru) to get valid cron examples |
| generator.image.repository | mkitzmann/awwesome | Define which Docker repository the generator can be found in.                        |
| generator.image.tag        | 0.7.9              | Define which Docker tag of the generator should be used.                             |
| generator.secret.name      | awwesome           | Name of the secret containing the GitHub token.                                      |
| generator.secret.key       | awwesome           | Name of the key that has the token as its value.                                     |
| generator.secret.key       | awwesome           | Name of the key that has the token as its value.                                     |
| generator.resources        | empty object       | At this point, the resources of the generator can be defined.                        |
| generator.nodeSelector     | empty object       | At this point, the nodeselector of the generator can be defined.                     |
| generator.affinity         | empty object       | At this point, the affinity of the generator can be defined.                         |
| generator.tolerations      | empty object       | At this point, the tolerations of the generator can be defined.                      |


## Complete example with all values, but not all values are needed.

```yaml
---
# -- String to fully override names.fullname
fullnameOverride: "awwesome"

web:
  labels:
    foo: bar
  mountPath: "/usr/share/nginx/html"
  replicas: 1
  podAnnotations:
    foo: bar
  image:
    reposiory: nginx
    tag: alpine
  resources:
    limits:
      memory: 50Mi
    requests:
      cpu: 10m
      memory: 50Mi
  nodeSelector: {}
  affinity: {}
  tolerations: {}
  ingress:
    enabled: true
    annotations:
      cert-manager.io/cluster-issuer: clusterissuer-prod
    className: "nginx"
    tls:
      - hosts:
          - awwesome.example.com
        secretName: "awwesome-secret"
    hosts:
      - host: awwesome.example.com
        paths:
          - "/"
  persistence:
    storageClassName: "local-path"
    size: 10Gi

generator:
  labels:
    bar: foo
  schedule: "0 0 * * *"
  image:
    reposiory: mkitzmann/awwesome
    tag: 0.7.9
  secret:
    name: "awwesome"
    key: "github_token"
  resources:
    limits:
      memory: 400Mi
    requests:
      cpu: 15m
      memory: 400Mi
  restartPolicy: "OnFailure"
  nodeSelector: {}
  affinity: {}
  tolerations: {}
```
