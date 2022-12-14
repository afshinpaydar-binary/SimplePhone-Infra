# SimplePhone-Infra
Terraform for building EKS fargate infrastructure

## Set Environment variables and requirements

| Tool      | Version |
| --------- | ----------:|
| awscli    | ~> 2.7.1   |
| terraform | ~> v1.0.0  |
| helm      | ~> 3.0.0   |
| flux      | ~> 0.31.5  |

Export credentials as environment values:
```
export KUBE_CONFIG_PATH=~/.kube/config
export AWS_ACCESS_KEY_ID="xxxxxxx"
export AWS_SECRET_ACCESS_KEY="yyyyyyy"
export TF_VAR_DB_PASSWORD=<Secure Password for DB>
export TF_VAR_API_SECRET_KEY=<App Secret to encrypt cookies and save them to the browser>
```

#### Edit `aws_region` and `aws_profile` in the `variables.tf` file

## Setup S3 Bucket for terraform storage backend
```sh
git clone git@github.com:afshinpaydar-binary/SimplePhone-Infra.git
cd SimplePhone-Infra/terraform/s3_terraform_state/
terraform init
terraform plan -out .tfplan
terraform apply
```

## Setup Infra
```sh
cd SimplePhone-Infra/terraform/
terraform init -reconfigure
terraform plan -out .tfplan
terraform apply ".tfplan"
```

## Configure HTTPS and Certificate
Create a managed Certificate by AWS Certificate Manager and update value of `alb.ingress.kubernetes.io/certificate-arn` annotation in the `./k8s/ingress.yaml` file and also update `host` there same as your site URL.

## Check the status and get access to app
```sh
kubectl get ing -n production
# Create a CNAME that ponits <Site DNS > to <LB DNS - output of previous command>
helm list -n kube-system
kubectl logs -f -n kube-system \
  -l app.kubernetes.io/name=aws-loadbalancer-controller
curl $(kubectl get ing -n production -o jsonpath='{.items[0].status.loadBalancer.ingress[0].hostname}')
```

## Clean up
```sh
cd
kubectl delete namespace production
terraform refresh
terraform destroy
aws s3 rm s3://terraform-simplephone --recursive
cd s3_terraform_state
terraform destroy
```
