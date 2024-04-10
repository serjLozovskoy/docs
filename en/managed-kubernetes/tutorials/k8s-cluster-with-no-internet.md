# Creating and configuring a {{ managed-k8s-name }} cluster with no internet access

You can create and configure a {{ managed-k8s-name }} cluster with no internet connectivity. To do this, use the following configuration:

* {{ managed-k8s-name }} cluster and node group without a public address. You can only connect to such a cluster using a {{ yandex-cloud }} virtual machine.
* The cluster and node group are hosted by subnets with no internet access.
* Service accounts have no roles to use resources with internet access, e.g., [{{ network-load-balancer-full-name }}](../../network-load-balancer/index.yaml).
* Cluster security groups restrict incoming and outgoing traffic.

To create a {{ managed-k8s-name }} cluster with no internet access:

1. [{#T}](#infra).
1. [{#T}](#vm).
1. [{#T}](#check).
1. [{#T}](#cert).

If you no longer need the resources you created, [delete them](#clear-out).

## Prepare the infrastructure for {{ managed-k8s-name }} {#infra}

{% list tabs group=instructions %}

- Manually {#manual}

   1. [Create service accounts](../../iam/operations/sa/create.md):

      * `resource-sa` with `{{ roles.k8s.clusters.agent }}`, `{{ roles-logging-writer }}`, and `kms.keys.encrypterDecrypter` [roles](../../iam/concepts/access-control/roles.md) for the folder where the {{ k8s }} cluster is created. This account will be used to create the resources required for the {{ k8s }} cluster.
      * `node-sa` with the `{{ roles-cr-puller }}` role. Nodes will pull the required Docker images from the registry on behalf of this account.

      {% note tip %}

      You can use the same service account to manage your {{ k8s }} cluster and its node groups.

      {% endnote %}

   1. [Create a {{ kms-full-name }} symmetric encryption key](../../kms/operations/key.md#create) with the following parameters:

      * **{{ ui-key.yacloud.common.name }}**: `my-kms-key`
      * **{{ ui-key.yacloud.kms.symmetric-key.form.field_algorithm }}**: `AES-256`
      * **{{ ui-key.yacloud.kms.symmetric-key.form.field_rotation }}**: 365 days

   1. [Create a network](../../vpc/operations/network-create.md) named `my-net`.
   1. [Create a subnet](../../vpc/operations/subnet-create.md) named `my-subnet` with the `internal.` domain name.
   1. [Create a security group](../../vpc/operations/security-group-create.md) named `k8s-security-group` and specify its rules:

      {% cut "Security group rules" %}

      * For incoming traffic:

         * For a network load balancer:

            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-port-range }}**: `{{ port-any }}`
            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-protocol }}**: `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_tcp }}`
            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-source }}**: `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-sg-type-balancer }}`

         * To transfer service traffic between the [master](../concepts/index.md#master) and [nodes](../concepts/index.md#node-group):

            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-port-range }}**: `{{ port-any }}`
            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-protocol }}**: `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_any }}` (`Any`)
            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-source }}**: `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-sg }}`
            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-sg-type }}**: `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-sg-type-self }}` (`Self`)

         * To transfer traffic between [pods](../concepts/index.md#pod) and [services](../concepts/index.md#service):

            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-port-range }}**: `{{ port-any }}`
            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-protocol }}**: `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_any }}` (`Any`)
            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-source }}**: `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}`
            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-cidr-blocks }}**: Address ranges `172.19.0.0/16` and `172.20.0.0/16`. You will need these ranges later when creating a {{ managed-k8s-name }} cluster as a cluster and service CIDR.

         * To test the nodes using ICMP requests from the subnets within {{ yandex-cloud }}:

            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-protocol }}**: `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_icmp }}`
            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-source }}**: `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}`
            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-cidr-blocks }}**: Address ranges of the subnets within {{ yandex-cloud }} to diagnose the cluster from:

               * `10.0.0.0/8`
               * `192.168.0.0/16`
               * `172.16.0.0/12`

         * To connect to services from the internet:

            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-port-range }}**: `30000-32767`
            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-protocol }}**: `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_tcp }}`
            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-source }}**: `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}`
            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-cidr-blocks }}**: `0.0.0.0/0`

         * To connect to nodes over SSH:

            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-port-range }}**: `{{ port-ssh }}`
            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-protocol }}**: `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_tcp }}`
            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-source }}**: `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}`
            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-cidr-blocks }}**: `0.0.0.0/0`

         * To access the {{ k8s }} API and manage a cluster using `kubectl` or other utilities:

            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-port-range }}**: `{{ port-https }}`, `{{ port-k8s }}`
            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-protocol }}**: `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_tcp }}`
            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-source }}**: `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}`
            * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-cidr-blocks }}**: `0.0.0.0/0`

      * For outgoing traffic:

         * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-port-range }}**: `{{ port-any }}`
         * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-protocol }}**: `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_any }}` (`Any`)
         * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-destination }}**: `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}`
         * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-cidr-blocks }}**: `0.0.0.0/0`

      {% endcut %}

   1. [Create a {{ managed-k8s-name }} cluster](../operations/kubernetes-cluster/kubernetes-cluster-create.md#kubernetes-cluster-create) with the following parameters:

      * **{{ ui-key.yacloud.k8s.clusters.create.field_service-account }}**: `resource-sa`
      * **{{ ui-key.yacloud.k8s.clusters.create.field_node-service-account }}**: `node-sa`
      * **{{ ui-key.yacloud.k8s.clusters.create.field_kms-key }}**: `my-kms-key`
      * **{{ ui-key.yacloud.k8s.clusters.create.field_address-type }}**: No address
      * **{{ ui-key.yacloud.k8s.clusters.create.field_network }}**: `my-net`
      * **{{ ui-key.yacloud.k8s.clusters.create.field_subnetwork }}**: `my-subnet`
      * **{{ ui-key.yacloud.mdb.forms.field_security-group }}**: `k8s-security-group`
      * **{{ ui-key.yacloud.k8s.clusters.create.field_cluster-cidr }}**: `172.19.0.0/16`
      * **{{ ui-key.yacloud.k8s.clusters.create.field_service-cidr }}**: `172.20.0.0/16`
      * **{{ ui-key.yacloud.k8s.clusters.create.label_logging-enabled }}**: Enabled
      * **{{ ui-key.yacloud.k8s.clusters.create.label_stream-cluster-autoscaler }}**: Enabled
      * **{{ ui-key.yacloud.k8s.clusters.create.label_stream-events }}**: Enabled
      * **{{ ui-key.yacloud.k8s.clusters.create.label_stream-kube-apiserver }}**: Enabled

   1. In the {{ managed-k8s-name }} cluster, [create a node group](../operations/node-group/node-group-create.md) with the following parameters:

      * **{{ ui-key.yacloud.k8s.node-groups.create.field_address-type }}**: No address
      * **{{ ui-key.yacloud.mdb.forms.field_security-group }}**: `k8s-security-group`
      * **{{ ui-key.yacloud.k8s.node-groups.create.field_locations }}**: `my-subnet`

- {{ TF }} {#tf}

   1. {% include [terraform-install-without-setting](../../_includes/mdb/terraform/install-without-setting.md) %}
   1. {% include [terraform-authentication](../../_includes/mdb/terraform/authentication.md) %}
   1. {% include [terraform-setting](../../_includes/mdb/terraform/setting.md) %}
   1. {% include [terraform-configure-provider](../../_includes/mdb/terraform/configure-provider.md) %}
   1. Download the [k8s-cluster-with-no-internet.tf](https://github.com/yandex-cloud-examples/yc-mk8s-cluster-without-internet/blob/main/k8s-cluster-with-no-internet.tf) configuration file to the same working directory. This file will be used to create the following resources:

      * Network.
      * Route table.
      * Subnets.
      * [Security groups](../../vpc/concepts/security-groups.md) and [rules](../operations/connect/security-groups.md) required for the {{ managed-k8s-name }} cluster and node group to operate:

         * Rules for service traffic.
         * Rule for connection to services from the internet.
         * Rule for connection to nodes.
         * Rules for accessing the {{ k8s }} API and managing the {{ managed-k8s-name }} cluster using `kubectl`.

      * [Service accounts](../../iam/concepts/users/service-accounts.md) for {{ k8s }} resources and nodes.
      * {{ kms-full-name }} [symmetric encryption key](../../kms/concepts/key.md).
      * {{ managed-k8s-name }} cluster.
      * {{ managed-k8s-name }} node group.

      The file is generated using the libraries of the [terraform-yc-vpc](https://github.com/terraform-yc-modules/terraform-yc-vpc) and [terraform-yc-kubernetes](https://github.com/terraform-yc-modules/terraform-yc-kubernetes) modules. For more information on the configuration of the resources you create using these modules, see the library pages.

   1. Make sure the {{ TF }} configuration files are correct using this command:

      ```bash
      terraform validate
      ```

      If there are any errors in the configuration files, {{ TF }} will point them out.

   1. Create the required infrastructure:

      {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

      {% include [explore-resources](../../_includes/mdb/terraform/explore-resources.md) %}

{% endlist %}

## Set up a virtual machine {#vm}

As the {{ managed-k8s-name }} cluster has no internet access, you can only connect to it from a VM that is in the same network as the cluster. Therefore, to check the cluster availability, set up the infrastructure:

1. Create the required resources:

   {% list tabs group=instructions %}

   - Manually {#manual}

      1. Create a service account named `vm-sa` with the `{{ roles.k8s.cluster-api.cluster-admin }}` and `{{ roles.k8s.admin }}` roles. This account will be used to connect to the {{ managed-k8s-name }} cluster.
      1. Create a security group named `vm-security-group` and specify its rule for incoming traffic:

         * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-port-range }}**: `{{ port-ssh }}`
         * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-protocol }}**: `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_tcp }}`
         * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-source }}**: `{{ ui-key.yacloud.vpc.network.security-groups.forms.value_sg-rule-destination-cidr }}`
         * **{{ ui-key.yacloud.vpc.network.security-groups.forms.field_sg-rule-cidr-blocks }}**: `0.0.0.0/0`

      1. [Create a Linux VM](../../compute/operations/vm-create/create-linux-vm.md) with the following parameters:

         * **{{ ui-key.yacloud.component.compute.network-select.field_subnetwork }}**: `my-subnet`
         * **{{ ui-key.yacloud.component.compute.network-select.field_external }}**: `{{ ui-key.yacloud.component.compute.network-select.switch_auto }}` or you can [reserve a static public IP address](../../vpc/operations/get-static-ip.md) and assign it to the new VM.
         * **{{ ui-key.yacloud.component.compute.network-select.field_security-groups }}**: `vm-security-group`
         * **{{ ui-key.yacloud.compute.instances.create.field_service-account }}**: `vm-sa`

   - {{ TF }} {#tf}

      1. Download the [virtual-machine-for-k8s.tf](https://github.com/yandex-cloud-examples/yc-mk8s-cluster-without-internet/blob/main/virtual-machine-for-k8s.tf) configuration file to the directory containing the `k8s-cluster-with-no-internet.tf` file.

         This file describes:

         * Service account for VM
         * Security group for VM
         * VM

      1. In the `virtual-machine-for-k8s.tf` file, specify:

         * Folder ID.
         * ID of the network created together with the {{ managed-k8s-name }} cluster.
         * ID of the subnet created together with the {{ managed-k8s-name }} cluster and residing in the `{{ region-id }}-b` availability zone. You can find this zone in the VM settings.
         * Username to be used for connection to the VM over SSH.
         * Absolute path to the public part of the SSH key for connection to the VM.

      1. Make sure the {{ TF }} configuration files are correct using this command:

         ```bash
         terraform validate
         ```

         If there are any errors in the configuration files, {{ TF }} will point them out.

      1. Create the required infrastructure:

         {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

         {% include [explore-resources](../../_includes/mdb/terraform/explore-resources.md) %}

   {% endlist %}

1. [Connect to the VM](../../compute/operations/vm-connect/ssh.md#vm-connect) over SSH:

   ```bash
   ssh <username>@<VM_public_IP_address>
   ```

   Where `<username>` is the VM account username.

1. [Install the {{ yandex-cloud }} command line interface](../../cli/operations/install-cli.md#interactive) (YC CLI).
1. [Create a YC CLI profile](../../cli/operations/profile/profile-create.md#create).
1. [Install kubectl]({{ k8s-docs }}/tasks/tools/#kubectl).

## Check cluster availability {#check}

Run the following command on the VM:

```bash
kubectl cluster-info
```

The command will return the following {{ managed-k8s-name }} cluster information:

```text
Kubernetes control plane is running at https://<cluster_address>
CoreDNS is running at https://<cluster_address>/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

## (Optional) Connect a private Docker image registry {#cert}

You can connect a [private Docker image registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/) to your {{ managed-k8s-name }} cluster. To get authenticated in the registry and connect to it over HTTPS, the cluster will need certificates issued by the CA (Certificate Authority). Use the [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) controller to add and later automatically update the certificates on cluster nodes. It runs the following process in pods:

1. A Bash script constantly checks cluster nodes for required certificates.
1. If not, the certificates are copied from the {{ k8s }} [secret](https://kubernetes.io/docs/concepts/configuration/secret/) and updated.
1. The container runtime environment is restarted: containerd or Docker.

To configure certificate updates using DaemonSet, do the following on your VM:

1. Place the `.crt` certificate files.
1. Create a file named `certificate-updater-namespace.yaml` with the namespace configuration. This namespace will be used for DaemonSet operation and isolation.

   ```yaml
   apiVersion: v1
   kind: Namespace
   metadata:
     name: certificate-updater
     labels:
       name: certificate-updater
   ```

1. Create a file named `certificate-updater-daemonset.yaml` with the DaemonSet configuration:

   {% cut "File contents" %}

   ```yaml
   ---
   kind: NetworkPolicy
   apiVersion: networking.k8s.io/v1
   metadata:
     name: certificate-updater-deny-all
     namespace: certificate-updater
   spec:
     podSelector:
       matchLabels:
         k8s-app: certificate-updater
     policyTypes:
       - Ingress
       - Egress
     ingress: []
     egress:  []
   ---
   apiVersion: "apps/v1"
   kind: DaemonSet
   metadata:
     name: certificate-updater
     namespace: certificate-updater
     labels:
       k8s-app: certificate-updater
       version: 1v
   spec:
     selector:
       matchLabels:
         k8s-app: certificate-updater
     template:
       metadata:
         labels:
           k8s-app: certificate-updater
       spec:
         hostPID: true
         hostIPC: true
         containers:
         - name: certificate-updater
           image: cr.yandex/yc/mk8s-openssl:stable
           command:
             - sh
             - -c
             - |
               while true; do
                 diff -x '.*' -r /mnt/user-cert-path/ /usr/local/share/ca-certificates
                 if [ $? -ne 0 ];
                   then
                       echo "Removing all old certificates"
                       rm -r /usr/local/share/ca-certificates/*
                       echo "Copying certificates from configmap"
                       cp /mnt/sbin/update-ca-certificates /usr/sbin/
                       cp /mnt/user-cert-path/* /usr/local/share/ca-certificates

                       echo "Updating cerfificates authorities"
                       update-ca-certificates

                       echo "Restarting containerd and dockerd"
                       ps -x -o pid= -o comm= | awk '$2 ~ "^(containerd|dockerd)$" { print $1 }' | xargs kill
                       #systemd will get them back less than a minute
                   else
                     echo "Doing Nothing as no certs has not been changed"
                   fi
                 sleep 60
               done
           imagePullPolicy: Never
           securityContext:
             privileged: true
           resources:
             limits:
               memory: 200Mi
             requests:
               cpu: 100m
               memory: 200Mi
           volumeMounts:
           - mountPath: /etc/
             name: etc
           - mountPath: /usr/local/share/ca-certificates
             name: docker-cert
           - name: secret
             mountPath: /mnt/user-cert-path
           - name: sbin
             mountPath: /mnt/sbin
             readOnly: true
           - name: ca-cert
             mountPath: /usr/share/ca-certificates
         volumes:
         - name: secret
           secret:
             secretName: crt
         - name: sbin
           hostPath:
             path: /usr/sbin/
             type: Directory
         - name: ca-cert
           hostPath:
             path: /usr/share/ca-certificates
             type: Directory
         - name: docker-cert
           hostPath:
             path: /usr/local/share/ca-certificates
             type: DirectoryOrCreate
         - name: etc
           hostPath:
             path: /etc/
             type: Directory
   ```

   {% endcut %}

1. Create a namespace:

   ```bash
   kubectl apply -f certificate-updater-namespace.yaml
   ```

1. Create a secret with the contents of the certificates issued by the CA:

   ```bash
   kubectl create secret generic crt \
      --from-file=<path_to_certificate_file>.crt \
      --namespace="certificate-updater"
   ```

   Specify a certificate with the `.crt` extension in the command. If you need to add multiple certificates, provide each one in the command using the `--from-file` flag.

   You can check the secret configuration using the command below and see if it contains the information about the certificates:

   ```bash
   kubectl get secret crt -o yaml
   ```

1. Create a DaemonSet:

   ```bash
   kubectl apply -f certificate-updater-daemonset.yaml
   ```

Now you can monitor the state of the DaemonSet controller. As soon as the certificates are updated, the cluster will restart the container runtime environment processes (containerd or Docker).

## Delete the resources you created {#clear-out}

Some resources are not free of charge. Delete the resources you no longer need to avoid paying for them.

{% list tabs group=instructions %}

- Manually {#manual}

   Delete:

   1. [Service accounts](../../iam/operations/sa/delete.md)
   1. {{ kms-name }} [encryption key](../../kms/operations/key.md#delete)
   1. [Security groups](../../vpc/operations/security-group-delete.md)
   1. {{ managed-k8s-name }} [node group](../operations/node-group/node-group-delete.md)
   1. {{ managed-k8s-name }} [cluster](../operations/kubernetes-cluster/kubernetes-cluster-delete.md)
   1. [Virtual machine](../../compute/operations/vm-control/vm-delete.md)
   1. [Subnet](../../vpc/operations/subnet-delete.md)
   1. [Network](../../vpc/operations/network-delete.md)

- {{ TF }} {#tf}

   1. In the command line, go to the directory with the current {{ TF }} configuration file with an infrastructure plan.
   1. Delete the `k8s-cluster-with-no-internet.tf` and `virtual-machine-for-k8s.tf` configuration files.
   1. Make sure the {{ TF }} configuration is correct using this command:

      ```bash
      terraform validate
      ```

      If the configuration contains any errors, {{ TF }} will point them out.

   1. Confirm updating the resources.

      {% include [terraform-apply](../../_includes/mdb/terraform/apply.md) %}

      All the resources described in the `k8s-cluster-with-no-internet.tf` and `virtual-machine-for-k8s.tf` configuration files will be deleted.

{% endlist %}

If you assigned a static public IP address to your VM, [release and delete it](../../vpc/operations/address-delete.md).
