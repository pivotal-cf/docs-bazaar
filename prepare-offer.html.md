---
title: Preparing a Service Offering
owner: Platform Engineering (KSM Team)
---

<strong><%= modified_date %></strong>

This topic describes how to prepare an offered service for <%= vars.product_full %> (<%= vars.product_short %>).

##<a id='using'></a> Overview

<%= vars.product_short %> enables platform operators to offer Helm charts as
on-demand services to developers on <%= vars.app_runtime_full %> (<%= vars.app_runtime_abbr %>).
Before you add a service offering to `cf marketplace`,
you must configure your Helm charts to be compatible with <%=  vars.product_short %>.
You can also  customize your Helm charts to enable additional features.

A service offering includes the following:

+ **One or more Helm charts:** You must configure a Helm chart to compatible with
 <%=  vars.product_short %>. See [Configure Your Helm Charts](#config-helm) below.
 VMware recommends that you refer to the following resources when creating your Helm chart:
     + For information about creating your first Helm chart,
     see this topic in the [Bitnami documentation](https://docs.bitnami.com/kubernetes/how-to/create-your-first-helm-chart/).
     + For information about best practices for creating production-ready Helm charts,
     see this topic in the  [Bitnami documentation](https://docs.bitnami.com/tutorials/production-ready-charts/).
     + For more information about best practices,
     see the [Helm documentation](https://helm.sh/docs/topics/chart_best_practices/)

+ **A service offering directory that can include the following files:**
  + **`bind.yaml`:** This file is a template for app binding and service consumption.
  See [(Optional) Create Binding Template for App Consumption](prepare-offer.html#env) below.
  + **`ksm.yaml`  :** This file enables you to offer multiple Helm charts in a single
   service offering. You must include this file if you deploy multiple charts or use Operator patterns.
   See [(Optional) Offer Multiple Charts in a Single Offering](#multiple-charts) below.
   For information about Operators in Kubernetes,
   see [Operator pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator)
   in the Kubernetes documentation.
  + **`overrides` directory:** You use files in this directory to override the
     default values defined in the chart `values.yaml` file.
    See [(Optional) Override Chart Values](#override) below.
  + **`plans.yaml` and `plans` directory:** You use this file and the directory files to offer
  customized plans, override default values, and change the default cluster.
  See [(Optional) Define Plans Configuration](#plans) below.
  + **`cluster` directory:**
  You use the files in this directory to provide credentials for clusters.
  + **`resource-quotas` directory:** You use the files in this directory to
  define resource quotas that limit the memory and CPU that instances consume.
  See [(Optional) Define Plans Configuration](#plans) below.

The following code snippet shows an example of a service offering directory structure:

```yaml
service-name/
   bind.yaml
   ksm.yaml
   plans.yaml
   overrides/
     override-values.yaml
   plans/
     small.yaml
     medium.yaml
   clusters/
     cluster-creds.yaml
   resource-quotas/
     quota.yaml

helm-chart.tgz
```

###<a id='cf-marketplace'></a> Service Offerings in CF Marketplace

Uploading a Helm chart to <%= vars.product_short %> with either a `ksm.yaml` or `plans.yaml` file
overrides the default values for the service offering name, description, and plan names.
These values are displayed when a <%= vars.app_runtime_abbr %> developer runs `cf marketplace`.

The following table describes the values for your service that are used when you upload a Helm chart
without additional configurations, with a `ksm.yaml` file or a `plans.yaml` file:

<table>
  <tr>
    <th width="17%">Additional YAML file</th>
    <th width="24%">Name</th>
    <th>Description</th>
    <th>Plan Name</th>
  </tr>

  <tr>
    <td><strong>None</strong></td>
    <td><code>name</code> value in <code>Chart.yaml</code></td>
    <td><code>description</code> value in <code>Chart.yaml</code></td>
    <td><code>default</code></td>
  </tr>

  <tr>
    <td><strong><code>ksm.yaml</code></strong></td>
    <td><code>marketplace-name</code> value in <code>ksm.yaml</code></td>
    <td>No effect</td>
    <td>No effect</td>
  </tr>

  <tr>
    <td><strong><code>plans.yaml</code></strong></td>
    <td>No effect</td>
    <td>For each plan, <code>description</code> value in <code>plans.yaml</code></td>
    <td>For each plan, <code>name</code> value in <code>plans.yaml</code></td>
  </tr>

</table>

##<a id='prereq'></a> Prerequisites

Before you save your service offering to <%= vars.product_short %, you must:

+ **Install the <%= vars.product_short %> (Command Line Interface) CLI:**
See [Install the <%= vars.product_short %> CLI](installing.html#ksm).
+ **Register a Kubernetes cluster using the <%= vars.product_short %> CLI:**
See [Managing Kubernetes Clusters for Services](managing-clusters.html).
+ **Install External Dependencies:** If your Helm charts have any external dependencies,
they must be installed before you offer your service.
For example, if your Helm chart requires an Ingress controller to enable service through ingress,
    the Ingress controller must be manually installed onto the cluster. For information
    about Ingress, see [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
    in the Kubernetes documentation.
+ **Load Container Images:** You might have to load your container images into a private
container registry.
See [Load Container Images into a Private Container Registry](#images) below.


##<a id='offer-init'></a> Create a Sample Service Offering Directory

You can use the <%= vars.product_short %> CLI to create a directory structure for your service offering.
When you create the service offering directory with the <%= vars.product_short %> CLI,
the directory includes the following sample files:

+ `SERVICE-NAME/bind.yaml`
+ `SERVICE-NAME/ksm.yaml`
+ `SERVICE-NAME/plans.yaml`
+ `SERVICE-NAME/plans/small.yaml`
+ `SERVICE-NAME/plans/medium.yaml`

To create a sample service offering directory:

1. Create a directory for your service offering by running:

    ```
    ksm offer init SERVICE-NAME
    ```
    Where `SERVICE-NAME` is the name of the service offering.


##<a id='config-helm'></a> Configure Your Helm Charts

To upload a Helm chart to <%= vars.product_short %>,
you must ensure that the chart is compatible with <%= vars.product_short %>.
For examples of <%= vars.product_short %>-compatible charts,
see [ksm-sample](https://github.com/cf-platform-eng/ksm-sample) on GitHub.

<p class="note"><strong>Note:</strong> If you do not include a <code>plans.yaml</code>,
  a <code>ksm.yaml</code> file, or override files with your service offering, one plan is offered.
This plan uses the default values for the Helm chart.
</p>

To configure your Helm chart:

1. Override values in the chart `values.yaml` file.
  See [Override Chart Values](#override).
1. If you are using Helm 2 charts, define Custom Resource Definitions (CRDs) using
    Helm 2 and Helm 3 methods.
    See [Define Custom Resource Definitions](#crd).

    <p class="note"><strong>Note:</strong> Helm 3 handles CRDs differently from other Kubernetes Resources.
    For information about best practices for using CRDs, see
    the <a href="https://helm.sh/docs/chart_best_practices/custom_resource_definitions/">Helm documentation</a>.
  </p>

###<a id='override'></a> Override Chart Values
To ensure that your chart is compatible with <%= vars.product_short %>,
you might need to override specific values in <code>values.yaml</code>.

You must follow the procedure in this section if your chart uses any of the following properties:

+ storageClass
+ services.type
+ hosts.name

You can optionally override other properties defined the chart `values.yaml` by
adding those properties to an `OVERRIDE-FILE.yaml` file.

<p class="note warning"><strong>Warning:</strong> You cannot change the chart name or
   resize a PersistentVolumeClaim after you add the service offering to <%= vars.product_short %>.
</p>

To override chart values:

1. Create an `overrides` directory and an `overrides/OVERRIDE-FILE.yaml` file. <br><br>
    Ensure that the properties you add to `OVERRIDE-FILE.yaml` match the properties
    in `values.yaml`.
    Do not change the names of the properties.

1. Edit your override file to configure storage. Edit the `storageClass:` value to correspond to
the storageClass name for your configured <%= vars.k8s_runtime_full %> cluster.
The default name for <%= vars.k8s_runtime_abbr %> is `standard`.
For more information, see [Configuring and Using PersistentVolumes](https://docs.pivotal.io/runtimes/pks/volumes.html).
<br><br>For example:

    ```yaml
    ...
    persistence:
      enabled: true
      storageClass: "standard"
      ...
    ```

1. Edit your override file to enable service access.
<%= vars.product_short %> supports using a load balancer, Ingress, or ExternalDNS to access services.
    + **If want your service to be accessed directly using a load balancer,** set
    `services.type:` to `LoadBalancer`. For example:

        ```yaml
        ...
        service:
          type: LoadBalancer
          port: 3306
        ...
        ```
    + **If want your service to be accessed using Ingress or ExternalDNS:**
    Edit the value of the `hosts.name` to include a templatized value for uniqueness.
    For example:

        ```yaml
        ...
        ingress:
          enabled: true
          hosts:
          - name: "{{ .Release.Name }}.example.com"
        ...
        ```

    For more information, see [(Optional) Template Values](#values-templating) below.

    <p class="note"><strong>Note:</strong> The Ingress controller or ExternalDNS
      controller need to be installed and configured manually. <br><br>For information
      about Ingress, see the
      <a href="https://kubernetes.io/docs/concepts/services-networking/ingress/">Kubernetes documentation</a>.
      For information about
      ExternalDNS, see <a href="https://github.com/kubernetes-incubator/external-dns">external-dns</a> in GitHub.
    </p>
1. Edit the `ksm.yaml` file to use the override file you created in step 1.<br><br>
For example:

    ```yaml
    marketplace-name: SERVICE-NAME
    charts:
      - chart: CHART-NAME
        version: CHART-VERSION
        offered: OFFERED-VALUE
        scope: SCOPE-VALUE
        overrideValuesFile: OVERRIDE-FILE
    ```
    For information about the properties used in `ksm.yaml`,
    see [(Optional) Offer Multiple Charts in a Single Offering](#multiple-charts) below.

### <a id='crd'></a> Define Custom Resource Definitions

<%= vars.product_short %> supports charts that use the Helm 2 and Helm 3 CLIs.
Helm 2 and Helm 3 use different methods to install CRDs.
 <%= vars.product_short %> requires Helm 3, while Helm 2 is optional.

If you are using a Helm 2 chart, use both the Helm 2 and Helm 3 CLIs methods:

1. Define your CRDs using the Helm 3 method by referring to the
   [Helm 3 documentation](https://helm.sh/docs/topics/chart_best_practices/custom_resource_definitions/#install-a-crd-declaration-before-using-the-resource).
   Helm 3 uses a `crds` directory to install CRDs.

1. Define your CRDs using the Helm 2 method by referring to the
[Helm 2 documentation](https://v2.helm.sh/docs/charts_hooks/#defining-a-crd-with-the-crd-install-hook).
Helm 2 uses a hook annotation to install CRDs.


##<a id='plans'></a> (Optional) Define Plans Configuration

<p class="note"><strong>Note:</strong> The properties in <code>plans.yaml</code> and
  <code>PLAN-FILE.yaml</code> files override configurations in
  override files.
  For information about override files, see <a href="#override">Override Chart Values</a> above.
</p>

In `cf marketplace`, each service has a set of plans that developers can provision.
A plan is a template for service instances.
For example, different plans might represent a large or a small instance of a database.

For Kubernetes services, each <%= vars.product_short %> plan represents a set of values overriding
the default values in the Helm chart.
These values are configured in `plans.yaml`.
For examples of `plans.yaml` files,
see [kibosh-sample](https://github.com/cf-platform-eng/kibosh-sample/blob/master/sample-charts) on GitHub.

If you want to override the default settings in `values.yaml` and offer custom plans:

1. Create and edit `plans/PLAN-FILE.yaml` with the Helm values that you want to override.
   `PLAN-FILE.yaml` must consist of only lowercase letters, digits, `.`, or `-`.
    <br><br>
    Ensure that the properties you add to `PLAN-FILE.yaml` match the properties
    in `values.yaml` and that you only change the values.
    <br><br>
    For example:

    ```
    ---
    resources:
      requests:
        memory: 128Mi
        cpu: 100m
    ```
1. (Optional) If you want to use a nondefault cluster for a plan,
    create a `clusters` subdirectory in the directory for your service offering.
    For example, you could create a plan that enables all instances created
    with that plan to run on their own cluster.

1. (Optional) Create and edit `clusters/CLUSTER-CREDENTIALS-FILE.yaml`
    with the credentials for the alternate cluster you want to use.
    <br><br>
    For example:

    ```
    ---
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data: aBcDefG1HIJ== ....
        server: https://127.0.0.1:8443
      name: my-cluster
    contexts:
    - context:
        cluster: my-cluster
        user: my-user
      name: my-cluster
    current-context: my-cluster
    kind: Config
    preferences: {}
    users:
    - name: my-user
      user:
        token: kLmNoP2qrSt3= ...
    ```

1. (Optional) Create and edit `resource-quotas/RESOURCE-QUOTA-FILE.yaml`
    with the `ResourceQuota` definition you want to use.
    This `ResourceQuota` is created in each Kubernetes namespace where a service instance is provisioned.
    For more information about `ResourceQuota` objects, see the
    [Kubernetes documentation](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/).
    <br><br>
    For example:

    ```
    ---
    apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: mem-cpu-demo
    spec:
      hard:
        requests.cpu: "1"
        requests.memory: 1Gi
        limits.cpu: "2"
        limits.memory: 2Gi
    ```

1. Edit your `plans.yaml` file to add your custom plan information using the following template:

    ```
    - name: PLAN-NAME
      description: PLAN-DESCRIPTION
      file: PLAN-FILE
      credentials: CLUSTER-CREDENTIALS-FILE
      resourceQuotaPath: RESOURCE-QUOTA-FILE
    ```

    Where:
    * `PLAN-NAME` is the name of the plan that developers see when they run `cf marketplace`
        The plan name is listed under `plans`.
       Ensure that the plan name is lowercase only and contains no special characters.
    * `PLAN-DESCRIPTION` is the description of the plan that developers see when they run `cf marketplace`.
        The plan description is listed under `description`.
    * `PLAN-FILE` is the name of the plan file in the `plans` subdirectory.
    * (Optional) `CLUSTER-CREDENTIALS-FILE` is name of the cluster credentials file for the plan.
    * (Optional) `RESOURCE-QUOTA-FILE` is name of the resource quota file for the plan.

    For example:

    ```
    - name: "small"
      description: "default (small) plan for mysql"
      file: "small.yaml"
      resourceQuotaPath: "quota.yaml"
    - name: "medium"
      description: "medium sized plan for mysql"
      file: "medium.yaml"
      credentials: "cluster.yaml"
    ```

<p class="note"><strong>Note:</strong> Developers can override plan and Helm chart values
  using the <code>-c</code> flag when they provision or update a service instance with the cf CLI.
  They can use the <code>-c</code> flag to provide service-specific configuration parameters. <br><br>
  When developers provide these configuration parameters,
  the JSON configurations are mapped directly onto the chart
  values and take precedence over any chart default and plan values.
  For information about service-specific configuration parameters,
  see <a href="https://docs.pivotal.io/platform/application-service/devguide/services/managing-services.html#-arbitrary-parameters">
  Arbitrary Parameters</a>.
</p>


##<a id='multiple-charts'></a> (Optional) Offer Multiple Charts in a Single Offering

<%= vars.product_short %> enables you to offer multiple Helm charts in a single service offering.
You might want to offer multiple Helm charts in the following cases:

* A cluster-scoped operator or controller where the individual charts are custom resources
* A namespace-scoped operator chart with another chart that represents the custom resource
* A service that uses multiple charts.
For example, a service that uses a data service chart and a monitoring service chart.

To offer a service with multiple Helm charts, you must edit the `ksm.yaml` file.
The `charts` section in a `ksm.yaml` file is an ordered list of the charts that are included
in the service offering.

1. Edit the `ksm.yaml` file directory using the following as a template:

    ```yaml
    marketplace-name: SERVICE-NAME
    charts:
      - chart: CHART-NAME
        version: CHART-VERSION
        offered: OFFERED-VALUE
        scope: SCOPE-VALUE
      - chart: CHART-NAME
        version: CHART-VERSION
        offered: OFFERED-VALUE
        scope: SCOPE-VALUE
    ```

    Where:
    + `SERVICE-NAME` is the name of the service that developers see when they run `cf marketplace`.
        The service name is listed under `service`.
    + `CHART-NAME` is the name of a chart that you are including in the service offering.
    + `CHART-VERSION` is the version of the chart that you are including in the service offering.
    + `OFFERED-VALUE` is set to either `true` or `false`. For more information, see the below table.
    + `SCOPE-VALUE` is set to either `cluster` or `namespace`. For more information, see the below table.

The following table describes the Helm chart configurations in `ksm.yaml`:

<table>
  <tr>
    <th width=14%>Key</th>
    <th width=21% >Values</th>
    <th>Description</th>
  </tr>

  <tr>
    <td><strong><code>scope</code></strong></td>
    <td><code>cluster</code></td>
    <td>If you use this value, the chart is only deployed once on a cluster.
    Subsequent service instances created on the same Kubernetes cluster
    do not cause a cluster-scoped chart to be deployed additional times.<br><br>
    You can use this scope for operators and controllers that manage multiple
    instances from a single deployment.</td>
  </tr>

  <tr>
    <td></td>
    <td><code>namespace</code></td>
    <td>If you use this value, the chart is deployed on the cluster every time
      the service is created.</td>
  </tr>

  <tr>
    <td><strong><code>offered</code></strong></td>
    <td><code>true</code> | <code>false</code></td>
    <td>Only a single chart can have <code>offered</code> to <code>true</code>.
      The chart that is set to <code>true</code> must also have
      <code>scope</code> set to <code>namespace</code>. <br><br>
      The offered chart represents the service the user binds to.
      However, you can expose more resources with <code>cf bind-service</code>.</td>

  </tr>

</table>

##<a id='env'></a> (Optional) Create Binding Template for App Consumption

Apps running in <%= vars.app_runtime_abbr %> gain access to bound service instances through an environment
variable credentials hash called `VCAP_SERVICES`.
Libraries such as Spring Cloud
Cloud Foundry Connector automatically support services if their credentials in
`VCAP_SERVICES` are formatted in a valid JSON object.

For an example of the requirements for using Spring Cloud
Cloud Foundry Connector with a MySQL service, see the
[Spring Cloud Cloud Foundry Connector documentation](https://cloud.spring.io/spring-cloud-connectors/spring-cloud-cloud-foundry-connector.html#_mysql).

Chart authors can edit the `bind.yaml` file to enable the broker to
return a valid JSON object containing service credentials.
This `bind.yaml` file contains a Jsonnet template.
When a developer binds the service instance to their app,
service credentials are delivered to `VCAP_SERVICES` for app consumption.

For information about Jsonnet templates, see [Jsonnet](https://jsonnet.org).

To edit and test the `bind.yaml` file:

1. Edit the `bind.yaml` file. <br><br>
    For example, the following `bind.yaml` enables the broker to return
    MySQL credentials in a valid JSON obejct:

    ```yaml
    template: |
      {
        hostname: $.services[0].status.loadBalancer.ingress[0].ip,
        name: $.services[0].name,
        jdbcUrl: "jdbc:mysql://" + self.hostname + "/my_db?user=" + self.username + "&password=" + self.password + "&useSSL=false",
        uri: "mysql://" + self.username + ":" + self.password + "@" + self.hostname + ":" + self.port + "/my_db?reconnect=true",
        password: $.secrets[0].data['mysql-root-password'],
        port: 3306,
        username: "root"
      }
    ```
    <p class="note"><strong>Note:</strong> The values of <code>$.services</code>, <code>$.secrets</code>
    and <code>$.ingresses</code> are from the namespace where the chart is deployed.
      </p>

1. Test that `bind.yaml` is creating a valid JSON object by running:

    ```
    ksm test-bind-template NAMESPACE bind.yaml
    ```

    Where `NAMESPACE` is the name of the namespace in your cluster where
    your chart is deployed.
    <br><br>
    For example:
    <pre class="terminal">$ ksm test-bind-template default bind.yaml
    Services, secrets, and ingress:
    {
        "secrets": [
            {
                "data": {
                    "mysql-password": "aBcDefG1HIJ",
                    "mysql-root-password": "kLmNoP2qrSt3"
                },
                "name": "1-23456789-mysql"
            },
        ],
        "services": [
            {
                "metadata": {
                    "name": "1-23456789-mysql",
                    "namespace": "ksm-f5d531d5-7b07-4bc0-8114-8aba5e2bd706",
                    "uid": "9f2d94c1-4900-11ea-a03a-42010a800053",
                    "resourceVersion": "4943530",
                    "creationTimestamp": "2020-02-06T16:49:20Z",
                    "labels": {
                        "app": "1-23456789-mysql",
                        "heritage": "Helm",
                        "release": "1-23456789"
                    }
                },
                "name": "1-23456789-mysql",
                "spec": {
                    "ports": [
                        {
                            "name": "mysql",
                            "protocol": "TCP",
                            "port": 3306,
                            "targetPort": "mysql",
                            "nodePort": 32257
                        }
                    ],
                    "selector": {
                        "app": "1-23456789-mysql"
                    },
                    "clusterIP": "10.24.2.216",
                    "type": "LoadBalancer",
                    "sessionAffinity": "None",
                    "externalTrafficPolicy": "Cluster"
                },
                "status": {
                    "loadBalancer": {
                        "ingress": [
                            {
                                "ip": "33.44.55.66"
                            }
                        ]
                    }
                }
            }
        ]
    }
    Rendered template {
       "hostname": "33.44.55.66",
       "jdbcUrl": "jdbc&#58;mysql&#58;//33.44.55.66/my&#95;db?user=root&password=kLmNoP2qrSt3&useSSL=false",
       "name": "1-23456789-mysql",
       "password": "kLmNoP2qrSt3",
       "port": 3306,
       "uri": "mysql&#58;//root&#58;kLmNoP2qrSt3&#64;33.44.55.66:3306/my_db?reconnect=true",
       "username": "root"
    }</pre>


###<a id='values-templating'></a> (Optional) Template Values

Helm uses the literal contents of the `values.yaml` file
as the concrete set of substitutions when creating a release.
However, <%= vars.product_short %> uses `values.yaml` as a template for multiple releases.

This difference can create cases where `values.yaml`
needs to go through the Helm templating engine before
any template files in the `/templates` directory are rendered with those values.

`values.yaml` might need to go through the Helm templating engine in the following cases:

* When a chart uses Ingress controller. For information, see
[Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
in the Kubernetes documentation.
* When there is a unique name, in addition to the release name, required in a chart.

<%= vars.product_short %> runs the values through the Helm templating engine.
For example, in a chart with ingress hosts,
you can edit `values.yaml` to use the release name as
the unique subdomain discriminator in the ingress definition:

```yaml
...
ingress:
  enabled: true
  hosts:
  - name: "{{ .Release.Name }}.example.com"
...
```

The plan-specific values files also go through the Helm templating engine.

##<a id='images'></a> Load Container Images into a Private Container Registry

VMware recommends using a private container registry in production.
<%= vars.product_short %> can then modify your Helm charts to point to images in the private
container registry.

If you either of the following applies, you must configure a private
container registry:

+ Your environment is air-gapped and cannot connect to public container registries such as
Docker Hub or Quay.
For information about commonly used public container registries,
see [Docker Hub](https://hub.docker.com/), [Harbor](https://goharbor.io/), and [Quay](https://quay.io/).
+ The images referenced in the Helm chart are not public.


If you do not load your container images in to a private container registry,
a developer cannot create a service instance
offered on <%= vars.product_short %>.

To load your container images:

1. Load your container images by following the instructions for your specific container registry.
For example, if you use Harbor, follow the instructions in
[Pulling and pushing images using Docker client](https://github.com/goharbor/harbor/blob/master/docs/user_guide.md#pulling-and-pushing-images-using-docker-client)
in GitHub.


1. Confirm that your `values.yaml` file uses `image`, `images`, or `global.imageRegistry` keys.
   If you use the keys above, <%= vars.product_short %> always
   uses the private registry configured in the tile to fetch your images.

    For example:
    * If you use a chart with a single image:

        ```yaml
        ---
        image: "my-image"
        imageTag: "5.7.14"
        ```
    * If you use a chart with multiple images:

        ```yaml
        ---
        images:
          component1:
            image: "my-first-image"
            imageTag: "5.7.14"
          component2:
            image: "my-second-image"
            imageTag: "1.2.3"
        ```
    * If you use a chart with a global.imageRegistry key:

        ```yaml
        ---
        global:
          imageRegistry: docker.example.com/image-registry
        ```


    If the Helm chart uses a key other than an `image`, `images` or `global.imageRegistry` key,
    then <%= vars.product_short %> does not change
    the image reference used to pull from the private container registry.

1. Configure your private container registry in the <%= vars.product_short %> tile and apply changes.
See [Configure Private Container Registry](./installing.html#container).