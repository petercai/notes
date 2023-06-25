# Basics of Kubernetes — beginner
James has a microservices application. To ease the application development and deployment process, he containerized the application. As the size of the application grew, the number of containers also grew. Managing the containers became a problem. To solve this problem, he did some research.

He found tools that automate the process of developing and managing multi-container applications. These tools are called **Orchestration tools**.

**Kubernetes** is an example of an orchestration tool. This article explains the basic components of Kubernetes.

![](_assets/1!UgVtOlgQEE1XEnY-7lLdTA.png)

**Kubernetes** is an orchestration tool that helps to manage and scale multi-container applications. Kubernetes groups your containers into pods and then deploys the pods to different servers.

![](_assets/1!2Q4FsymDIDtWS_aqiDbwSA.png)

**A pod** is a collection of one or more containers. Most times, a pod contains only one container. A pod usually contains more than one container when the containers are tightly coupled ( _are dependent on each other_ ).

Each pod has a static **IP address**. They use the IP address to communicate with each other. A **Kubernetes service** provides a static IP address.

**Pods are ephemeral.** This means that they die/crash. When a pod dies, this causes downtime of your application. So instead of relying on one instance of your pod, you create a **replica/clone** of the pod.

To create a replica of your pod, you would specify a blueprint for the pods. And inside the blueprint, you specify how many replicas of the pod you want. This blueprint is called a **deployment.**

So if a pod dies or is overloaded, Kubernetes replicates it and redeploy the replica/clone. This helps to prevent application downtime.

Pods run on a server or a virtual machine. In a Kubernetes architecture, these servers or virtual machines are called **nodes**.

![](_assets/1!ER-4hz_zJr232_EhvIAciA.png.jpg)

image from [sysspace](https://www.sysspace.net/post/kubernetes-diving-in)

A node is a physical server or a virtual machine used to run applications (pods).

> **A Kubernetes cluster is a collection of linked nodes**.

A basic Kubernetes cluster has at least one **master node**, and **several worker nodes**.

Worker nodes
------------

The pods are deployed and run on the **worker nodes**. Each worker node can handle many pods.

Every worker node has a **kubelet. The kubelet** allows the nodes in the cluster to communicate with each other.

Worker nodes are much bigger than master nodes. This is because they have more workloads. But master nodes are more important than worker nodes.

Master node
-----------

The master node controls and manages the worker nodes. A master node has the following components

*   **API server:** it serves as an entry point into the cluster. All external communication with the cluster is done through the API server.
*   **Controller manager:** controllers are in charge of monitoring the performance of the cluster. They make sure that nodes are always up and running.
*   **Scheduler:** the scheduler monitors the pods and nodes in a cluster. It assigns pods to different nodes based on the capacity of the nodes.
*   **etcd:** is a storage unit. It holds the configuration details of the cluster.

The worker nodes and master nodes communicate with each other through the virtual network.

Kubernetes is a container orchestration platform. It helps to manage, scale and deploy containerized applications. It groups your containers into pods, then deploys them on different servers called nodes.

Thanks for reading.