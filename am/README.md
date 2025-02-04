# Gravitee.io Access Management Helm Chart

**Chart supported versions: 2.10.x and higher**

## Components

This chart will deploy the following:

- Gravitee UI
- Gravitee API
- Gravitee Access Gateway
- MongoDB replica-set (optional dependency)

## Installing

* Add the Gravitee.io helm charts repo
  ```
  $ helm repo add graviteeio https://helm.gravitee.io
  ```
* Install it
  ```
  $ helm install --name graviteeio-am graviteeio/am

**Note:** If you're using Helm 3, the name parameter is no more valid. Please check https://helm.sh/docs/faq/#release-names-are-now-scoped-to-the-namespace

## Create a chart archive

To package this chart directory into a chart archive, run:

```
$ helm package .
```

## Installing the Chart

To install the chart from the Helm repository with the release name `graviteeio-am`:

```bash
$ helm install --name graviteeio-am graviteeio/am
```

To install the chart using the chart archive, run:

```
$ helm install am-1.0.0.tgz
```

## Configuration

The following tables list the configurable parameters of the Gravitee chart and their default values.

You can rely on kubernetes _ConfigMaps_ and _Secrets_ to initialize Gravitee settings since AM 3.15.0.
To use this feature, you have to create the ServiceAccount that allows AM to connect to the Kubernetes API (the helm chart should do it by default) and then you simply have to define your application settings like this:

* for a Secret : `kubernetes://<namespace>/secrets/<my-secret-name>/<my-secret-key>`
* for a ConfigMap : `kubernetes://<namespace>/configmaps/<my-configmap-name>/<my-configmap-key>`


Here is an example for the mongodb uri initialized from the `mongo` secret deployed in the `default` namespace:

```yaml
mongo:
  uri: kubernetes://default/secrets/mongo/mongouri
```


### Shared configuration

To configure common features such as:

