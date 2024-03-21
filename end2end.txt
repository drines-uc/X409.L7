## Step 1: Setup the Basic K8s Cluster

From this repo move inot the demo0 directory
```
cd demo0
```

Add your aws credentials to your terminal...

Terraform Apply
```
terraform apply
```
... respond 'yes'
... this will take a few minutes

## Step 2: Get your container into ECR

Get the URI for your ECR Repository

Use the aws cli to get ECR creds into docker

```
aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 387254918478.dkr.ecr.us-east-2.amazonaws.com/devops
```

use `docker images` to get the image id of the docker image you care about

tag the image you want to upload

```
docker tag <image_id> <ECR URI>:<TAG>
```
where TAG is how you want the new image referenced in ECR

push the image to ECR
```
docker push <ECR URI>:<TAG>

```

## Step 3: Add Argo to your cluster

Install the argo cli
```
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

Point kubectl to your cluster
```
aws eks --region $(terraform output -raw region) update-kubeconfig  --name $(terraform output -raw cluster_name)
```
Create a name space for argo
```
kubectl create namespace argocd
```

Apply the ArgoCd manifest
```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Make Argo accessible extrenally
```
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

Get the address of your argo instance

```
kubectl get services --namespace=argocd
```

Get the default password for argo

```
argocd admin initial-password -n argocd

```

Login to Argo and change your password
```
argocd login <argo network address from above>
argocd account update-password
```

Set kubectl context to the argocd namespace

```
kubectl config set-context --current --namespace=argocd
```

Setup secure ssh access to your repo
```
argocd repo add git@github.com:drines-uc/hello_http.git --ssh-private-key-path ~/.ssh/hellodeploy
```


Add your app
```
argocd app create hellohttp --repo git@github.com:drines-uc/hello_http.git --path kube_conf --dest-server https://kubernetes.default.svc --dest-namespace default
```

Get the URL of your app
```
kubectl get services --namespace=default
```
