# Deployment of ArgoCD, Argo Events, and Argo workflows to AWS to implement GitOps CI/CD

# Deploy to the AWS EKS using Terraform! 


- EKS Cluster: AWS managed Kubernetes cluster of master servers
- EKS Node Group
- Associated VPC, Internet Gateway, Security Groups, and Subnets: Operator managed networking resources for the EKS Cluster and worker node instances
- Associated IAM Roles and Policies: Operator managed access resources for EKS and worker node instances

```
terraform init
terraform get
terraform apply
```


Verify the aws-load-balancer-controller is installed
```
kubectl get deployment -n kube-system aws-load-balancer-controller
```
Our infrastructure in aws will look like this:
![AWS infra](https://docs.aws.amazon.com/prescriptive-guidance/latest/patterns/images/pattern-img/abf727c1-ff8b-43a7-923f-bce825d1b459/images/281936fa-bc43-4b4e-a343-ba1eab97df38.png)



# Installing Argo-cd 
[Getting started with ArgoCD](https://argo-cd.readthedocs.io/en/stable/getting_started/)

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.4.12/manifests/install.yaml

kubectl port-forward svc/argocd-server -n argocd 31719:443

xdg-open https://localhost:8080
```

The API server can then be accessed using https://localhost:31719

```
kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
echo U1lub2lpWk55U3NuYjI0aA== | base64 --decode
```
Take the decoded password and login to the ui

# clone the repo which has our ArgoCD manifests for the project and app
```
git clone git@github.com:davidfox87/argocd-production.git
```
Install projects and apps
```
kubectl apply -f projects.yaml
kubectl apply -f apps.yaml

```

## generate tls-secret using sealed secrets
```
kubectl create secret generic tls-secret -n staging \
--from-file=key=certs/key.pem \
--from-file=cert=certs/cert.pem

kubectl get secret tls-secret -n staging -o yaml | kubeseal --controller-namespace kube-system \
                                                 --controller-name sealed-secrets \
                                                 --format yaml tls-secret.yaml \ 
                                                > base/tls-secret.yaml
kubectl delete secret tls-secret -n staging

```

## wait for everything to sync and be healthy in the argo-cd UI
woop

## apply ingress object
Ingress object will spawn aws-load-balancer-controler, which has a backend that points to istio-ingressgateway
Important!!! The ingress object has to go into the istio-system namespace.
# Our true GitOps CI/CD platform
![GitOps ArgoCD](https://www.eksworkshop.com/images/argocd/argocd_architecture.png)

## installing argocd, argo workflow, argo events
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


kubectl get secret argocd-initial-admin-secret -n argocd
echo anFJcTNnY3pmeVZPTWN5LQ== | base64 --decode

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```


# Clean up your workspace

Delete the application in argo-cd

Then destroy all infrastructure in AWS using
```
terraform destroy
```





    add an Ingress to create an ALB with the ALB Ingress controller
    update a Service of the Istio Ingress Gateway, and instead of the LoadBalancer type will set the NodePort, so AWS ALB Ingress Controller can create a TargetGroup to be used with the ALB
    deploy a test application with a common Kubernetes Service
    for the testing application need to create a Gateway and VirtualService that will configure Envoy of the Istio Ingress Gateway to route traffic to the Service of the application
