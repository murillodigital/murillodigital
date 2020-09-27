---
title: "The Open Application Model and Crossplane - Part 2"
description: "An agnostic way to build application centric platforms and infrastructure in a team centric way, across clouds"
date: "2020-09-26"
featured_image: "/images/crossplane_oam/oam_hero.jpg"
author: "Leonardo Murillo"
images:
- "/images/crossplane_oam/oam_hero.jpg"
---
# Let's get experimenting

Welcome to Part 2 of my series on the Open Application Model and Crossplane. If you just got here and want to learn what OAM and CrossPlane are, start with [part 1]({{< ref crossplane >}}) of this series for an overview of both.

In this experiment we will take Gooogle's "Online Boutique" and transform one of its services into a OAM / Crossplane framework based solution!

The original version on the sample solution can be found in [this GitHub repo](https://github.com/GoogleCloudPlatform/microservices-demo) but we will be working on a forked and modified version of it which you can [find in my forked copy of the repo here](https://github.com/murillodigital/microservices-demo). 

# A little bit of story

Before we dive into the technical, I want to share with you a quick story about my experience experimenting with CrossPlane and OAM.

It took me a lot longer to get this solution working than I had anticipated - of course I dedicate an hour or so a day at most to this type of endeavour, nevertheless, finding information and troubleshooting problems took considerable effort mostly due to the relatively still immature amount of documentation and documented "experiences" out there.

Things go rather smoothly when you follow the documented process, but once you start mixing things up and run into issues it took me quite a bit of googling and relatively fragmented documentation reading to get to the bottom of things.

*But there is a community!* When you get to the point of reaching out to the community usually means you've gotten to a point of resource exhaustion, in other words, you don't know where else to look. I hit that point trying to get connection data to propagate to the right namespace, an issue that blocked me for over a week.

And that's the beauty of open source: there is a community to reach out to, and there's no better help than that which is given out of technical curiosity and the will to help one another.

I'd like to give a shout out to [Krish Chowdhary](https://www.linkedin.com/in/krishchow), a Software Engineer with the [RedHat Emerging Technologies](https://next.redhat.com/) team that, in the spirit of community, shared his own experimentation and pointed me in the right direction.

# Our starting point

Google has put together a very useful project to demonstrate how solutions built using a microservice architecture actually work. It has evolved over multiple iterations and in its latest version it is known as "Online Catalog".

It's a fantastic resource for training and experimentation. The solution is super diverse, with services written in five different languages! This fully containerized solution is ready for deployment onto a Kubernetes cluster using Skaffold. 

Microservices communicate with one another using gRPC and each one gets deployed as a Kubernetes `Deployment` with a `Service` in front of it.

The source code for each service can be found in a subdirectory inside the `src/` directory at the root of the repository:

```bash
lmurillo@lmurillo-eliteubuntu:~/development/workspaces/microservices-demo/src$ ls -gG
total 44
drwxrwxr-x 4 4096 Sep  7 11:35 adservice
drwxrwxr-x 7 4096 Sep  7 11:35 cartservice
drwxrwxr-x 4 4096 Sep  7 11:35 checkoutservice
drwxrwxr-x 4 4096 Sep  7 11:35 currencyservice
drwxrwxr-x 3 4096 Sep  7 11:35 emailservice
drwxrwxr-x 6 4096 Sep  7 11:35 frontend
drwxrwxr-x 2 4096 Sep  7 11:35 loadgenerator
drwxrwxr-x 3 4096 Sep  7 11:35 paymentservice
drwxrwxr-x 3 4096 Sep  7 11:35 productcatalogservice
drwxrwxr-x 2 4096 Sep  7 11:35 recommendationservice
drwxrwxr-x 3 4096 Sep  7 11:35 shippingservice
```

In this experiment we are specifically not going to modify the source code of the services, considering one of the objectives is to demonstrate the implementation of OAM patterns without impacting existing application code.

This is important since, were there a need to modify the applications, the effort and risk of adopting the Open Application Model would dramatically increase.

Kubernetes manifests defining each service can be found inside the aptly named directory `kubernetes-manifests`:

```bash
lmurillo@lmurillo-eliteubuntu:~/development/workspaces/original-microservices-demo/kubernetes-manifests$ ls -gG
total 52
-rw-rw-r-- 1  317 Sep 27 08:54 README.md
-rw-rw-r-- 1 1826 Sep 27 08:54 adservice.yaml
-rw-rw-r-- 1 1772 Sep 27 08:54 cartservice.yaml
-rw-rw-r-- 1 2305 Sep 27 08:54 checkoutservice.yaml
-rw-rw-r-- 1 1739 Sep 27 08:54 currencyservice.yaml
-rw-rw-r-- 1 1692 Sep 27 08:54 emailservice.yaml
-rw-rw-r-- 1 2900 Sep 27 08:54 frontend.yaml
-rw-rw-r-- 1 1272 Sep 27 08:54 loadgenerator.yaml
-rw-rw-r-- 1 1545 Sep 27 08:54 paymentservice.yaml
-rw-rw-r-- 1 1833 Sep 27 08:54 productcatalogservice.yaml
-rw-rw-r-- 1 1900 Sep 27 08:54 recommendationservice.yaml
-rw-rw-r-- 1 1541 Sep 27 08:54 redis.yaml
-rw-rw-r-- 1 1791 Sep 27 08:54 shippingservice.yaml
```

We are going to be using the `cartservice` as our example because of its dependence on Redis.

# The objectives

To demonstrate a OAM scenario, these will be our objectives:

- Replace Redis as a service deployment inside the cluster with a published cloud managed resource
- Turn `cartservice` into an OAM component, that can be deployed by an application operator inside an `ApplicationConfiguration`

## Step No. 0 - create a new structure for K8s manifests

For demonstration purposes, I'm going to create three subdirectories inside the `kubernetes-manifests/` directory that reflect the three roles in a usual OAM workflow.

```bash
lmurillo@lmurillo-eliteubuntu:~/development/workspaces/microservices-demo/kubernetes-manifests$ ls -gGd */
drwxr-xr-x 2 4096 Sep 25 11:21 application-developer/
drwxr-xr-x 3 4096 Sep 25 10:47 application-operator/
drwxr-xr-x 2 4096 Sep 25 10:38 platform-builders/
```

In actual scenarios, these different manifests may be located in different repositories, and they may be managed by different teams.

## Step No. 1 - Platform builder resources

We will start working inside the `platform-builders` directory, this is where we will place all manifests for creating and publishing the platform components that will provide cloud managed resources, provided by CrossPlane and OAM, to teams that require them when building their solutions. 

### ContainerizedWorkload workload definition and traits

In its original version, both `cartservice` and `redis` are `Deployment` kind of objects, we will be turning them into OAM object kinds.

There are two main benefits of using the `ContainerizedWorkload` object as opposed to the native `Deployment` kind:

1. Abstraction: The `ContainerizedWorkload` kind "hides" a lot of the complexity of a deployment, and provides a layer of abstraction between development and deployment.
2. Configuration: By using a `ContainerizedWorkload` you enable application operators to use `Traits` to modify specific parameters of the solution during release.

As per `redis`, since we will now be providing this service as a managed cloud resource via CrossPlane, we will also create a WorkloadDefinition that refers to the custom Redis requirement definition that we will be publishing. `redisinstancerequirements.infrastructure.murillodigital.com` is a CRD that CrossPlane will create for us as we define and publish our custom infrastructure.

In order for us to enable this transformation, we of course need to have the necessary OAM resources deployed to the cluster. For that purpose we've added two definitions:

#### workloads.yaml
```yaml
---
apiVersion: core.oam.dev/v1alpha2
kind: WorkloadDefinition
metadata:
  name: containerizedworkloads.core.oam.dev
spec:
  definitionRef:
    name: containerizedworkloads.core.oam.dev
---
apiVersion: core.oam.dev/v1alpha2
kind: WorkloadDefinition
metadata:
  name: redisinstancerequirements.infrastructure.murillodigital.com
spec:
  definitionRef:
    name: redisinstancerequirements.infrastructure.murillodigital.com
```

And of course  we also need to deploy the `TraitDefinition` that the Application Operator will use to configure the release of the `cartservice` service.

#### traits.yaml
```yaml
---
apiVersion: core.oam.dev/v1alpha2
kind: TraitDefinition
metadata:
  name: manualscalertraits.core.oam.dev
spec:
  definitionRef:
    name: manualscalertraits.core.oam.dev
```

### Publishing a managed Redis resource

We will be using CrossPlane to publish a `RedisInstance` custom resource definition, managed by CrossPlane. To accomplish this we need to create three different resources:

- A `InfrastructureDefinition`: This definition will declare the name and characteristics of our `RedisInstance` custom resource.

```yaml
apiVersion: apiextensions.crossplane.io/v1alpha1
kind: InfrastructureDefinition
metadata:
  name: redisinstances.infrastructure.murillodigital.com
spec:
  connectionSecretKeys:
    - endpoint
    - port
  crdSpecTemplate:
    group: infrastructure.murillodigital.com
    version: v1alpha1
    names:
      kind: RedisInstance
      listKind: RedistInstanceList
      plural: redisinstances
      singular: redisinstance
    validation:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              parameters:
                type: object
                properties:
                  tier:
                    type: string
                  storage:
                    type: integer
                required:
                  - tier
                  - storage
            required:
              - parameters
```

CrossPlane will automatically create a CRD using the template defined in `crdSpecTemplate` and use name.Kind property as name, in this case we're defining a `RedisInstance` CRD that requires two properties defined, `tier` and `storage` which we will pass to the underlying managed service resource when we create a Composition that "maps" this kind to a CrossPlane managed cloud resource.

Another important detail here is the `connectionSecretKeys` list. One of the more convenient (and secure) features of having CrossPlane provision your cloud resources, is the automated way in which it manages connection details for you, to enable and use this feature, you must specify which keys you will be using for connection details from the underlying CrossPlane object. To understand which connection keys are exposed by each cloud provider resource, refer to the [API documentation](https://crossplane.io/docs/v0.12/api-docs/overview.html) for your provider of choice.

- A `InfrastructurePublication`: Defining a specific type of custom infrastructure as we did does not mean this new type of resource will automatically be referable for consumption. Before we can use it, it must be published:

```yaml
apiVersion: apiextensions.crossplane.io/v1alpha1
kind: InfrastructurePublication
metadata:
  name: redisinstances.infrastructure.murillodigital.com
spec:
  infrastructureDefinitionRef:
    name: redisinstances.infrastructure.murillodigital.com
``` 

- A `Composition`: The composition is used by CrossPlane to map your custom resource to a Cloud Provided resource, in our case we are "mapping" our RedisInstance custom resource to a GCP `CloudMemoryStoreInstance` CrossPlane resource.

> Pay special attention to the composition labels, your application developers can use them to choose which specific composition they want to use, to satisfy a specific service requirement.

```yaml
apiVersion: apiextensions.crossplane.io/v1alpha1
kind: Composition
metadata:
  name: redisinstances.gcp.infrastructure.murillodigital.com
  labels:
    provider: gcp
    experiment: murillodigital
spec:
  writeConnectionSecretsToNamespace: crossplane-system
  reclaimPolicy: Delete
  from:
    apiVersion: infrastructure.murillodigital.com/v1alpha1
    kind: RedisInstance
  to:
    - base:
        apiVersion: cache.gcp.crossplane.io/v1beta1
        kind: CloudMemorystoreInstance
        spec:
          forProvider:
            tier: STANDARD_HA
            region: us-east1
            memorySizeGb: 1
          providerRef:
            name: gcp-provider
          reclaimPolicy: Delete
          writeConnectionSecretToRef:
            name: redis-secret
            namespace: crossplane-system
      patches:
        - fromFieldPath: "spec.parameters.secretName"
          toFieldPath: "spec.writeConnectionSecretToRef.name"
        - fromFieldPath: "spec.parameters.secretNamespace"
          toFieldPath: "spec.writeConnectionSecretToRef.namespace"
        - fromFieldPath: "spec.parameters.tier"
          toFieldPath: "spec.forProvider.tier"
        - fromFieldPath: "spec.parameters.storage"
          toFieldPath: "spec.forProvider.memorySizeGb"
      connectionDetails:
        - fromConnectionSecretKey: endpoint
        - fromConnectionSecretKey: port

```

> Pay attention to your patches. These items will define, when using this composition, which parameters from your Component will be applied as overrides to resources being created by your Composition.

## Step No. 2 - Application Developer Resources

Now that our Platform Builder team has created and published all managed resources that our Application Developers will need for their `cartservice`, we're going to rewrite the cart service definition to match.

As you learned in [part 1]({{< ref crossplane >}}), application developers create components, which represent the functional units (eg. services) of their solution.

Since we got rid of our `redis.yaml` manifest now we will add Redis as a component of our solution, and will convert the `cartservice` service into a `ContainerizedWorkload`.

```yaml
---
apiVersion: core.oam.dev/v1alpha2
kind: Component
metadata:
  name: cartservice-redis-component
spec:
  workload:
    apiVersion: infrastructure.murillodigital.com/v1alpha1
    kind: RedisInstanceRequirement
    metadata:
      name: cartservice-redis-req
    spec:
      parameters:
        tier: STANDARD_HA
        storage: 1
      compositionSelector:
        matchLabels:
          experiment: murillodigital
  parameters:
    - name: secretName
      required: true
      fieldPaths:
        - spec.writeConnectionSecretToRef.name
    - name: secretNamespace
      required: true
      fieldPaths:
        - spec.writeConnectionSecretToRef.namespace
    - name: provider
      required: true
      fieldPaths:
        - spec.compositionSelector.matchLabels.provider
---
apiVersion: core.oam.dev/v1alpha2
kind: Component
metadata:
  name: cartservice-component
spec:
  workload:
    apiVersion: core.oam.dev/v1alpha2
    kind: ContainerizedWorkload
    metadata:
      name: cartservice-workload
    spec:
      containers:
      - name: server
        image: sjodevops/cartservice:v0.1.4-33-g0d635b9
        ports:
        - containerPort: 7070
          name: cartsvcgrpc
        env:
          - name: REDIS_ENDPOINT
            fromSecret:
              key: endpoint
          - name: REDIS_PORT
            fromSecret:
              key: port
          - name: REDIS_ADDR
            value: "$(REDIS_ENDPOINT):$(REDIS_PORT)"
          - name: PORT
            value: "7070"
          - name: LISTEN_ADDR
            value: "0.0.0.0"
        readinessProbe:
          initialDelaySeconds: 15
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:7070", "-rpc-timeout=5s"]
        livenessProbe:
          initialDelaySeconds: 15
          periodSeconds: 10
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:7070", "-rpc-timeout=5s"]
  parameters:
    - name: secretName
      required: true
      fieldPaths:
        - spec.containers[0].env[0].fromSecret.name
        - spec.containers[0].env[1].fromSecret.name
...
```

Note how our `cartservice-redis-component` points to a kind of workload named `RedisInstanceRequirement`, which is a CRD automatically created by CrossPlane that we use to indicate that we need a RedisInstance created to satisfy the requirement.

Also note that no `RedisInstance` nor `ContainerizedWorkload` will be created simply by applying these objects, they will need to be used by our application operator in the context of an `ApplicationConfiguration`. And this takes us to our final step, the application operator actually releasing this as a runtime solution.

## Step No. 3 - Application Operator Resources

The application operator will now use the components defined by the application developers to provision a configured release of the solution. We are going to create a single `ApplicationConfiguration` that ties together both Components into a single unit, and we will use `Traits` to define runtime parameter of our `ContainerizedWorkload`

```yaml
apiVersion: core.oam.dev/v1alpha2
kind: ApplicationConfiguration
metadata:
  name: online-catalog
spec:
  components:
    - componentName: cartservice-redis-component
      parameterValues:
        - name: secretName
          value: online-catalog-redis-secret
        - name: secretNamespace
          value: default
        - name: provider
          value: gcp
    - componentName: cartservice-component
      parameterValues:
        - name: secretName
          value: online-catalog-redis-secret
      traits:
        - trait:
            apiVersion: core.oam.dev/v1alpha2
            kind: ManualScalerTrait
            metadata:
              name: cartservice-component
            spec:
              replicaCount: 2
```

And with this we are done, now the application operator can go and apply the `ApplicationConfiguration` and this will create all resources, including those that exist as managed services in the cloud - it will take a few minutes for resources to be created and reach a `Ready` state, considering the time it takes for cloud managed resources to be provisioned:

```bash
$ kubectl get RedisInstanceRequirement
NAME                    READY   SYNCED   CONNECTION-SECRET
cartservice-redis-req   True    True     online-catalog-redis-secret

$ kubectl get RedisInstance
NAME                          READY   SYNCED   COMPOSITION
cartservice-redis-req-f8st8   True    True     redisinstances.gcp.infrastructure.murillodigital.com

$ kubectl get secret
NAME                          TYPE                                  DATA   AGE
default-token-zndtk           kubernetes.io/service-account-token   3      17d
online-catalog-redis-secret   connection.crossplane.io/v1alpha1     2      54s

$ kubectl get secret online-catalog-redis-secret -o yaml
apiVersion: v1
data:
  endpoint: MTAuMTUyLjEzMi4xMTY=
  port: NjM3OQ==
kind: Secret
metadata:
  annotations:
    from.propagate.crossplane.io/name: 3fb8ce8c-26f6-466e-9eec-9c7936c053f9
    from.propagate.crossplane.io/namespace: crossplane-system
  creationTimestamp: "2020-09-27T11:28:43Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:endpoint: {}
        f:port: {}
      f:metadata:
        f:annotations:
          .: {}
          f:from.propagate.crossplane.io/name: {}
          f:from.propagate.crossplane.io/namespace: {}
        f:ownerReferences:
          .: {}
          k:{"uid":"50965dd8-86a1-455c-8d18-9c0eb778e205"}:
            .: {}
            f:apiVersion: {}
            f:controller: {}
            f:kind: {}
            f:name: {}
            f:uid: {}
      f:type: {}
    manager: crossplane
    operation: Update
    time: "2020-09-27T11:28:43Z"
  name: online-catalog-redis-secret
  namespace: default
  ownerReferences:
  - apiVersion: infrastructure.murillodigital.com/v1alpha1
    controller: true
    kind: RedisInstanceRequirement
    name: cartservice-redis-req
    uid: 50965dd8-86a1-455c-8d18-9c0eb778e205
  resourceVersion: "4049402"
  selfLink: /api/v1/namespaces/default/secrets/online-catalog-redis-secret
  uid: 80a0816d-5343-45c8-b6ee-9ad94f78d2d1
type: connection.crossplane.io/v1alpha1


$ kubectl get all
NAME                                        READY   STATUS             RESTARTS   AGE
pod/cartservice-workload-85fc7b46d6-pn7nb   1/1     Running            6          6m21s
pod/cartservice-workload-85fc7b46d6-zmg5x   1/1     Running            5          5m51s

NAME                           TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/cartservice-workload   LoadBalancer   10.103.247.214   <pending>     7070:31337/TCP   6m21s
service/kubernetes             ClusterIP      10.96.0.1        <none>        443/TCP          17d

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/cartservice-workload   2/2     2            2           6m21s

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/cartservice-workload-85fc7b46d6   2         2         0       6m21s

```

And we will find the corresponding Redis instance provisioned in GCP:

![CrossPlane managed GCP Redis Memory Store Instance](/images/crossplane_oam/redis-gcp.png)

# Closing thoughts

I love the ability of managing cloud resources within the same context as the rest of my Kubernetes native solution, the fact you get full solution visibility and you can integrate the lifecycle of all components is really effective and powerful.

Having the ability to manage these resources across clouds makes the approach even richer.

The Open Application Model I think likely works best in enterprise contexts with large scale solutions and organized cloud initiatives, as powerful as the separation of concerns is, I believe for a lot of corporate solutions or relatively small sized projects it may be overkill.

In terms of adoption, as I mentioned it was not without effort, we are definitely looking at a model and suite of technologies that are in early stages and industry utilization must increase and documentation must mature for it to be appealing to a less adventurous audience.




Get notified of new experiments and articles on cloud native technologies, join my mailing list!.

<!-- Begin Mailchimp Signup Form -->
<link href="//cdn-images.mailchimp.com/embedcode/horizontal-slim-10_7.css" rel="stylesheet" type="text/css">
<style type="text/css">
	#mc_embed_signup{background:#fff; clear:left; font:14px Helvetica,Arial,sans-serif; width:100%;}
	/* Add your own Mailchimp form style overrides in your site stylesheet or in this style block.
	   We recommend moving this block and the preceding CSS link to the HEAD of your HTML file. */
</style>
<div id="mc_embed_signup">
<form action="https://murillodigital.us10.list-manage.com/subscribe/post?u=c12ff1afa71003663de3762cc&amp;id=4cff0f72fe" method="post" id="mc-embedded-subscribe-form" name="mc-embedded-subscribe-form" class="validate" target="_blank" novalidate>
    <div id="mc_embed_signup_scroll">
	<label for="mce-EMAIL">Subscribe</label>
	<input type="email" value="" name="EMAIL" class="email" id="mce-EMAIL" placeholder="email address" required>
    <!-- real people should not fill this in and expect good things - do not remove this or risk form bot signups-->
    <div style="position: absolute; left: -5000px;" aria-hidden="true"><input type="text" name="b_c12ff1afa71003663de3762cc_4cff0f72fe" tabindex="-1" value=""></div>
    <div class="clear"><input type="submit" value="Subscribe" name="subscribe" id="mc-embedded-subscribe" class="button"></div>
    </div>
</form>
</div>

<!--End mc_embed_signup-->
