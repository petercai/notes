# How to autoscale your SaaS application infrastructure | Red Hat Developer
This article discusses both how and why to scale your infrastructure automatically so that you aren't paying for resources you don't need. This is the last installment in the [SaaS architecture checklist series](https://developers.redhat.com/articles/2022/05/18/saas-architecture-checklist-kubernetes).

If you are a Software-as-a-Service (SaaS) provider, it is important to manage operational expenses while ensuring that your platform's capacity can always meet the needs of your users. Whether the traffic to your SaaS peaks on a predictable schedule (for example, office hours on weekdays or seasonal shopping) or whether you are planning for growth, [Kubernetes](https://developers.redhat.com/topics/kubernetes) has features to make sure you have the right level of capacity at any given time.

SaaS revenue typically comes through recurring fees that scale based on metrics such as the number of users, quantity of data stored or processed, access to advanced features, and other similar points of value. While each SaaS provider decides on their own pricing model and consumption metric, there is one common goal: The more your SaaS gets used, the more revenue you make.

But that usage also incurs increased infrastructure demand and expense. Maintaining a healthy operating income depends on keeping your infrastructure expenses below the corresponding revenue.

It pays to build on a compute platform that can scale in response to demand, with automation that scales the application in real-time. This article will discuss basic techniques for automatically scaling a Kubernetes cluster and an application based on load by combining three components:

*   Cluster API to enable cluster resizing
*   Cluster Autoscaler to resize the cluster based on load
*   Horizontal Pod Autoscaler to scale an application based on load

Kubernetes native infrastructure

Kubernetes native infrastructure
--------------------------------

Kubernetes native infrastructure is a pattern for using the Kubernetes API to manage some or all of the compute and network infrastructure used by a cluster. This concept is the foundation for some of the scalability components described in the following sections.

You can extend the Kubernetes API through [custom resources](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/), which allow a software developer to add a new API endpoint to a cluster, write a controller to implement the features of that endpoint, and thus create a Kubernetes-native API to manage almost anything. You might have heard of the [Operator Pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) that uses this technique of extending the Kubernetes API, typically for the purpose of managing applications, but custom resources can also be used to manage infrastructure.

Cluster API

Cluster API
-----------

The [Cluster API project](https://cluster-api.sigs.k8s.io/) is a popular example of Kubernetes-native infrastructure management. One of its core capabilities is to add and remove nodes in an existing cluster using a concept called Machines. The API has a Machine API resource, which represents the desire to have a node in a cluster. The API also has a MachineSet API resource (similar in concept to a ReplicaSet), which includes a desired size and a template for creating Machines.

The Machine and MachineSet APIs let you resize a cluster just by changing an integer field to the number of nodes you desire. Figure 1 shows the API's basic behavior.

[![](https://developers.redhat.com/sites/default/files/styles/article_floated/public/Openshift%20cluster%20image.png?itok=iqBIj8Bd)
](https://developers.redhat.com/sites/default/files/Openshift%20cluster%20image.png)

At a very high level, cluster scaling using the cluster API project works like this:

*   A person or automation decides to change the size of a MachineSet (more on this later).
*   A controller process either creates new Machine resources or deletes existing ones based on comparing the desired number of machines to the number that actually exists.
*   When a new Machine resource gets created, a platform-specific controller uses a cloud API to add a host to the cluster. For example, on AWS the controller would use the EC2 API to provision a new host.
*   When a Machine resource gets deleted, the platform-specific controller drains the corresponding node of workloads and then uses the platform-specific API to delete the host.

With this simple approach, it's easy to scale a cluster up and down. The API works the same across different cloud providers, so scaling changes can be made without any cloud-specific knowledge.

But how can the scaling decision to add and remove nodes be automated based on the real-time load? That's the responsibility of the Cluster Autoscaler.

Cluster Autoscaler

Cluster Autoscaler
------------------

The [Cluster Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler#cluster-autoscaler) uses the Machine API to add and remove nodes in a cluster based on its observations of workload scheduling pressure and node utilization. The Autoscaler adds nodes when pods cannot be scheduled due to resource constraints and removes underused nodes. Additional configuration is available, such as limits on how many nodes to create and settings that tune how aggressively the Autoscaler removes underused nodes.

Combining the Machine API with the Cluster Autoscaler, you can have a self-scaling compute platform that responds to the resource needs of your SaaS application.

To learn more about how [Red Hat OpenShift](https://developers.redhat.com/openshift) enables machine management and scaling, including example resources, see the [overview of machine management](https://docs.openshift.com/container-platform/4.11/machine_management/index.html) and the [autoscaler section](https://docs.openshift.com/container-platform/4.11/machine_management/applying-autoscaling.html) of the product documentation.

But there is one more piece to the puzzle: How can you scale your application, adding and removing pods as its load changes over time? That is the job of the Horizontal Pod Autoscaler.

Horizontal Pod Autoscaler

Horizontal Pod Autoscaler
-------------------------

[Horizontal Pod Autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) adds and removes pods in response to changing load. The service bases decisions on two metrics: CPU utilization and memory footprint. You can define thresholds at which pods will be added or removed to keep per-pod resource utilization near target values. In a common use case where a workload is defined as a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), this Autoscaler adjusts the **replicas** field based on load.

In the following example, a `frontend-app` Deployment resource varies from 10 to 100 pods. Crucially, the Deployment's pod template must include a CPU [resource request](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#requests-and-limits). The Autoscaler uses each pod's actual CPU utilization to calculate a percentage of its CPU request and then uses the average to determine whether to scale the application up or down.

```null

apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-app
  namespace: default
spec:
  maxReplicas: 100
  minReplicas: 10
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: frontend-app
  targetCPUUtilizationPercentage: 75
```

The newer [autoscaling/v2 API](https://github.com/kubernetes/design-proposals-archive/blob/main/autoscaling/hpa-v2.md) lets you scale based on memory utilization and define more sophisticated policies. New policy options can configure how rapidly to make changes, including different policies that apply to scaling up versus scaling down. This API also supports the use of custom metrics.

SaaS provisioning can be automated under Kubernetes

SaaS provisioning can be automated under Kubernetes
---------------------------------------------------

Combining the Cluster API, Cluster Autoscaler, and Horizontal Pod Autoscaler, you can automatically scale your SaaS application up and down based on changes in utilization. The underlying infrastructure will automatically scale with your resource specifications, saving costs when utilization is low.

Red Hat SaaS Foundations is a partner program designed for building enterprise-grade SaaS solutions on the [Red Hat OpenShift](https://developers.redhat.com/products/openshift/) or [Red Hat Enterprise Linux](https://developers.redhat.com/products/rhel/overview) platforms and deploying them across multiple cloud and non-cloud footprints. [Email](http://mailto:saas@redhat.com/) us to learn more about partnering with Red Hat to build your SaaS.

Be sure to read all the articles in the [SaaS architecture checklist series](https://developers.redhat.com/articles/2022/05/18/saas-architecture-checklist-kubernetes). Comment below if you have questions. We welcome your feedback.