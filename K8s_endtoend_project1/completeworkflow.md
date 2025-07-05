
---

### ✅ Flow OF EKS Cluster :

1. ### **Create an EKS Cluster with Fargate**

   * **Command**:

     ```bash
     eksctl create cluster --name demo-cluster --region us-east-1 --fargate
     ```
   * **Why?**
     To provision a fully managed Kubernetes cluster on AWS, using **AWS Fargate** for serverless pod hosting (no EC2/nodegroup management).
   * **What it does?**
     Creates EKS control plane + default Fargate profile (`fp-default`) that allows `kube-system` and `default` namespace pods to run on Fargate.

---

2. ### **Create a Fargate Profile for Custom Namespace**

   * **Command**:

     ```bash
     eksctl create fargateprofile \
       --cluster demo-cluster \
       --region us-east-1 \
       --name alb-sample-app \
       --namespace game-2048
     ```
   * **Why?**
     To run application pods (like 2048 game) in `game-2048` namespace using **Fargate**.
   * **What it does?**
     Ensures only workloads in `game-2048` run serverlessly.

---

3. ### **Deploy the 2048 App Resources**

   * **Command**:

     ```bash
     kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
     ```
   * **Why?**
     To create the full app setup: Namespace → Deployment → Service → Ingress.
   * **What it does?**
     Deploys the 2048 game pods, a NodePort service, and an Ingress resource.

---

4. ### **Associate IAM OIDC Provider with Cluster**

   * **Command**:

     ```bash
     eksctl utils associate-iam-oidc-provider \
       --region us-east-1 \
       --cluster demo-cluster \
       --approve
     ```
   * **Why?**
     Required to allow Kubernetes service accounts to assume AWS IAM roles.
   * **What it does?**
     Enables secure IAM role association for the ALB Ingress Controller pod.

---

5. ### **Create IAM Policy for ALB Controller**

   * **Command**:

     ```bash
     curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json

     aws iam create-policy \
       --policy-name AWSLoadBalancerControllerIAMPolicy \
       --policy-document file://iam_policy.json
     ```
   * **Why?**
     Allows the ALB Controller pod to call AWS APIs like `CreateLoadBalancer`.
   * **What it does?**
     Creates a custom IAM policy for ELB management.

---

6. ### **Create IAM Service Account for the ALB Controller**

   * **Command**:

     ```bash
     eksctl create iamserviceaccount \
       --cluster=demo-cluster \
       --namespace=kube-system \
       --name=aws-load-balancer-controller \
       --region=us-east-1 \
       --attach-policy-arn=arn:aws:iam::<ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
       --approve
     ```
   * **Why?**
     Needed to run the ALB controller pod with IAM permissions.
   * **What it does?**
     Creates a Kubernetes service account linked to IAM role with permissions.

---

7. ### **Install ALB Ingress Controller via Helm**

   * **Command**:

     ```bash
     helm repo add eks https://aws.github.io/eks-charts
     helm repo update

     helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
       --namespace kube-system \
       --set clusterName=demo-cluster \
       --set serviceAccount.create=false \
       --set serviceAccount.name=aws-load-balancer-controller \
       --set region=us-east-1 \
       --set vpcId=<Your-VPC-ID>
     ```
   * **Why?**
     To deploy the controller that listens to Ingress objects and manages AWS ALBs.
   * **What it does?**
     ALB Ingress Controller gets installed, and starts watching your ingress rules.

---

8. ### **ALB Gets Automatically Created**

   * **How?**
     The ALB Ingress Controller:

     * Watches for `Ingress` in `game-2048`
     * Finds the related `Service` → then `Pods`
     * Talks to AWS ELB API → creates an **Application Load Balancer**
     * Routes traffic from ALB DNS → your 2048 pods

---

9. ### ✅ **Access the App**

   * **Command**:

     ```bash
     kubectl get ingress -n game-2048
     ```
   * **Why?**
     To get the external ALB DNS name.
   * **What it does?**
     Shows `ADDRESS` field — the public URL where your 2048 app is available.

---

### 💥 Final Flow Summary:

```text
EKS Cluster (Fargate Mode)
     ↓
Fargate Profile for game-2048
     ↓
K8s Namespace + Deployment + Service + Ingress
     ↓
OIDC + IAM Role + IAM Policy
     ↓
Helm deploys ALB Ingress Controller using ServiceAccount with IAM Role
     ↓
ALB Controller watches Ingress and creates AWS ALB
     ↓
You access app via ALB DNS
```
