# prerequisite for this project.
[kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) – A command line tool for working with Kubernetes clusters. For more information, see Installing or updating kubectl.

[eksctl](https://eksctl.io/installation/) – A command line tool for working with EKS clusters that automates many individual tasks. For more information, see Installing or updating.

[AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) – A command line tool for working with AWS services, including Amazon EKS. For more information, see Installing, updating, and uninstalling the AWS CLI in the AWS Command Line Interface User Guide. After installing the AWS CLI, we recommend that you also configure it. For more information, see Quick configuration with aws configure in the AWS Command Line Interface User Guide.




# Create a cluster and Fargate
```bash 
eksctl create cluster --name demo-cluster --region us-east-1 --fargate
```

# Create Fargate profile
``` 
eksctl create fargateprofile \
    --cluster demo-cluster \
    --region us-east-1 \
    --name alb-sample-app \
    --namespace game-2048
```
# Deploy the deployment, service and Ingress
``` 
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```

## commands to configure IAM OIDC provider
``` 
export cluster_name=demo-cluster
```
```
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5) 
```

```
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```

# How to setup alb add on

## download IAM Policy
```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json

```
## Create IAM Policy
```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```
## Create IAM role
``` 
eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```
# install helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh

sudo yum install helm
```

# Deploy ALB controller
## add helm repo
```
helm repo add eks https://aws.github.io/eks-charts
```
## update repo
```
helm repo update eks
```
## install
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \            
  -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>
```
## Verify that the deployments are running.
```
kubectl get deployment -n kube-system aws-load-balancer-controller
```



# Delete the cluster
```
eksctl delete cluster --name demo-cluster --region us-east-1
```
