---
layout: default
permalink: /user-guide/
title: User Guide
redirect_from:
  - /docs/user-guide.md/
  - /docs/user-guide/
---

# User Guide

* TOC
{:toc}

Kompose has support for two providers: OpenShift and Kubernetes.
You can choose a targeted provider using global option `--provider`. If no provider is specified, Kubernetes is set by default.

## Kompose Convert

Kompose supports conversion of V1, V2, and V3 Docker Compose files into Kubernetes and OpenShift objects.

### Kubernetes

```sh
$ kompose --file docker-voting.yml convert
WARN Unsupported key networks - ignoring
WARN Unsupported key build - ignoring
INFO Kubernetes file "worker-svc.yaml" created
INFO Kubernetes file "db-svc.yaml" created
INFO Kubernetes file "redis-svc.yaml" created
INFO Kubernetes file "result-svc.yaml" created
INFO Kubernetes file "vote-svc.yaml" created
INFO Kubernetes file "redis-deployment.yaml" created
INFO Kubernetes file "result-deployment.yaml" created
INFO Kubernetes file "vote-deployment.yaml" created
INFO Kubernetes file "worker-deployment.yaml" created
INFO Kubernetes file "db-deployment.yaml" created

$ ls
db-deployment.yaml  docker-compose.yml         docker-gitlab.yml  redis-deployment.yaml  result-deployment.yaml  vote-deployment.yaml  worker-deployment.yaml
db-svc.yaml         docker-voting.yml          redis-svc.yaml     result-svc.yaml        vote-svc.yaml           worker-svc.yaml
```

You can also provide multiple docker-compose files at the same time:

```sh
$ kompose -f docker-compose.yml -f docker-guestbook.yml convert
INFO Kubernetes file "frontend-service.yaml" created
INFO Kubernetes file "mlbparks-service.yaml" created
INFO Kubernetes file "mongodb-service.yaml" created
INFO Kubernetes file "redis-master-service.yaml" created
INFO Kubernetes file "redis-replica-service.yaml" created
INFO Kubernetes file "frontend-deployment.yaml" created
INFO Kubernetes file "mlbparks-deployment.yaml" created
INFO Kubernetes file "mongodb-deployment.yaml" created
INFO Kubernetes file "mongodb-claim0-persistentvolumeclaim.yaml" created
INFO Kubernetes file "redis-master-deployment.yaml" created
INFO Kubernetes file "redis-replica-deployment.yaml" created

$ ls
mlbparks-deployment.yaml  mongodb-service.yaml                       redis-replica-service.jsonmlbparks-service.yaml
frontend-deployment.yaml  mongodb-claim0-persistentvolumeclaim.yaml  redis-master-service.yaml
frontend-service.yaml     mongodb-deployment.yaml                    redis-replica-deployment.yaml
redis-master-deployment.yaml
```

When multiple docker-compose files are provided the configuration is merged. Any configuration that is common will be over ridden by subsequent file.

You can provide your docker-compose files via environment variables as following:
```sh
$ COMPOSE_FILE="docker-compose.yaml alternative-docker-compose.yaml" kompose convert
```

### OpenShift

```sh
$ kompose --provider openshift --file docker-voting.yml convert
WARN [worker] Service cannot be created because of missing port.
INFO OpenShift file "vote-service.yaml" created
INFO OpenShift file "db-service.yaml" created
INFO OpenShift file "redis-service.yaml" created
INFO OpenShift file "result-service.yaml" created
INFO OpenShift file "vote-deploymentconfig.yaml" created
INFO OpenShift file "vote-imagestream.yaml" created
INFO OpenShift file "worker-deploymentconfig.yaml" created
INFO OpenShift file "worker-imagestream.yaml" created
INFO OpenShift file "db-deploymentconfig.yaml" created
INFO OpenShift file "db-imagestream.yaml" created
INFO OpenShift file "redis-deploymentconfig.yaml" created
INFO OpenShift file "redis-imagestream.yaml" created
INFO OpenShift file "result-deploymentconfig.yaml" created
INFO OpenShift file "result-imagestream.yaml" created
```

