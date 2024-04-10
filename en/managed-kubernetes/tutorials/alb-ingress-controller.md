# Setting up the {{ alb-name }} Ingress controller

The [{{ alb-full-name }}](../../application-load-balancer/) service is designed for load balancing and traffic distribution across applications. To use it for managing ingress traffic of applications running in a [{{ managed-k8s-name }} cluster](../concepts/index.md#kubernetes-cluster), you need an [Ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/).

To set up access to the applications running in your {{ managed-k8s-name }} cluster via {{ alb-name }}:
1. [{#T}](#create-ingress-and-apps).
1. [{#T}](#configure-group).
1. [{#T}](#verify-setup).

## Getting started {#before-you-begin}

1. [Register a public domain zone and delegate your domain](../../dns/operations/zone-create-public.md).
1. If you already have a certificate for the domain zone, [add its details](../../certificate-manager/operations/import/cert-create.md) to the [{{ certificate-manager-full-name }}](../../certificate-manager/) service. Alternatively, you can [add a new Let's Encrypt® certificate](../../certificate-manager/operations/managed/cert-create.md).
1. {% include [k8s-ingress-controller-create-cluster](../../_includes/application-load-balancer/k8s-ingress-controller-create-cluster.md) %}
1. {% include [k8s-ingress-controller-create-node-group](../../_includes/application-load-balancer/k8s-ingress-controller-create-node-group.md) %}
1. Create [security group](../../vpc/concepts/security-groups.md) rules to allow:

   * [Incoming and outgoing service traffic](../operations/connect/security-groups.md#rules-internal) for your {{ managed-k8s-name }} cluster and node group.
   * Incoming traffic for the node group:

      * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-port-range }}**: `10501-10502`.
      * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-protocol }}**: `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_tcp }}`.
      * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-source }}**: `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}`.
      * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-cidr-blocks }}**: Specify the ranges of subnet IP addresses to be used when [creating an Ingress controller](#create-ingress-and-apps), e.g., `10.96.0.0/16` or `10.112.0.0/16`.

1. [Install the {{ alb-name }} Ingress controller](../operations/applications/alb-ingress-controller.md).
1. {% include [install externaldns](../../_includes/managed-kubernetes/install-externaldns.md) %}
1. {% include [Install and configure kubectl](../../_includes/managed-kubernetes/kubectl-install.md) %}

   {% include [Run kubectl cluster-info](../../_includes/managed-kubernetes/kubectl-info.md) %}

## Set up the Ingress controller and test applications {#create-ingress-and-apps}

The Ingress controller's workload can include [{{ k8s }} services](../concepts/index.md#service) or [backend groups](../../application-load-balancer/concepts/backend-group.md#types), such as {{ alb-name }} target groups or {{ objstorage-full-name }} buckets.

Before getting started, get the ID of the [previously added](#before-you-begin) TLS certificate:

```bash
yc certificate-manager certificate list
```

Command result:

```text
+----------------------+-----------+----------------+---------------------+----------+--------+
|          ID          |   NAME    |    DOMAINS     |      NOT AFTER      |   TYPE   | STATUS |
+----------------------+-----------+----------------+---------------------+----------+--------+
| fpq8diorouhp******** | sert-test |    test.ru     | 2022-01-06 17:19:37 | IMPORTED | ISSUED |
+----------------------+-----------+----------------+---------------------+----------+--------+
```

{% list tabs %}

- Ingress controller for {{ k8s }} services

   1. In a separate folder, create `demo-app-1.yaml` and `demo-app-2.yaml` application files:

      {% cut "demo-app-1.yaml" %}

      ```yaml
      apiVersion: v1
      kind: ConfigMap
      metadata:
        name: alb-demo-1
      data:
        nginx.conf: |
          worker_processes auto;
          events {
          }
          http {
            server {
              listen 80 ;
              location = /_healthz {
                add_header Content-Type text/plain;
                return 200 'ok';
              }
              location / {
                add_header Content-Type text/plain;
                return 200 'Index';
              }
              location = /app1 {
                add_header Content-Type text/plain;
                return 200 'This is APP#1';
              }
            }
          }
      ---
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: alb-demo-1
        labels:
          app: alb-demo-1
          version: v1
      spec:
        replicas: 2
        selector:
          matchLabels:
            app: alb-demo-1
        strategy:
          type: RollingUpdate
          rollingUpdate:
            maxSurge: 1
            maxUnavailable: 0
        template:
          metadata:
            labels:
              app: alb-demo-1
              version: v1
          spec:
            terminationGracePeriodSeconds: 5
            volumes:
              - name: alb-demo-1
                configMap:
                  name: alb-demo-1
            containers:
              - name: alb-demo-1
                image: nginx:latest
                ports:
                  - name: http
                    containerPort: 80
                livenessProbe:
                  httpGet:
                    path: /_healthz
                    port: 80
                  initialDelaySeconds: 3
                  timeoutSeconds: 2
                  failureThreshold: 2
                volumeMounts:
                  - name: alb-demo-1
                    mountPath: /etc/nginx
                    readOnly: true
                resources:
                  limits:
                    cpu: 250m
                    memory: 128Mi
                  requests:
                    cpu: 100m
                    memory: 64Mi
      ---
      apiVersion: v1
      kind: Service
      metadata:
        name: alb-demo-1
      spec:
        selector:
          app: alb-demo-1
        type: NodePort
        ports:
          - name: http
            port: 80
            targetPort: 80
            protocol: TCP
            nodePort: 30081
      ```

      {% endcut %}

      {% cut "demo-app-2.yaml" %}

      ```yaml
      apiVersion: v1
      kind: ConfigMap
      metadata:
        name: alb-demo-2
      data:
        nginx.conf: |
          worker_processes auto;
          events {
          }
          http {
            server {
              listen 80 ;
              location = /_healthz {
                add_header Content-Type text/plain;
                return 200 'ok';
              }
              location / {
                add_header Content-Type text/plain;
                return 200 'Add app#';
              }
              location = /app2 {
                add_header Content-Type text/plain;
                return 200 'This is APP#2';
              }
            }
          }
      ---
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: alb-demo-2
        labels:
          app: alb-demo-2
          version: v1
      spec:
        replicas: 2
        selector:
          matchLabels:
            app: alb-demo-2
        strategy:
          type: RollingUpdate
          rollingUpdate:
            maxSurge: 1
            maxUnavailable: 0
        template:
          metadata:
            labels:
              app: alb-demo-2
              version: v1
          spec:
            terminationGracePeriodSeconds: 5
            volumes:
              - name: alb-demo-2
                configMap:
                  name: alb-demo-2
            containers:
              - name: alb-demo-2
                image: nginx:latest
                ports:
                  - name: http
                    containerPort: 80
                livenessProbe:
                  httpGet:
                    path: /_healthz
                    port: 80
                  initialDelaySeconds: 3
                  timeoutSeconds: 2
                  failureThreshold: 2
                volumeMounts:
                  - name: alb-demo-2
                    mountPath: /etc/nginx
                    readOnly: true
                resources:
                  limits:
                    cpu: 250m
                    memory: 128Mi
                  requests:
                    cpu: 100m
                    memory: 64Mi
      ---
      apiVersion: v1
      kind: Service
      metadata:
        name: alb-demo-2
      spec:
        selector:
          app: alb-demo-2
        type: NodePort
        ports:
          - name: http
            port: 80
            targetPort: 80
            protocol: TCP
            nodePort: 30082
      ```

      {% endcut %}

   1. In the same folder, create a file named `ingress.yaml` and specify the [previously delegated domain name](#before-you-begin), certificate ID, and settings for {{ alb-name }} in it:

      ```yaml
      apiVersion: networking.k8s.io/v1
      kind: Ingress
      metadata:
        name: alb-demo-tls
        annotations:
          ingress.alb.yc.io/subnets: <list_of_subnet_IDs>
          ingress.alb.yc.io/security-groups: <list_of_security_group_IDs>
          ingress.alb.yc.io/external-ipv4-address: <IP_address_assignment_method>
          ingress.alb.yc.io/group-name: my-ingress-group
      spec:
        tls:
          - hosts:
              - <domain_name>
            secretName: yc-certmgr-cert-id-<TLS_certificate_ID>
        rules:
          - host: <domain_name>
            http:
              paths:
                - path: /app1
                  pathType: Prefix
                  backend:
                    service:
                      name: alb-demo-1
                      port:
                        number: 80
                - path: /app2
                  pathType: Prefix
                  backend:
                    service:
                      name: alb-demo-2
                      port:
                        number: 80
                - pathType: Prefix
                  path: "/"
                  backend:
                    service:
                      name: alb-demo-2
                      port:
                        name: http
      ```

      Where:

      * `ingress.alb.yc.io/subnets`: One or more subnets that {{ alb-name }} is going to work with.
      * `ingress.alb.yc.io/security-groups`: One or more [security groups](../../application-load-balancer/concepts/application-load-balancer.md#security-groups) for {{ alb-name }}. If you skip this parameter, the default security group will be used. At least one of the security groups must allow outgoing TCP connections on ports 10501 and 10502 in the {{ managed-k8s-name }} node group subnet or security group.
      * `ingress.alb.yc.io/external-ipv4-address`: Providing public online access to {{ alb-name }}. Enter the [previously obtained IP address](../../vpc/operations/get-static-ip.md) or set `auto` to obtain a new IP address automatically.

         If you set `auto`, deleting the Ingress controller will also delete the [IP address](../../vpc/concepts/address.md) from the [cloud](../../resource-manager/concepts/resources-hierarchy.md#cloud). To avoid this, use an existing reserved IP address.

      * `ingress.alb.yc.io/group-name`: Group name. {{ k8s }} Ingress resources are grouped together, with each group served by a separate {{ alb-name }} instance.

         You can replace `my-ingress-group` with any group name you like. Make sure it meets the naming [requirements]({{ k8s-docs }}/concepts/overview/working-with-objects/names/).

      In [ALB Ingress Controller](/marketplace/products/yc/alb-ingress-controller) versions prior to 0.2.0, each backend group corresponds to a bundle of `host`, `http.paths.path`, and `http.paths.pathType` parameters. In versions 0.2.0 and later, the backend group corresponds to the `backend.service` parameter. This may cause collisions when updating the ALB Ingress Controller. To avoid them, [find out whether upgrade restrictions apply](../operations/applications/upgrade-alb-ingress-controller.md) to your infrastructure.

      (Optional) Enter the advanced settings for the controller:

      {% cut "Additional settings" %}

      * `ingress.alb.yc.io/group-settings-name`: Name for the Ingress group settings to be described in the optional `IngressGroupSettings` resource. For more information, see [Set up the Ingress group](#configure-group).
      * `ingress.alb.yc.io/internal-ipv4-address`: Provide internal access to {{ alb-name }}. Enter the internal IP address or use `auto` to obtain the IP address automatically.

         {% note info %}

         You can only use one type of access to {{ alb-name }} at a time: `ingress.alb.yc.io/external-ipv4-address` or `ingress.alb.yc.io/internal-ipv4-address`.

         {% endnote %}

      * `ingress.alb.yc.io/internal-alb-subnet`: Subnet for hosting the {{ alb-name }} internal IP address. This parameter is required if the `ingress.alb.yc.io/internal-ipv4-address` parameter is selected.
      * `ingress.alb.yc.io/protocol`: Connection protocol used by the load balancer and the backends:
         * `http`: HTTP/1.1; default value
         * `http2`: HTTP/2
         * `grpc`: gRPC
      * `ingress.alb.yc.io/transport-security`: Encryption protocol for connections between the load balancer and backends.

         {% note warning %}

         In [ALB Ingress Controller](/marketplace/products/yc/alb-ingress-controller) version 0.2.0 and later, you can only use an annotation in the [Service](../../application-load-balancer/k8s-ref/service.md#metadata) object.

         If you specify an annotation in the `Ingress` resources that use a single service with the same settings for backend groups, such annotation will apply correctly. However, this mechanism is obsolete and will not be supported going forward.

         {% endnote %}

         The acceptable value is `tls`: TLS with no certificate challenge.

         If no annotation is specified, the load balancer connects to the backends with no encryption.
      * `ingress.alb.yc.io/prefix-rewrite`: Replace the path for the specified value.
      * `ingress.alb.yc.io/upgrade-types`: Valid values for the `Upgrade` HTTP header, e.g., `websocket`.
      * `ingress.alb.yc.io/request-timeout`: Maximum period for which the connection can be established.
      * `ingress.alb.yc.io/idle-timeout`: Maximum connection keep-alive time with zero data transmission.

         Values for `request-timeout` and `idle-timeout` must be specified with units of measurement, e.g., `300ms`, `1.5h`. Acceptable units of measurement include:
         * `ns`: Nanoseconds
         * `us`: Microseconds
         * `ms`: Milliseconds
         * `s`: Seconds
         * `m`: Minutes
         * `h`: Hours

      * `ingress.alb.yc.io/use-regex`: Support for [RE2](https://github.com/google/re2/wiki/Syntax) regular expressions when matching the request path. If the `true` string is provided, the support is enabled. Only applies if the `pathType` parameter is set to `Exact`.

      {% endcut %}

      {% note info %}

      The settings only apply to the hosts of the given controller rather than the entire Ingress group.

      {% endnote %}

      For more information about the Ingress resource settings, see [{#T}](../alb-ref/ingress.md).

   1. Create an Ingress controller and applications:

      ```bash
      kubectl apply -f .
      ```

   1. Wait until the Ingress controller is created and assigned a public IP address. This may take several minutes.

      To track the progress of controller creation and check that it is error-free, open the logs of the pod where the controller is being created:

      1. In the [management console]({{ link-console-main }}), go to the folder page and select **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-kubernetes }}**.
      1. Click the cluster name and select **{{ ui-key.yacloud.k8s.cluster.switch_workloads }}** in the left-hand panel.
      1. Select one of the `alb-demo-***` pods the Ingress controller is being created on.
      1. Go to the **{{ ui-key.yacloud.k8s.workloads.label_tab-logs }}** tab on the pod page.

         You will see the Ingress controller creation logged, with the logs displayed in real time. If an error occurs while creating the controller, it will appear in the logs.

   1. Make sure the Ingress controller has been created. To do this, run the appropriate command and check that the command output shows the following value in the `ADDRESS` field:

      ```bash
      kubectl get ingress alb-demo-tls
      ```

      Result:

      ```bash
      NAME          CLASS   HOSTS          ADDRESS       PORTS    AGE
      alb-demo-tls  <none>  <domain_name>  <IP_address>  80, 443  15h
      ```

      Based on the Ingress controller configuration, an [L7 load balancer](../../application-load-balancer/concepts/application-load-balancer.md) will be automatically deployed.

- Ingress controller for a backend group

   To set up a [backend group](../../application-load-balancer/concepts/backend-group.md) use the `HttpBackendGroup` [CustomResourceDefinition](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/). As a backend, you can use an {{ alb-name }} target group or an {{ objstorage-name }} bucket.

   To configure {{ alb-name }} to work with a backend group:
   1. Create a [backend group with a bucket](../../application-load-balancer/operations/backend-group-create.md#with-s3-bucket):
      1. Create a [public bucket in {{ objstorage-name }}](../../tutorials/web/static.md#create-public-bucket).
      1. [Configure the website homepage and error page](../../tutorials/web/static.md#index-and-error).
   1. Create a configuration file named `demo-app-1.yaml` for your application:

      {% cut "demo-app-1.yaml" %}

      ```yaml
      apiVersion: v1
      kind: ConfigMap
      metadata:
        name: alb-demo-1
      data:
        nginx.conf: |
          worker_processes auto;
          events {
          }
          http {
            server {
              listen 80 ;
              location = /_healthz {
                add_header Content-Type text/plain;
                return 200 'ok';
              }
              location / {
                add_header Content-Type text/plain;
                return 200 'Index';
              }
              location = /app1 {
                add_header Content-Type text/plain;
                return 200 'This is APP#1';
              }
            }
          }
      ---
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: alb-demo-1
        labels:
          app: alb-demo-1
          version: v1
      spec:
        replicas: 2
        selector:
          matchLabels:
            app: alb-demo-1
        strategy:
          type: RollingUpdate
          rollingUpdate:
            maxSurge: 1
            maxUnavailable: 0
        template:
          metadata:
            labels:
              app: alb-demo-1
              version: v1
          spec:
            terminationGracePeriodSeconds: 5
            volumes:
              - name: alb-demo-1
                configMap:
                  name: alb-demo-1
            containers:
              - name: alb-demo-1
                image: nginx:latest
                ports:
                  - name: http
                    containerPort: 80
                livenessProbe:
                  httpGet:
                    path: /_healthz
                    port: 80
                  initialDelaySeconds: 3
                  timeoutSeconds: 2
                  failureThreshold: 2
                volumeMounts:
                  - name: alb-demo-1
                    mountPath: /etc/nginx
                    readOnly: true
                resources:
                  limits:
                    cpu: 250m
                    memory: 128Mi
                  requests:
                    cpu: 100m
                    memory: 64Mi
      ---
      apiVersion: v1
      kind: Service
      metadata:
        name: alb-demo-1
      spec:
        selector:
          app: alb-demo-1
        type: NodePort
        ports:
          - name: http
            port: 80
            targetPort: 80
            protocol: TCP
            nodePort: 30081
      ```

      {% endcut %}

   1. In a separate directory, create a file named `http-group.yaml` with the `HttpBackendGroup` object settings:

      ```yaml
      apiVersion: alb.yc.io/v1alpha1
      kind: HttpBackendGroup
      metadata:
        name: example-backend-group
      spec:
        backends: # The list of backends.
          - name: alb-demo-1
            weight: 70 # The relative weight of the backend when distributing traffic. The load will be distributed proportionally to the weight of other backends in the group. Be sure to specify the weight even if you have only one backend in the group.
            service:
               name: alb-demo-1
               port:
                 number: 80
          - name: bucket-backend
            weight: 30
            storageBucket:
              name: <bucket_name>
      ```

      (Optional) Enter the advanced settings for the controller:
      * `spec.backends.useHttp2`: The mode using the `HTTP/2` protocol.
      * `spec.backends.tls`: A certificate from the certificate authority that the load balancer will trust when establishing a secure connection with backend endpoints. Specify the certificate contents in the `trustedCa` field as open text.

      For more information, see [{#T}](../../application-load-balancer/concepts/backend-group.md).
   1. Create a file named `ingress-http.yaml` and specify the previously [delegated domain name](#before-you-begin), certificate ID, and settings for {{ alb-name }} in it:

      ```yaml
      apiVersion: networking.k8s.io/v1
      kind: Ingress
      metadata:
        name: alb-demo-tls
        annotations:
          ingress.alb.yc.io/subnets: <list_of_subnet_IDs>
          ingress.alb.yc.io/security-groups: <list_of_security_group_IDs>
          ingress.alb.yc.io/external-ipv4-address: <IP_address_assignment_method>
          ingress.alb.yc.io/group-name: my-ingress-group
      spec:
        tls:
          - hosts:
            - <domain_name>
            secretName: yc-certmgr-cert-id-<TLS_certificate_ID>
        rules:
          - host: <domain_name>
            http:
              paths:
                - path: /app1
                  pathType: Exact
                  backend:
                    resource:
                      apiGroup: alb.yc.io
                      kind: HttpBackendGroup
                      name: example-backend-group
      ```

      Where:
      * `ingress.alb.yc.io/subnets`: One or more subnets that {{ alb-name }} is going to work with.
      * `ingress.alb.yc.io/security-groups`: One or more [security groups](../../application-load-balancer/concepts/application-load-balancer.md#security-groups) for {{ alb-name }}. If you skip this parameter, the default security group will be used. At least one of the security groups must allow outgoing TCP connections on ports 10501 and 10502 in the {{ managed-k8s-name }} node group subnet or security group.
      * `ingress.alb.yc.io/external-ipv4-address`: Providing public online access to {{ alb-name }}. Enter the [previously obtained IP address](../../vpc/operations/get-static-ip.md) or set `auto` to obtain a new IP address automatically.

         If you set `auto`, deleting the Ingress controller will also delete the [IP address](../../vpc/concepts/address.md) from the [cloud](../../resource-manager/concepts/resources-hierarchy.md#cloud). To avoid this, use an existing reserved IP address.

      * `ingress.alb.yc.io/group-name`: Group name. {{ k8s }} Ingress resources are grouped together, with each group served by a separate {{ alb-name }} instance.

         You can replace `my-ingress-group` with any group name you like. Make sure it meets the naming [requirements]({{ k8s-docs }}/concepts/overview/working-with-objects/names/).

      (Optional) Enter the advanced settings for the controller:
      * `ingress.alb.yc.io/group-settings-name`: Name for the Ingress group settings to be described in the optional `IngressGroupSettings` resource. For more information, see [Set up the Ingress group](#configure-group).
      * `ingress.alb.yc.io/internal-ipv4-address`: Provide internal access to {{ alb-name }}. Enter the internal IP address or use `auto` to obtain the IP address automatically.

         {% note info %}

         You can only use one type of access to {{ alb-name }} at a time: `ingress.alb.yc.io/external-ipv4-address` or `ingress.alb.yc.io/internal-ipv4-address`.

         {% endnote %}

      * `ingress.alb.yc.io/internal-alb-subnet`: Subnet for hosting the {{ alb-name }} internal IP address. This parameter is required if the `ingress.alb.yc.io/internal-ipv4-address` parameter is selected.
      * `ingress.alb.yc.io/protocol`: Connection protocol used by the load balancer and the backends:
         * `http`: HTTP/1.1; default value
         * `http2`: HTTP/2
         * `grpc`: gRPC
      * `ingress.alb.yc.io/prefix-rewrite`: Replace the path for the specified value.
      * `ingress.alb.yc.io/upgrade-types`: Valid values for the `Upgrade` HTTP header, e.g., `websocket`.
      * `ingress.alb.yc.io/request-timeout`: Maximum period for which the connection can be established.
      * `ingress.alb.yc.io/idle-timeout`: Maximum connection keep-alive time with zero data transmission.

         Values for `request-timeout` and `idle-timeout` must be specified with units of measurement, e.g., `300ms`, `1.5h`. Acceptable units of measurement include:
         * `ns`: Nanoseconds
         * `us`: Microseconds
         * `ms`: Milliseconds
         * `s`: Seconds
         * `m`: Minutes
         * `h`: Hours

      * `ingress.alb.yc.io/use-regex`: Support for [RE2](https://github.com/google/re2/wiki/Syntax) regular expressions when matching the request path. If the `true` string is provided, the support is enabled. Only applies if the `pathType` parameter is set to `Exact`.

      {% note info %}

      The settings only apply to the hosts of the given controller rather than the entire Ingress group.

      {% endnote %}

      For more information about the Ingress resource settings, see [{#T}](../alb-ref/ingress.md).
   1. Create an Ingress controller, an `HttpBackendGroup` object, and a {{ k8s }} app:

      ```bash
      kubectl apply -f .
      ```

   1. Wait until the Ingress controller is created and assigned a public IP address. This may take several minutes:

      To track the progress of controller creation and check that it is error-free, open the logs of the pod where the controller is being created:

      1. In the [management console]({{ link-console-main }}), go to the folder page and select **{{ ui-key.yacloud.iam.folder.dashboard.label_managed-kubernetes }}**.
      1. Click the cluster name and select **{{ ui-key.yacloud.k8s.cluster.switch_workloads }}** in the left-hand panel.
      1. Select one of the `alb-demo-***` pods the Ingress controller is being created on.
      1. Go to the **{{ ui-key.yacloud.k8s.workloads.label_tab-logs }}** tab on the pod page.

         You will see the Ingress controller creation logged, with the logs displayed in real time. If an error occurs while creating the controller, it will appear in the logs.

   1. Make sure the Ingress controller has been created. To do this, run the appropriate command and check that the command output shows the following value in the `ADDRESS` field:

      ```bash
      kubectl get ingress alb-demo-tls
      ```

      Result:

      ```bash
      NAME          CLASS   HOSTS          ADDRESS       PORTS    AGE
      alb-demo-tls  <none>  <domain_name>  <IP_address>  80, 443  15h
      ```

      Based on the Ingress controller configuration, an L7 load balancer will be automatically deployed.

{% endlist %}

## (Optional) Set up the Ingress group {#configure-group}

Specifying a name for the Ingress group settings using the `ingress.alb.yc.io/group-settings-name` annotation during the installation of the Ingress controller enables you to set logging settings for the L7 balancer. To do this, [create a custom log group](../../logging/operations/create-group.md) and specify the Ingress group settings in the optional `IngressGroupSettings` resource.
1. Create a `settings.yaml` file with your logging settings and the custom log group ID. For example:

   ```yaml
   apiVersion: alb.yc.io/v1alpha1
   kind: IngressGroupSettings
   metadata:
     name: <name_for_Ingress_group_settings>
   logOptions:
     logGroupID: <custom_log_group_ID>
     discardRules:
       - discardPercent: 50
         grpcCodes:
           - OK
           - CANCELLED
           - UNKNOWN
       - discardPercent: 67
         httpCodeIntervals:
           - HTTP_1XX
       - discardPercent: 20
         httpCodes:
           - 200
           - 404
   ```

   Where `name` is the name for Ingress group settings in the `ingress.alb.yc.io/group-settings-name` annotation.

1. Apply the settings for the Ingress group:

   ```bash
   kubectl apply -f settings.yaml
   ```

## Make sure the {{ managed-k8s-name }} cluster applications are accessible through {{ alb-name }} {#verify-setup}

1. If you have no [ExternalDNS with a plugin for {{ dns-name }}](/marketplace/products/yc/externaldns) installed, [add an A record to your domain zone](../../dns/operations/resource-record-create.md). In the **Value** field, specify the public IP address of the Ingress controller. If you are using ExternalDNS with a plugin for {{ dns-full-name }}, this record will be created automatically.
1. [Configure the load balancer's security groups](../../application-load-balancer/concepts/application-load-balancer.md#security-groups).
1. Test {{ alb-name }}:

   {% list tabs %}

   - {{ k8s }} services

      Open the application URIs in your browser:

      ```http
      https://<your_domain>/app1
      https://<your_domain>/app2
      ```

      Make sure the applications are accessible via {{ alb-name }} and return pages with `This is APP#1` and `This is APP#2` text, respectively.

   - Backend group

      Open the application URI in your browser:

      ```http
      https://<your_domain>/app1
      ```

      Make sure that the target resources are accessible via {{ alb-name }}.

   {% endlist %}

## Delete the resources you created {#clear-out}

Some resources are not free of charge. To avoid paying for them, delete the resources you no longer need:
1. [Delete the {{ managed-k8s-name }} cluster](../../managed-kubernetes/operations/kubernetes-cluster/kubernetes-cluster-delete.md).
1. [Delete the {{ alb-name }} target groups](../../application-load-balancer/operations/target-group-delete.md).
1. [Delete the {{ objstorage-name }} bucket](../../storage/operations/buckets/delete.md).