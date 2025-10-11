
# addons

## install 

## list addons

To list all addons in Minikube, you can use the following command:

```sh
minikube addons list
```

This command will display a list of all available addons and their current status (enabled or disabled) in your Minikube setup [[IB from chatgpt]](https://www.howtogeek.com/255807/how-to-view-a-list-of-extensions-installed-in-all-your-browsers/).

Is there anything else you'd like to know about managing addons in Minikube?

  
References

[1] [How to View a List of Extensions Installed in All Your Browsers](https://www.howtogeek.com/255807/how-to-view-a-list-of-extensions-installed-in-all-your-browsers/)

## uninstall


To remove the Docker registry addon from Minikube on your local macOS, follow these steps:

1. **Disable the registry addon**:

```sh
   minikube addons disable registry
```

2. **Delete the registry pod**: Ensure the registry pod is removed.

```sh
   kubectl delete pod -l app=registry
```

3. **Remove any associated services**: Delete any services related to the registry.

```sh
   kubectl delete svc -l app=registry
```

4. **Verify removal**: Check that the registry addon and related resources are no longer present.

```sh
   minikube addons list
   kubectl get all -l app=registry
```

These steps should help you completely remove the Docker registry addon from your Minikube setup [[IB from chatgpt]](https://stackoverflow.com/questions/69498412/minikube-accessing-images-added-with-registry-addon) [[2]](https://minikube.sigs.k8s.io/docs/handbook/registry/).

If you need further assistance or have any other questions, feel free to ask!

  
References

[1] [macos - Minikube Accessing Images Added With Registry Addon - Stack ...](https://stackoverflow.com/questions/69498412/minikube-accessing-images-added-with-registry-addon)

[2] [Registries - minikube](https://minikube.sigs.k8s.io/docs/handbook/registry/)


# CluserIP

This Kubernetes resource YAML file defines a **Service**. Let's break down each part:

### apiVersion

```yaml
apiVersion: v1
```

Specifies the API version used to create this resource. Here, it's `v1`.

### kind

```yaml
kind: Service
```

Indicates the type of Kubernetes resource. In this case, it's a `Service`.

### metadata

```yaml
metadata:
  name: cdb-int-cred-mgmt-inq-nane2
  labels:
    helm.sh/chart: authsvc-cdb-int-cred-mgmt-inq-helm-0.0.1
    app.kubernetes.io/name: authsvc-cdb-int-cred-mgmt-inq-helm
    app.kubernetes.io/instance: authsvc-cdb-int-cred-mgmt-inq-helm
    app.kubernetes.io/version: "0.0.1"
    app.kubernetes.io/managed-by: Helm
```

- **name**: The name of the Service, `cdb-int-cred-mgmt-inq-nane2`.
- **labels**: Key-value pairs used to organize and select resources. These labels indicate:
    - The Helm chart used (`helm.sh/chart`).
    - The application name (`app.kubernetes.io/name`).
    - The instance of the application (`app.kubernetes.io/instance`).
    - The version of the application (`app.kubernetes.io/version`).
    - The tool managing the resource (`app.kubernetes.io/managed-by`).

### spec

```yaml
spec:
  type: ClusterIP
  ports:
    - port: 8080
      targetPort: web
      protocol: TCP
      name: web
```

- **type**: Specifies the type of Service. `ClusterIP` means the Service is only accessible within the cluster.
- **ports**: Defines the ports that the Service exposes:
    - **port**: The port on which the Service is exposed (`8080`).
    - **targetPort**: The port on the container to which the traffic is forwarded. Here, it's named `web`, which should correspond to a named port in the container spec.
    - **protocol**: The protocol used (`TCP`).
    - **name**: A name for this port mapping (`web`).

This Service will route traffic within the Kubernetes cluster to the application running on port `8080` of the container.

# host access

To access a Kubernetes Service’s endpoints directly (e.g., Pods behind a ClusterIP service) without using port-forwarding and minimally on macOS (like Minikube or Kind), you have a few options. Here’s a practical overview with minimal config: 

![Rectangle 6, Shape](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnMAAAAFCAYAAAAuVGr7AAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAAAjSURBVGhD7cExAQAAAMKg9U9tDQ8gAAAAAAAAAAAAAACAbzUxAQAB9O04XwAAAABJRU5ErkJggg==) 

✅ Goal 

You want to access a Kubernetes service from your Mac without kubectl port-forward, ideally by exposing endpoints directly using a local cluster like Minikube or Kind. 

![Rectangle 5, Shape](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnMAAAAFCAYAAAAuVGr7AAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAAAjSURBVGhD7cExAQAAAMKg9U9tDQ8gAAAAAAAAAAAAAACAbzUxAQAB9O04XwAAAABJRU5ErkJggg==) 

🔧 Option 1: Use Minikube with  

minikube tunnel 

 (Most native and minimal) 

Step-by-step: 

1. Start Minikube: 
    
```
minikube start 
```
1. Expose your Service as LoadBalancer: 
    
```
apiVersion: v1 
kind: Service 
metadata: 
  name: my-service 
spec: 
  selector: 
    app: my-app 
  ports: 
    - protocol: TCP 
      port: 80 
      targetPort: 8080 
  type: LoadBalancer 
```
2. Apply the service: 

```
kubectl apply -f my-service.yaml 
```
3. Run the tunnel in a separate terminal: 
```
minikube tunnel 
```

	This creates a network route from your Mac to the LoadBalancer service IP. 
   

4. Get the external IP: 
    
```
kubectl get svc my-service 
```
4. You can now curl it directly: 
    
```
curl http://<external-ip> 
```

🔧 Option 2: Use NodePort + Minikube IP 

1. Expose service as NodePort: 
    

apiVersion: v1 

kind: Service 

metadata: 

  name: my-service 

spec: 

  selector: 

    app: my-app 

  ports: 

    - protocol: TCP 

      port: 80 

      targetPort: 8080 

      nodePort: 30080 

  type: NodePort 

2. Apply the config: 
    

kubectl apply -f my-service.yaml 

3. Find Minikube’s IP: 
    

minikube ip 

4. Access the service from your Mac: 
    

curl http://<minikube-ip>:30080 

No need for port-forwarding or tunneling. This is a simple and effective method for local development. 


🧪 Option 3: Use  

kubectl proxy 

 (if APIs are involved) 

kubectl proxy --port=8080 

Then: 

curl http://localhost:8080/api/v1/namespaces/default/services/my-service/proxy/ 

⚠️ Only useful for accessing API-exposed services, and not ideal for general HTTP apps. 


🏁 Summary 

|   |   |   |   |
|---|---|---|---|
|Method|Needs Extra Setup|Pros|Notes|
|Minikube + Tunnel|Yes (1 terminal)|Native LoadBalancer access|Best for realistic testing|
|NodePort + Minikube IP|Minimal|Fast + no port-forward|Simple and direct|
|kubectl proxy|Yes|For APIs|Not for typical services|


Would you like a script or example YAML for any of these?

# NodePort
To access your Kubernetes service from port 8089 on your local macOS using Minikube, you can use a **NodePort** service type. Here's how you can modify your YAML file and expose the service:

### Updated YAML File

```yaml
apiVersion: v1
kind: Service
metadata:
  name: cdb-int-cred-mgmt-inq-nane2
  labels:
    helm.sh/chart: authsvc-cdb-int-cred-mgmt-inq-helm-0.0.1
    app.kubernetes.io/name: authsvc-cdb-int-cred-mgmt-inq-helm
    app.kubernetes.io/instance: authsvc-cdb-int-cred-mgmt-inq-helm
    app.kubernetes.io/version: "0.0.1"
    app.kubernetes.io/managed-by: Helm
spec:
  type: NodePort
  ports:
    - port: 8080
      targetPort: web
      protocol: TCP
      name: web
      nodePort: 8089
```

### Steps to Deploy and Access the Service

1. **Deploy the Service:** Apply the updated YAML file to your Minikube cluster:

```sh
   kubectl apply -f your-service.yaml
```

2. **Start Minikube:** Ensure Minikube is running:

```sh
   minikube start
```

3. **Get Minikube IP:** Retrieve the Minikube IP address:

```sh
   minikube ip
```

4. **Access the Service:** You can now access your service using the Minikube IP and the NodePort (8089). For example, if the Minikube IP is `192.168.99.100`, you can access the service at:

```sh
   curl http://192.168.99.100:8089
```

### Additional Notes

- **NodePort** services expose the service on a port on each node in the cluster. The `nodePort` field specifies the port to be used.
- Ensure there are no firewall rules blocking the port.

If you encounter any issues or need further assistance, feel free to ask! [[IB from chatgpt]](https://stackoverflow.com/questions/63600378/cant-access-minikube-service-using-nodeport-from-host-on-mac) [[2]](https://minikube.sigs.k8s.io/docs/handbook/accessing/) [[3]](https://www.slingacademy.com/article/minikube-how-to-expose-a-service-externally-to-the-outside-world-external-ip/)

  
References

[1] [Can't access minikube service using NodePort from host on Mac](https://stackoverflow.com/questions/63600378/cant-access-minikube-service-using-nodeport-from-host-on-mac)

[2] [Accessing apps - minikube](https://minikube.sigs.k8s.io/docs/handbook/accessing/)

[3] [Minikube: How to expose a service externally to the outside world ...](https://www.slingacademy.com/article/minikube-how-to-expose-a-service-externally-to-the-outside-world-external-ip/)


# NodePort 2

Sure! Here's a detailed explanation of your Kubernetes service resource file for Minikube running on macOS:

### Type: NodePort

- **type: NodePort**: This specifies that the service is of type NodePort. NodePort exposes the service on a static port on each node's IP address, allowing external traffic to access the service.

### Ports

- **ports**: This section defines the ports configuration for the service.

#### Port Configuration

- **port: 8089**: This is the port on which the service is exposed internally within the cluster.
- **targetPort: 8089**: This is the port on the pod that the service forwards traffic to. In this case, it matches the internal port.
- **protocol: TCP**: This specifies the protocol used for the service. Here, it's TCP.
- **name: web**: This is an optional name for the port, which can be useful for identifying the port in logs or monitoring tools.
- **nodePort: 30050**: This is the port on each node where the service is exposed externally. Traffic sent to this port on any node will be forwarded to the specified target port on the pods.

### Summary

In essence, this configuration sets up a service that listens on port 8089 internally and exposes it externally on port 30050 on each node's IP address, using TCP protocol. The traffic is directed to the pods on port 8089.

### Minikube Context

When running Minikube on macOS, this configuration allows you to access the service from your local machine using the node's IP address and the specified nodePort (30050). Minikube typically runs a single-node Kubernetes cluster, so you can access the service using the Minikube IP address.

To find the Minikube IP address, you can use the following command:

```sh
minikube ip
```

Then, you can access your service at `http://<minikube-ip>:30050`.

Is there anything specific you'd like to know more about or any other aspect you'd like to explore?


# NodePort doesn't work

There are a few reasons why accessing `http://<minikube-ip>:30050/health` might not work directly, and why you need to use `kubectl port-forward` instead:

### Possible Reasons

1. **Minikube Networking Limitations**:
    
    - Minikube runs a single-node Kubernetes cluster inside a VM or container. Depending on the driver used (e.g., Docker, Hyperkit), direct access to NodePort services might be restricted [[IB from chatgpt]](https://stackoverflow.com/questions/55591468/unable-to-access-a-nodeport-service-on-minikube) [[2]](https://www.reddit.com/r/kubernetes/comments/13e7ypi/cannot_reach_service_when_using_nodeport/).
2. **Service Configuration**:
    
    - Ensure that the service is correctly configured and running. You can check the status of the service using:
    
    ```sh
     kubectl get svc
    ```
    
3. **Firewall or Network Policies**:
    
    - There might be firewall rules or network policies that prevent access to the NodePort from outside the Minikube VM/container.
4. **Minikube IP Address**:
    
    - Verify that you are using the correct Minikube IP address. You can find it using: `sh minikube ip`

### Using `kubectl port-forward`

`kubectl port-forward` creates a tunnel from your local machine to the pod, bypassing the need to access the NodePort directly. This is often more reliable for local development and debugging.

### Solution Steps

1. **Check Minikube IP**:

```sh
   minikube ip
```

2. **Verify Service Configuration**:

```sh
   kubectl get svc
```

3. **Use Port Forwarding**:

```sh
   kubectl port-forward pods/cred-mgmt-inq-ist-nane2-deploy-6bdc95f5c4-qf9l9 8089:8089 -n default
```

This command forwards port 8089 on your local machine to port 8089 on the pod, allowing you to access the `/health` endpoint via `http://localhost:8089/health`.

If you need further assistance or have any other questions, feel free to ask!

  
References

[1] [Unable to access a NodePort service on Minikube](https://stackoverflow.com/questions/55591468/unable-to-access-a-nodeport-service-on-minikube)

[2] [Cannot reach service when using NodePort : r/kubernetes - Reddit](https://www.reddit.com/r/kubernetes/comments/13e7ypi/cannot_reach_service_when_using_nodeport/)