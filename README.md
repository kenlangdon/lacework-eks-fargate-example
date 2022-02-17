# Lacework on EKS Fargate Example

## Environment Set-up

1. Install [eksctl](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)
2. Clone this repo
3. Create a Lacework Data Collector agent token (https://docs.lacework.com/create-agent-access-tokens-and-download-agent-installers)


## Steps

Create an AWS EKS Fargate cluster (this will take ~15 minutes)

`eksctl create cluster --name fargate-fun --region us-east-2 --fargate`

The `--fargate` option above automatically creates a fargateprofile `fp-default` in the default namespace so the below namespace and profile creation are not needed for this.

Add the Config Map to the cluster (use the lw-config-map.example as your guide)

`kubectl apply -f lw-config-map.yml`

Add the RBAC rules which includes a ServiceAccount, ClusterRole and ClusterRoleBinding with the environment variables needed by the Data Collector

`kubectl apply -f lw-rbac.yml`

Add container pods at will. This repo contains one example:

`kubectl apply -f lw-sidecar-pod-example.yml`

It will take some time for data to be processed and revealed in the Lacework Polygraph Data Platform. Happy Fargating!


### Cleanup

Delete all the objects and then the EKS Cluster

`eksctl delete cluster fargate-fun`

or if you need to force it:

`eksctl delete cluster fargate-fun --force`


### Optional Step to see this cluster and its workloads in the AWS console

If you would like to be able to see the cluster and workloads in the AWS Console for another role use the following guide (credit Eric Heiser and Rob Danz)

Docs: https://aws.amazon.com/premiumsupport/knowledge-center/eks-kubernetes-object-access-error/

Run `curl -o eks-console-full-access.yaml https://amazon-eks.s3.us-west-2.amazonaws.com/docs/eks-console-full-access.yaml` 
or use the eks-console-full-access-yaml already present in the repo

Run `kubectl apply -f eks-console-full-access.yaml`

Run `kubectl edit -n kube-system configmap/aws-auth`

Add under mapRoles adjusting for your role/user/group:

    - rolearn: arn:aws:iam::950194951070:role/lacework-customerdemo-admin-role
      username: lacework-customerdemo-admin-role
      groups:
      - system:masters


### IF you ever have an existing EKS Cluster and want to add fargate to it

Create a namespace to use for all things Fargate

`kubectl create ns your-fargate-namespace`

Create a Fargate Profile in your cluster (https://docs.aws.amazon.com/eks/latest/userguide/fargate-profile.html)

`eksctl create fargateprofile --cluster fargate-fun --name fargate-fp --namespace your-fargate-nammespace`

And be sure to update all your yaml files that create K8s objects to use the namespace you setup