- chaos testing (see
  [chaoskube](https://github.com/kubernetes/charts/tree/master/stable/chaoskube)
  chart)
- configuration database (see
  [mongodb-replicaset](https://github.com/kubernetes/charts/tree/master/stable/mongodb-replicaset)

| Parameter              | Description        | Default |
| ---------------------- | ------------------ | ------- |
| `chaos.enabled`        | Enable Chaos test  | false   |


### Mongo

#### MongoDB connections

There are three ways to configure MongoDB connections.

The most simple is to provide the [MongoDB URI](https://docs.mongodb.com/manual/reference/connection-string/).

| Parameter             | Description                                | Default                    |
| --------------------- | ------------------------------------------ | -------------------------- |
| `mongo.uri`           | Mongo URI                                  | `null`                     |

If no `mongo.uri` is provided, you can provide a `mongo.servers` raw definition in combination with `mongo.dbname`, plus
eventual authentication configuration:

```yaml
mongo:
  servers: |
    - host: mongo1
      port: 27017
    - host: mongo2
      port: 27017
  dbname: gravitee
  auth:
    enabled: false
    username: 
    password:
``` 

If neither `mongo.uri` or `mongo.servers` are provided, you have to define the following configuration options:

| Parameter             | Description                                | Default                    |
| --------------------- | ------------------------------------------ | -------------------------- |
| `mongo.rsEnabled`     | Whether Mongo replicaset is enabled or not | `true`                     |
| `mongo.rs`            | Mongo replicaset name                      | `rs0`                      |
| `mongo.dbhost`        | Mongo host address                         | `mongo-mongodb-replicaset` |
| `mongo.dbport`        | Mongo host port                            | `27017`                    |
| `mongo.dbname`        | Mongo DB name                              | `gravitee`                 |
| `mongo.auth.enabled`  | Enable Mongo DB authentication             | `false`                    |
| `mongo.auth.username` | Mongo DB username                          | `null`                     |
| `mongo.auth.password` | Mongo DB password                          | `null`                     |

#### Other keys

| Parameter               | Description                      | Default |
| ----------------------- | -------------------------------- | ------- |
| `mongo.sslEnabled`      | Enable SSL connection to MongoDB | `false` |
| `mongo.socketKeepAlive` | Enable keep alive for socket     | `false` |

### Mongo ReplicaSet

| Parameter                    | Description                           | Default |
| ---------------------------- | ------------------------------------- | ------- |
| `mongodb-replicaset.enabled` | Enable deployment of Mongo replicaset | `false` |

See [MongoDB replicaset](https://artifacthub.io/packages/helm/bitnami/mongodb) for detailed documentation on helm chart.

** Please be aware that the mongodb-replicaset installed by Gravitee is NOT recommended in production and it is just for testing purpose and running AM locally.


### Gravitee.io Configuration

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| api.autoscaling.enabled | bool | `true` |  |
| api.autoscaling.maxReplicas | int | `3` |  |
| api.autoscaling.minReplicas | int | `1` |  |
| api.autoscaling.targetAverageUtilization | int | `50` |  |
| api.autoscaling.targetMemoryAverageUtilization | int | `80` |  |
| api.enabled | bool | `true` |  |
| api.http.services.core.http.authentication.password | string | `"adminadmin"` |  |
| api.http.services.core.http.host | string | `"localhost"` |  |
| api.http.services.core.http.port | int | `18093` |  |
| api.http.services.core.ingress.enabled | bool | `false` |  |
| api.http.services.core.service.enabled | bool | `false` |  |
| api.image.pullPolicy | string | `"Always"` |  |
| api.image.repository | string | `"graviteeio/am-management-api"` |  |
| api.ingress.annotations."ingress.kubernetes.io/configuration-snippet" | string | `"etag on;\nproxy_pass_header ETag;\nproxy_set_header if-match \"\";\n"` |  |
| api.ingress.annotations."kubernetes.io/ingress.class" | string | `"nginx"` |  |
| api.ingress.enabled | bool | `true` |  |
| api.ingress.hosts[0].host | string | `"am.example.com"` |  |
| api.ingress.hosts[0].paths[0] | string | `"/management"` |  |
| api.ingress.hosts[0].paths[1] | string | `"/admin"` |  |
| api.ingress.path | string | `"/management"` |  |
| api.ingress.tls[0].hosts[0] | string | `"am.example.com"` |  |
| api.ingress.tls[0].secretName | string | `"api-custom-cert"` |  |
| api.jwt.secret | string | `"s3cR3t4grAv1t3310AMS1g1ingDftK3y"` |  |
| api.logging.debug | bool | `false` |  |
| api.logging.file.enabled | bool | `true` |  |
| api.logging.file.encoderPattern | string | `"%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n%n"` |  |
| api.logging.file.rollingPolicy | string | `"\u003crollingPolicy class=\"ch.qos.logback.core.rolling.TimeBasedRollingPolicy\"\u003e\n    \u003c!-- daily rollover --\u003e\n    \u003cfileNamePattern\u003e${gravitee.management.log.dir}/gravitee_%d{yyyy-MM-dd}.log\u003c/fileNamePattern\u003e\n    \u003c!-- keep 30 days' worth of history --\u003e\n    \u003cmaxHistory\u003e30\u003c/maxHistory\u003e\n\u003c/rollingPolicy\u003e\n"` |  |
| api.logging.graviteeLevel | string | `"DEBUG"` |  |
| api.logging.jettyLevel | string | `"INFO"` |  |
| api.logging.stdout.encoderPattern | string | `"%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n"` |  |
| api.name | string | `"management-api"` |  |
| api.reloadOnConfigChange | bool | `true` |  |
| api.replicaCount | int | `1` |  |
| api.resources.limits.cpu | string | `"500m"` |  |
| api.resources.limits.memory | string | `"1024Mi"` |  |
| api.resources.requests.cpu | string | `"200m"` |  |
| api.resources.requests.memory | string | `"512Mi"` |  |
| api.restartPolicy | string | `"OnFailure"` |  |
| api.service.externalPort | int | `83` |  |
| api.service.internalPort | int | `8093` |  |
| api.service.internalPortName | string | `http` |  |
| api.service.type | string | `"ClusterIP"` |  |
| api.ssl.clientAuth | bool | `false` |  |
| api.ssl.enabled | bool | `false` |  |
| api.updateStrategy.rollingUpdate.maxUnavailable | int | `1` |  |
| api.updateStrategy.type | string | `"RollingUpdate"` |  |
| chaos.enabled | bool | `false` |  |
| gateway.autoscaling.enabled | bool | `true` |  |
| gateway.autoscaling.maxReplicas | int | `3` |  |
| gateway.autoscaling.minReplicas | int | `1` |  |
| gateway.autoscaling.targetAverageUtilization | int | `50` |  |
| gateway.autoscaling.targetMemoryAverageUtilization | int | `80` |  |
| gateway.enabled | bool | `true` |  |
| gateway.image.pullPolicy | string | `"Always"` |  |
| gateway.image.repository | string | `"graviteeio/am-gateway"` |  |
|gateway.http.cookie.secure |bool |`false` |  |
|gateway.http.cookie.sameSite |string |`"Lax"` |  |
|gateway.http.cookie.session.name |string |`"GRAVITEE_IO_AM_SESSION"` |  |
|gateway.http.cookie.session.timeout |int |`1800000` |  |
| gateway.ingress.annotations."kubernetes.io/app-root" | string | `"/auth"` |  |
| gateway.ingress.annotations."kubernetes.io/ingress.class" | string | `"nginx"` |  |
| gateway.ingress.annotations."kubernetes.io/rewrite-target" | string | `"/auth"` |  |
| gateway.ingress.annotations."nginx.ingress.kubernetes.io/enable-rewrite-log" | string | `"true"` |  |
| gateway.ingress.annotations."nginx.ingress.kubernetes.io/ssl-redirect" | string | `"false"` |  |
| gateway.ingress.enabled | bool | `true` |  |
| gateway.ingress.hosts[0] | string | `"am.example.com"` |  |
| gateway.ingress.path | string | `"/auth"` |  |
| gateway.ingress.tls[0].hosts[0] | string | `"am.example.com"` |  |
| gateway.ingress.tls[0].secretName | string | `"api-custom-cert"` |  |
| gateway.jwt.secret | string | `"s3cR3t4grAv1t3310AMS1g1ingDftK3y"` |  |
| gateway.logging.debug | bool | `false` |  |
| gateway.logging.file.enabled | bool | `true` |  |
| gateway.logging.file.encoderPattern | string | `"%d{HH:mm:ss.SSS} [%thread] [%X{api}] %-5level %logger{36} - %msg%n"` |  |
| gateway.logging.file.rollingPolicy | string | `"\u003crollingPolicy class=\"ch.qos.logback.core.rolling.TimeBasedRollingPolicy\"\u003e\n    \u003c!-- daily rollover --\u003e\n    \u003cfileNamePattern\u003e${gravitee.home}/logs/gravitee_%d{yyyy-MM-dd}.log\u003c/fileNamePattern\u003e\n    \u003c!-- keep 30 days' worth of history --\u003e\n    \u003cmaxHistory\u003e30\u003c/maxHistory\u003e\n\u003c/rollingPolicy\u003e\n"` |  |
| gateway.logging.graviteeLevel | string | `"DEBUG"` |  |
| gateway.logging.jettyLevel | string | `"WARN"` |  |
| gateway.logging.stdout.encoderPattern | string | `"%d{HH:mm:ss.SSS} [%thread] [%X{api}] %-5level %logger{36} - %msg%n"` |  |
| gateway.name | string | `"gateway"` |  |
| gateway.reloadOnConfigChange | bool | `true` |  |
| gateway.replicaCount | int | `1` |  |
| gateway.resources.limits.cpu | string | `"500m"` |  |
| gateway.resources.limits.memory | string | `"512Mi"` |  |
| gateway.resources.requests.cpu | string | `"200m"` |  |
| gateway.resources.requests.memory | string | `"256Mi"` |  |
| gateway.service.externalPort | int | `82` |  |
| gateway.service.internalPort | int | `8092` |  |
| gateway.service.internalPortName | string | `http` |  |
| gateway.service.type | string | `"ClusterIP"` |  |
| gateway.ssl.clientAuth | bool | `false` |  |
| gateway.ssl.enabled | bool | `false` |  |
| gateway.type | string | `"Deployment"` |  |
| mongo.auth.enabled | bool | `false` |  |
| mongo.auth.password | string | `nil` |  |
| mongo.auth.source | string | `"admin"` |  |
| mongo.auth.username | string | `nil` |  |
| mongo.connectTimeoutMS | int | `30000` |  |
| mongo.dbhost | string | `"mongo-mongodb-replicaset"` |  |
| mongo.dbname | string | `"gravitee"` |  |
| mongo.dbport | int | `27017` |  |
| mongo.rs | string | `"rs0"` |  |
| mongo.rsEnabled | bool | `true` |  |
| mongo.socketKeepAlive | bool | `false` |  |
| mongo.sslEnabled | bool | `false` |  |
| mongodb-replicaset.auth.adminPassword | string | `"password"` |  |
| mongodb-replicaset.auth.adminUser | string | `"username"` |  |
| mongodb-replicaset.auth.enabled | bool | `false` |  |
| mongodb-replicaset.auth.key | string | `"keycontent"` |  |
| mongodb-replicaset.auth.metricsPassword | string | `"password"` |  |
| mongodb-replicaset.auth.metricsUser | string | `"metrics"` |  |
| mongodb-replicaset.configmap | object | `{}` |  |
| mongodb-replicaset.enabled | bool | `false` |  |
| mongodb-replicaset.image.repository | string | `"mongo"` |  |
| mongodb-replicaset.image.tag | float | `3.6` |  |
| mongodb-replicaset.persistentVolume.accessModes[0] | string | `"ReadWriteOnce"` |  |
| mongodb-replicaset.persistentVolume.enabled | bool | `true` |  |
| mongodb-replicaset.persistentVolume.size | string | `"1Gi"` |  |
| mongodb-replicaset.replicaSetName | string | `"rs0"` |  |
| mongodb-replicaset.replicas | int | `3` |  |
| mongodb-replicaset.resources.limits.cpu | string | `"500m"` |  |
| mongodb-replicaset.resources.limits.memory | string | `"512Mi"` |  |
| mongodb-replicaset.resources.requests.cpu | string | `"100m"` |  |
| mongodb-replicaset.resources.requests.memory | string | `"256Mi"` |  |
| smtp.enabled | bool | `true` |  |
| smtp.from | string | `"info@example.com"` |  |
| smtp.host | string | `"smtp.example.com"` |  |
| smtp.password | string | `"example.com"` |  |
| smtp.port | int | `25` |  |
| smtp.properties.auth | bool | `true` |  |
| smtp.properties.starttlsEnable | bool | `false` |  |
| smtp.subject | string | `"[gravitee] %s"` |  |
| smtp.username | string | `"info@example.com"` |  |
| ui.autoscaling.enabled | bool | `true` |  |
| ui.autoscaling.maxReplicas | int | `3` |  |
| ui.autoscaling.minReplicas | int | `1` |  |
| ui.autoscaling.targetAverageUtilization | int | `50` |  |
| ui.autoscaling.targetMemoryAverageUtilization | int | `80` |  |
| ui.enabled | bool | `true` |  |
| ui.image.pullPolicy | string | `"Always"` |  |
| ui.image.repository | string | `"graviteeio/am-management-ui"` |  |
| ui.ingress.annotations."ingress.kubernetes.io/configuration-snippet" | string | `"etag on;\nproxy_pass_header ETag;\n"` |  |
| ui.ingress.annotations."kubernetes.io/app-root" | string | `"/"` |  |
| ui.ingress.annotations."kubernetes.io/ingress.class" | string | `"nginx"` |  |
| ui.ingress.annotations."kubernetes.io/rewrite-target" | string | `"/"` |  |
| ui.ingress.enabled | bool | `true` |  |
| ui.ingress.hosts[0] | string | `"am.example.com"` |  |
| ui.ingress.path | string | `"/"` |  |
| ui.ingress.tls[0].hosts[0] | string | `"am.example.com"` |  |
| ui.ingress.tls[0].secretName | string | `"api-custom-cert"` |  |
| ui.name | string | `"management-ui"` |  |
| ui.replicaCount | int | `1` |  |
| ui.resources.limits.cpu | string | `"100m"` |  |
| ui.resources.limits.memory | string | `"128Mi"` |  |
| ui.resources.requests.cpu | string | `"50m"` |  |
| ui.resources.requests.memory | string | `"64Mi"` |  |
| ui.service.externalPort | int | `8002` |  |
| ui.service.internalPort | int | `80` |  |
| ui.service.internalPortName | string | `http` |  |
| ui.service.name | string | `"nginx"` |  |
| ui.service.type | string | `"ClusterIP"` |  |


## OpenShift

The Gravitee.io Access Management Helm Chart supports OpenShift > 3.10
This chart is only supporting Ingress standard objects and not the specific OpenShift Routes, reason why OpenShift is supported started from 3.10.

There are two major considerations to have in mind when deploying Gravitee.io Access Management within OpenShift:
1_ Use full host domain instead of paths for all the components (ingress paths are not well supported by OpenShift)
2_ Override the security context to let OpenShift to define automatically the user-id and the group-id to run the containers.

Here is an example regarding the UI component:

```yaml
ui:
    securityContext:
        runAsUser: null
        runAsGroup: null
        runAsNonRoot: true
```

By setting the value to `null` for `runAsUser` and `runAsGroup` it forces OpenShift to define the correct values for you while deploying the Helm Chart.