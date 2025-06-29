Project 1 - Deploying Scalable Applications on AWS EKS with Kubernetes and Ingress Routing

1. Prerequisites

kubectl
eksctl
AWS CLI
aws-iam-authenticator - separate installation needed using chocolatey because it was buggy during AWS CLI Installation

2. Creating eks cluster with a default fargate profile 

```eksctl create cluster --name vjproject1 --region eu-west-2 --fargate```

3. Creating a custom fargate profile with an adittional namespace for deployment

 ``` eksctl create fargateprofile  --cluster demo-cluster --region eu-west-2 --name alb-sample-app --namespace game-2048 ```


4. Deployment, Service and Ingress

```kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml```

See 2048_full.yaml for full explaination of what the yaml provisions and why

Wait till all pods and services are ready and running

5. Configure IAM OIDC provider 
```eksctl utils associate-iam-oidc-provider --cluster vjproject1 --approve```

6. Setting up IAM Policy and Role

Download IAM policy 

```curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json```

Create IAM Role

```eksctl create iamserviceaccount --cluster=vjproject1 --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy --approve```

7. Deploy ALB Controller

Add helm repo for helm charts

```helm repo add eks https://aws.github.io/eks-charts```

Installing 

```helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=vjproject1 --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller --set region=eu-west-2 --set vpcId=<your-vpc-id>```

8. Verification
To check if controller is up and running

```kubectl get deployment -n kube-system aws-load-balancer-controller```

To check if pods are up and running

```kubectl get pods -n kube-system```

To check if ingress resources are up and connected appropriately 

```kubectl get ingress -n game-2048```

Make sure its talking to alb and hosts are present







