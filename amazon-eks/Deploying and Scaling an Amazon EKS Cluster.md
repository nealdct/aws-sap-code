# Deploying and Scaling an Amazon EKS Cluster

In this lab, you'll learn how to deploy an Amazon EKS cluster, deploy a sample application, and scale the cluster using the Kubernetes Horizontal Pod Autoscaler (HPA).

**Step 1: Installing the required tools**

1. Install and configure the AWS CLI

2. Install [kubectl]

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

3. Install the [eksctl]) command-line tool.


```bash
# for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

# (Optional) Verify checksum
curl -sL "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

sudo mv /tmp/eksctl /usr/local/bin
```

Save the above script as `eksctl.sh` then run `chmod +x eksctl.sh` and `sudo sh eksctl.sh`

**Step 2: Creating an EKS Cluster**

1. Create a new Amazon EKS cluster using the `eksctl` command:

```bash
eksctl create cluster --name my-eks-cluster --region us-east-1 --nodegroup-name my-nodegroup --node-type t2.small --nodes 3 --nodes-min 1 --nodes-max 5 --managed
```

Replace `my-eks-cluster` and `us-east-1` with your preferred cluster name and AWS region.

**Step 3: Deploying a Sample Application**

1. Create a new file named `deployment.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

2. Deploy the sample application to your EKS cluster:

```bash
kubectl apply -f deployment.yaml
```

**Step 4: Exposing the Application**

1. Expose the application using a Kubernetes service:

```bash
kubectl expose deployment nginx-deployment --type=LoadBalancer --name=my-service
```

2. Get the external IP address of the LoadBalancer:

```bash
kubectl get services my-service
```

You can access the sample application using the external IP address and port number.

**Step 5: Setting up Horizontal Pod Autoscaler (HPA)**

1. Create a new file named `hpa.yaml` with the following content:

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-deployment-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

2. Apply the HPA configuration:

```bash
kubectl apply -f hpa.yaml
```

Now, your EKS cluster will automatically scale the number of pods based on the CPU utilization.

3. Configure permissions for your IAM user account

```bash
kubectl get configmap aws-auth -n kube-system -o yaml > aws-auth.yaml
```
Edit the `aws-auth.yaml` file and add the `mapUsers` section (as per the below example)

```yaml
apiVersion: v1
data:
  mapRoles: |
    - groups:
      - system:bootstrappers
      - system:nodes
      rolearn: arn:aws:iam::<account-number>:role/eksctl-my-eks-cluster-nodegroup-m-NodeInstanceRole-ZKA8ZSWLC7Z2
      username: system:node:{{EC2PrivateDNSName}}
  mapUsers: |
    - userarn: <your-iam-user-arn>
      username: <your-iam-user-name>
      groups:
        - system:masters

kind: ConfigMap
metadata:
  creationTimestamp: "2023-05-12T10:41:25Z"
  name: aws-auth
  namespace: kube-system
  resourceVersion: "1215"
  uid: 54d885a3-3484-41d4-b032-41959915d8b6
```

Then, run:

```bash
kubectl apply -f aws-auth.yaml
```

You should now have access to view all objects in the cluster

## Delete the service and cluster

kubectl delete svc my-service

eksctl delete cluster --name my-eks-cluster