It also supports creating buildconfig for build directive in a service. By default, it uses the remote repository for the current git branch as the source repository, and the current branch as the source branch for the build. You can specify a different source repository and branch using `--build-repo` and `--build-branch` options respectively.

```sh
$ kompose --provider openshift --file buildconfig/docker-compose.yml convert
WARN [foo] Service cannot be created because of missing port.
INFO OpenShift Buildconfig using git@github.com:rtnpro/kompose.git::master as source.
INFO OpenShift file "foo-deploymentconfig.yaml" created
INFO OpenShift file "foo-imagestream.yaml" created
INFO OpenShift file "foo-buildconfig.yaml" created
```

**Note**: If you are manually pushing the Openshift artifacts using `oc create -f`, you need to ensure that you push the imagestream artifact before the buildconfig artifact, to work around this Openshift issue: https://github.com/openshift/origin/issues/4518 .

## Alternative Conversions

The default `kompose` transformation will generate Kubernetes [Deployments](http://kubernetes.io/docs/user-guide/deployments/) and [Services](http://kubernetes.io/docs/user-guide/services/), in yaml format. You have alternative option to generate json with `-j`. Also, you can alternatively generate [Replication Controllers](http://kubernetes.io/docs/user-guide/replication-controller/) objects, [Daemon Sets](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/), [Statefulset](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) or [Helm](https://github.com/helm/helm) charts.

```sh
$ kompose convert -j
INFO Kubernetes file "redis-svc.json" created
INFO Kubernetes file "web-svc.json" created
INFO Kubernetes file "redis-deployment.json" created
INFO Kubernetes file "web-deployment.json" created
```

The `*-deployment.json` files contain the Deployment objects.

```sh
$ kompose convert --controller replicationController
INFO Kubernetes file "redis-svc.yaml" created
INFO Kubernetes file "web-svc.yaml" created
INFO Kubernetes file "redis-replicationcontroller.yaml" created
INFO Kubernetes file "web-replicationcontroller.yaml" created
```

The `*-replicationcontroller.yaml` files contain the Replication Controller objects. If you want to specify replicas (default is 1), use `--replicas` flag: `$ kompose convert --controller replicationController --replicas 3`

```sh
$ kompose convert --controller daemonSet
INFO Kubernetes file "redis-svc.yaml" created
INFO Kubernetes file "web-svc.yaml" created
INFO Kubernetes file "redis-daemonset.yaml" created
INFO Kubernetes file "web-daemonset.yaml" created
```

The `*-daemonset.yaml` files contain the Daemon Set objects

```sh
$ kompose convert --controller statefulset
INFO Kubernetes file "db-service.yaml" created
INFO Kubernetes file "wordpress-service.yaml" created
INFO Kubernetes file "db-statefulset.yaml" created
INFO Kubernetes file "wordpress-statefulset.yaml" created
```

The `*statefulset-.yaml` files contain the Statefulset objects.

If you want to generate a Chart to be used with [Helm](https://github.com/kubernetes/helm) simply do:

```sh
$ kompose convert -c
INFO Kubernetes file "web-svc.yaml" created
INFO Kubernetes file "redis-svc.yaml" created
INFO Kubernetes file "web-deployment.yaml" created
INFO Kubernetes file "redis-deployment.yaml" created
chart created in "./docker-compose/"

$ tree docker-compose/
docker-compose
├── Chart.yaml
├── README.md
└── templates
    ├── redis-deployment.yaml
    ├── redis-svc.yaml
    ├── web-deployment.yaml
    └── web-svc.yaml
```

The chart structure is aimed at providing a skeleton for building your Helm charts. It's compatible with both Helm V2 and Helm V3.

## Labels

`kompose` supports Kompose-specific labels within the `docker-compose.yml` file to
explicitly define the generated resources' behavior upon conversion, like Service, PersistentVolumeClaim...

The currently supported options are:

| Key                                                 | Value                                                                                |
| --------------------------------------------------- | ------------------------------------------------------------------------------------ |
| kompose.service.type                                | nodeport / clusterip / loadbalancer / headless                                       |
| kompose.service.group                               | name to group the containers contained in a single pod                               |
| kompose.service.expose                              | true / hostnames (separated by comma)                                                |
| kompose.service.nodeport.port                       | port value (string)                                                                  |
| kompose.service.expose.tls-secret                   | secret name                                                                          |
| kompose.service.expose.ingress-class-name           | ingress class name                                                                   |
| kompose.volume.size                                 | kubernetes supported volume size                                                     |
| kompose.volume.storage-class-name                   | kubernetes supported volume storageClassName                                         |
| kompose.volume.type                                 | use k8s volume type, eg "configMap", "persistentVolumeClaim", "emptyDir", "hostPath" |
| kompose.controller.type                             | deployment / daemonset / replicationcontroller / statefulset                         |
| kompose.image-pull-policy                           | kubernetes pods imagePullPolicy                                                      |
| kompose.image-pull-secret                           | kubernetes secret name for imagePullSecrets                                          |
| kompose.service.healthcheck.readiness.disable       | kubernetes readiness disable                                                         |
| kompose.service.healthcheck.readiness.test          | kubernetes readiness exec command                                                    |
| kompose.service.healthcheck.readiness.http_get_path | kubernetes readiness httpGet path                                                    |
| kompose.service.healthcheck.readiness.http_get_port | kubernetes readiness httpGet port                                                    |
| kompose.service.healthcheck.readiness.tcp_port      | kubernetes readiness tcpSocket port                                                  |
| kompose.service.healthcheck.readiness.interval      | kubernetes readiness interval value                                                  |
| kompose.service.healthcheck.readiness.timeout       | kubernetes readiness timeout value                                                   |
| kompose.service.healthcheck.readiness.retries       | kubernetes readiness retries value                                                   |
| kompose.service.healthcheck.readiness.start_period  | kubernetes readiness start_period                                                    |
| kompose.service.healthcheck.liveness.http_get_path  | kubernetes liveness httpGet path                                                     |
| kompose.service.healthcheck.liveness.http_get_port  | kubernetes liveness httpGet port                                                     |
| kompose.service.healthcheck.liveness.tcp_port       | kubernetes liveness tcpSocket port                                                   |
| kompose.service.external-traffic-policy       | 'cluster', 'local', ''                                                   |                                                |
| kompose.security-context.fsgroup       |  kubernetes pod security group fsgroup                                                  |                                                |
| kompose.volume.sub-path                        | kubernetes volume mount subpath                                                   |                                                |

**Note**: `kompose.service.type` label should be defined with `ports` only (except for headless service), otherwise `kompose` will fail.

- `kompose.service.type` defines the type of service to be created.

For example:

```yaml
version: "2"
services:
  nginx:
    image: nginx
    dockerfile: foobar
    build: ./foobar
    cap_add:
      - ALL
    container_name: foobar
    labels:
      kompose.service.type: nodeport
```

- `kompose.service.group` defines the group of containers included in a single pod.

For example:

```yaml
version: "3"

services:
  nginx:
    image: nginx
    depends_on:
      - logs
    labels:
      - kompose.service.group=sidecar

  logs:
    image: busybox
    command: ["tail -f /var/log/nginx/access.log"]
    labels:
      - kompose.service.group=sidecar
```

- `kompose.service.expose` defines if the service needs to be made accessible from outside the cluster or not. If the value is set to "true", the provider sets the endpoint automatically, and for any other value, the value is set as the hostname. If multiple ports are defined in a service, the first one is chosen to be the exposed.
  - For the Kubernetes provider, an ingress resource is created, and it is assumed that an ingress controller has already been configured. If the value is set to a comma-separated list, multiple hostnames are supported. Hostname with the path is also supported.
  - For the OpenShift provider, a route is created.
- `kompose.service.nodeport.port` defines the port value when service type is `nodeport`, this label should only be set when the service only contains 1 port. Usually kubernetes define a port range for node port values, kompose will not validate this.
- `kompose.service.expose.tls-secret` provides the name of the TLS secret to use with the Kubernetes ingress controller. This requires kompose.service.expose to be set.
- `kompose.service.expose.ingress-class-name` provides the name of ingress class to use with the Kubernetes ingress controller. This requires kompose.service.

For example:

```yaml
version: "2"
services:
  web:
    image: tuna/docker-counter23
    ports:
     - "5000:5000"
    links:
     - redis
    labels:
      kompose.service.expose: "counter.example.com,foobar.example.com"
      kompose.service.expose.tls-secret: "example-secret"
      kompose.service.expose.ingress-class-name: "nginx"
  redis:
    image: redis:3.0
    ports:
     - "6379"
```

- `kompose.serviceaccount-name` defines the service account name to provide the credential info of the pod.

For example:

```yaml
version: '3.4'
services:
  app:
    image: python
    labels:
      kompose.serviceaccount-name: "my-service"
```

- `kompose.image-pull-secret` defines a kubernetes secret name for imagePullSecrets podspec field.
  This secret will be used for pulling private images.
  For example:

```yaml
version: '2'
services:
  tm-service:
    image: premium/private-image
    labels:
      kompose.image-pull-secret: "example-kubernetes-secret"
```

- `kompose.volume.size` defines the requests storage's size in the PersistentVolumeClaim, or you can use the command line parameter `--pvc-request-size`.
- The priority follows label (kompose.volume.size) > command parameter(--pvc-request-size) > defaultSize (100Mi)

For example:

```yaml
version: '2'
services:
  db:
    image: postgres:10.1
    labels:
      kompose.volume.size: 1Gi
    volumes:
      - db-data:/var/lib/postgresql/data
```

- `kompose.volume.storage-class-name` defines the requests storage's class name in the PersistentVolumeClaim.

For example:

```yaml
version: '3'
services:
  db:
    image: postgres:10.1
    labels:
      kompose.volume.storage-class-name: custom-storage-class-name
    volumes:
      - db-data:/var/lib/postgresql/data
```

- `kompose.controller.type` defines which controller type should convert for this service

For example:

```
web:
  image: wordpress:4.5
  ports:
    - '80'
  environment:
    WORDPRESS_AUTH_KEY: changeme
    WORDPRESS_SECURE_AUTH_KEY: changeme
    WORDPRESS_LOGGED_IN_KEY: changeme
    WORDPRESS_NONCE_KEY: changeme
    WORDPRESS_AUTH_SALT: changeme
    WORDPRESS_SECURE_AUTH_SALT: changeme
    WORDPRESS_LOGGED_IN_SALT: changeme
    WORDPRESS_NONCE_SALT: changeme
    WORDPRESS_NONCE_AA: changeme
  restart: always
  links:
    - 'db:mysql'
db:
  image: mysql:5.7
  environment:
    MYSQL_ROOT_PASSWORD: password
  restart: always
  labels:
    project.logs: /var/log/mysql
    kompose.controller.type: daemonset
```

Service `web` will be converted to `Deployment` as default, service `db` will be converted to `DaemonSet` because of `kompose.controller.type` label.

- `kompose.image-pull-policy` defines Kubernetes PodSpec imagePullPolicy. One of Always, Never, IfNotPresent. Defaults to Always if :latest tag is specified, or IfNotPresent otherwise.

For example:

```yaml
version: '2'
services:
  example-service:
    image: example-image
    labels:
      kompose.image-pull-policy: "Never"
```

For example:

```yaml
version: '2'
services:
  example-service:
    image: example-image
    labels:
      kompose.service.healthcheck.liveness.http_get_path: /health/ping
      kompose.service.healthcheck.liveness.http_get_port: 8080
    healthcheck:
      interval: 10s
      timeout: 10s
      retries: 3
      start_period: 30s
```

- `kompose.service.healthcheck.liveness` defines Kubernetes [liveness HttpRequest](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-a-liveness-http-request), If you use healthcheck without liveness labels, have to define `test` in healcheck it's work to Kubernetes [liveness command](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-readiness-probes)

For example:

```yaml
version: '2'
services:
  example-service:
    image: example-image
    labels:
      kompose.service.healthcheck.readiness.test: CMD curl -f "http://localhost:8080/health/ping"
      kompose.service.healthcheck.readiness.interval: 10s
      kompose.service.healthcheck.readiness.timeout: 10s
      kompose.service.healthcheck.readiness.retries: 3
      kompose.service.healthcheck.readiness.start_period: 30s
```

- `kompose.service.healthcheck.readiness` defines Kubernetes [readiness](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-readiness-probes)

- `kompose.service.external-traffic-policy` defines Kubernetes Service [external traffic policy.](https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/#preserving-the-client-source-ip).

For example:

```yaml
version: "3.3"

services:
  front-end:
    image: gcr.io/google-samples/gb-frontend:v4
    environment:
      - GET_HOSTS_FROM=dns
    ports:
      - 80:80
    labels:
      kompose.service.expose: lb
      kompose.service.external-traffic-policy: local
      kompose.service.type: loadbalancer
```

- `kompose.security-context.fsgroup` defines Kubernetes Pod [security context FsGroup.](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/).

For example:

```yaml
version: '3.8'

services:
  pgadmin:
    image: nginx
    labels:
      kompose.security-context.fsgroup: 1001
```

- `kompose.volume.sub-path` defines Kubernetes Container [VolumeMounts Subpath](https://kubernetes.io/docs/concepts/storage/volumes/#using-subpath).

For example:

```yaml
version: '3.8'

services:
  pgadmin:
    image: postgres
    labels:
      kompose.volume.sub-path: pg-data
```
## Restart

If you want to create normal pods without controller you can use `restart` construct of docker-compose to define that. Follow table below to see what happens on the `restart` value.

| `docker-compose` `restart` | object created    | Pod `restartPolicy` |
| -------------------------- | ----------------- | ------------------- |
| `""`                       | controller object | `Always`            |
| `always`                   | controller object | `Always`            |
| `unless-stopped`           | controller object | `Always`            |
| `on-failure`               | Pod               | `OnFailure`         |
| `no`                       | Pod               | `Never`             |

**Note**: controller object could be `deployment` or `replicationcontroller`, etc.

For e.g. `pival` service will become pod down here. This container calculated value of `pi`.

```yaml
version: '2'

services:
  pival:
    image: perl
    command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
    restart: "on-failure"
```

#### Warning about Deployment Config's

If the Docker Compose file has a volume specified for a service, the Deployment (Kubernetes) or DeploymentConfig (OpenShift) strategy is changed to "Recreate" instead of "RollingUpdate" (default). This is done to avoid multiple instances of a service from accessing a volume at the same time.

If the Docker Compose file has service name with `_` or `.` in it (eg.`web_service` or `web.service`), then it will be replaced by `-` and the service name will be renamed accordingly (eg.`web-service`). Kompose does this because "Kubernetes" doesn't allow `_` in object name.

Please note that changing service name might break some `docker-compose` files.

## Network policies generation
[Network policies](https://kubernetes.io/docs/concepts/services-networking/network-policies) are not generated by default, because it's not mandatory to deploy your application. However, it's one of the best practices when it comes to deploy secure applications on top of Kubernetes.
To generate network policies, all you need is to use the `--generate-network-policies` flag.

## Build and push image

If the Docker Compose file has `build` or `build:context, build:dockerfile` keys, build will run when `--build` specified.

And Image will push to _docker.io_ (default) when `--push-image=true` specified.

It is possible to push to custom registry by specify `--push-image-registry`, which will override the registry from image name.


### Authentication on registry

Kompose uses the docker authentication from file `$DOCKER_CONFIG/config.json`, `$HOME/.docker/config.json`, and `$HOME/.dockercfg` after `docker login`.

**This only works fine on Linux but macOS would fail when using `"credsStore": "osxkeychain"`.**

However, there is an approach to push successfully on macOS, by not using `osxkeychain` for `credsStore`. To disable `osxkeychain`:

- remove `credsStore` from `config.json` file, and `docker login` again.
- for some docker desktop versions, there is a setting `Securely store Docker logins in macOS keychain`, which should be unchecked. Then restart docker desktop if needed, and `docker login` again.

Now `config.json` should contain base64 encoded passwords, then push image should succeed. Working, but not safe though! Use it at your risk.

For Windows, there is also `credsStore` which is `wincred`. Technically it will fail on authentication as macOS does, but you can try the approach above like macOS too.

### Custom Build and Push
If you want to customize the build and push processes and use another containers solution than Docker,
Kompose offers you the possibility to do that. You can use `--build-command` and `--push-command` flags
to achieve that.

e.g: `kompose -f convert --build-command 'whatever command --you-use' --push-command 'whatever command --you-use'`
