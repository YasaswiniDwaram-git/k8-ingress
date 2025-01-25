https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/deploy/installation/

1.Create an IAM OIDC provider. You can skip this step if you already have one for your cluster.

since ingress controller is external and we need our aws account to provide access , we create IAM OIDC

eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster expense \
    --approve



2.Download an IAM policy for the LBC using one of the following commands

curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json

3.Create an IAM policy named AWSLoadBalancerControllerIAMPolicy. If you downloaded a different policy, replace iam-policy with the name of the policy that you downloaded

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
	
4.Create an IAM role and Kubernetes ServiceAccount for the LBC. Use the ARN from the previous step.

eksctl create iamserviceaccount \
--cluster=expense \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::084375546724:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region us-east-1 \
--approve


and then install drivers(to properly connect eks and k8-apploadbalancer) through helm charts below commands..

helm repo add eks https://aws.github.io/eks-charts

helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=expense --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller