Step 1: Verify the Pods and Service Are Running

Before accessing the frontend pod, ensure all components are up and running.

    Check Pods:
    bash

kubectl get pods

    Look for both frontend and backend pods with a Running status. Example output:
    text

    NAME       READY   STATUS    RESTARTS   AGE
    frontend   1/1     Running   0          10m
    backend    1/1     Running   0          10m

Check Service:
bash
kubectl get svc

    Ensure backend-service exists with a ClusterIP assigned. Example output:
    text

NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
backend-service   ClusterIP   10.96.123.456   <none>        5678/TCP   10m
The frontend pod uses the service name backend-service to resolve this IP.

Step 2: Access the Frontend Pod

Since the frontend pod isn’t exposing a port for external access (it’s just running a curl loop), you’ll need to inspect it from within the cluster. Here are two main ways to do this:
Option 1: Check Frontend Logs

The frontend pod’s container runs a curl command that sends requests to backend-service:5678 every 5 seconds. You can check its logs to see the output of those requests.

    View Logs:
    bash

kubectl logs pod/frontend

    If the frontend is successfully communicating with the backend, you should see repeated output like:
    text

    Hello World from Backend Pod
    Hello World from Backend Pod
    ...
    This output comes from the curl command hitting the http-echo server in the backend pod via the service.



## Next Slide
What’s Happening?
Command:
bash
kubectl port-forward pod/backend 8080:5678

    kubectl port-forward: This is a Kubernetes command that allows you to forward traffic from a local port on your machine to a port on a pod in the cluster.
    pod/backend: Specifies the target pod, which is the backend pod defined in your earlier YAML (the one running the hashicorp/http-echo container).
    8080:5678: Maps port 8080 on your local machine (e.g., your laptop or VirtualBox) to port 5678 on the backend pod. Port 5678 is where the http-echo server is listening, as defined in the pod’s spec (containerPort: 5678).

Output:
text
Forwarding from 127.0.0.1:8080 -> 5678
Forwarding from [::1]:8080 -> 5678
Handling connection for 8080
Handling connection for 8080

    127.0.0.1:8080: Indicates that the forwarding is set up on your local machine’s IPv4 loopback address (localhost) at port 8080.
    [::1]:8080: Indicates the same for the IPv6 loopback address.
    -> 5678: Shows that traffic is being forwarded to port 5678 on the backend pod.
    Handling connection for 8080: Appears when a connection is made to port 8080 on your local machine, meaning kubectl is actively forwarding that traffic to the pod.

Second Terminal Command:
bash
curl http://localhost:8080

    This sends an HTTP request to localhost:8080 on your machine.
    Because of the port-forward, this request is tunneled to port 5678 on the backend pod.

Output:
text
Hello World from Backend Pod

    This is the response from the http-echo server running in the backend pod, as configured with the argument -text=Hello World from Backend Pod in the pod’s YAML.

Why Are We Doing This?

    Accessing the Pod: Pods in a Kubernetes cluster typically run in an isolated network and aren’t directly accessible from outside the cluster (e.g., your laptop). kubectl port-forward creates a temporary bridge between your local machine and the pod, allowing you to interact with it as if it were running locally.
    Testing/Debugging: This is a common technique to test or debug an application running inside a pod without needing to expose it externally (e.g., via a Kubernetes Service with type NodePort or LoadBalancer).
    Simplicity: It’s a quick way to verify that the backend pod’s application is working as expected.

What Are These Doing?

    kubectl port-forward pod/backend 8080:5678:
        Establishes a tunnel from your local machine’s port 8080 to the backend pod’s port 5678.
        Listens for incoming connections on 127.0.0.1:8080 (and [::1]:8080 for IPv6) and forwards them to the pod.
    curl http://localhost:8080:
        Sends an HTTP GET request to localhost:8080 on your machine.
        The port-forward command intercepts this request and sends it to the http-echo server in the backend pod.
        The server responds with the configured text, which curl displays in your terminal.

Use of This Setup

    Verification: Confirms that the backend pod is running and its application (the http-echo server) is responding correctly on port 5678.
    Development: Allows developers to interact with pod applications during development or troubleshooting without modifying the cluster’s networking setup.
    Learning: Demonstrates how Kubernetes pods can be accessed externally for testing purposes.

How It Ties to Your Earlier Setup

    In your original YAML files, the frontend pod uses the backend-service to communicate with the backend pod within the cluster. However, kubectl port-forward bypasses the service and directly connects your local machine to the backend pod.
    This is an alternative way to test the backend pod’s functionality without relying on the frontend pod or the service.

Additional Notes

    Temporary: The port-forward connection lasts only as long as the kubectl command is running. If you stop it (e.g., with Ctrl+C), the forwarding stops.
    Local Scope: Only your local machine can access localhost:8080. Other machines can’t use this unless you adjust the binding (e.g., using 0.0.0.0 with additional flags, though this is less common).
    Multiple Requests: The repeated Handling connection for 8080 lines suggest you ran curl multiple times, and each request was successfully forwarded.