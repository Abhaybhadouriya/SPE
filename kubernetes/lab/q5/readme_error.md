# Error 1
error: Metrics API not available
when running  **kubectl top nodes** or **kubectl top pods**
To fix this, follow these steps:
1. Check if Metrics Server is running

kubectl get deployment metrics-server -n kube-system

If it says NotFound or shows 0/1 pods available, you need to install or fix it.
2. Install Metrics Server (if not installed)

You can install it using:

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

⚠️ If you're using minikube, you can also enable it via:

minikube addons enable metrics-server

3. Fix common issues

Sometimes the metrics server fails due to insecure TLS settings. You can patch the deployment to allow insecure TLS:

kubectl edit deployment metrics-server -n kube-system
## Now wait for 2 min to up
After a minute or two, run:

kubectl top nodes

or

kubectl top pods

It should now display CPU and memory usage stats.

# Error 2
Error from server (NotFound): deployments.apps "metrics-server" not found

This confirms that the Metrics Server is not installed on your cluster.
✅ To install the Metrics Server:

Run the following command:

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

    🔒 This installs the official Metrics Server components into your cluster.

✅ Then patch it (if needed):

Some environments (like Minikube, kind, or local clusters) need insecure TLS. Run this command to patch the Metrics Server:

kubectl patch deployment metrics-server -n kube-system \
  --type='json' \
  -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'

🔍 After a minute, verify:

kubectl get pods -n kube-system | grep metrics-server

You should see the pod running.

Then try:

kubectl top nodes
kubectl top pods

# Error 3 
reload environment

Why do we reload .bashrc or .bash_profile?

When you make changes to your shell environment configuration files like .bashrc, .bash_profile, or .zshrc, those changes do not apply automatically to your current terminal session.

These files are only read when:

    A new shell session starts (like opening a new terminal)

    Or you manually reload them
