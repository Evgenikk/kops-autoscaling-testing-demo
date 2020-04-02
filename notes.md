

 ```
aws iam create-group --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess --group-name kops\naws iam
attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonRoute53FullAccess --group-name kops\naws iam
attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess --group-name kops\naws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/IAMFullAccess --group-name kops\naws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess --group-name kops\n
aws iam create-user --user-name kops
aws iam add-user-to-group --user-name kops --group-name kops
aws iam create-access-key --user-name kops

aws s3api create-bucket --bucket kops.us-east-2.test --region us-east-1
aws s3api put-bucket-versioning --bucket kops.us-east-2.test  --versioning-configuration Status=Enabled
export KOPS_STATE_STORE=s3://kops.us-east-2.test

export AWS_ACCESS_KEY_ID=$(aws configure get aws_access_key_id)

export AWS_SECRET_ACCESS_KEY=$(aws configure get aws_secret_access_key)


```


```
kops toolbox template --template cluster.tmpl.yaml --values us-2.values.yaml --output us-east-2.config.yaml
kops replace -f us-east-2.config.yaml --force
kops update cluster us-east-2.test.k8s.local
kops rolling-update cluster us-east-2.test.k8s.local  --yes
```


Установка и запусл polaris (только в качестве теста, в реальности лучше использовать helm)
```
kubectl apply -f https://github.com/fairwindsops/polaris/releases/latest/download/dashboard.yaml
kubectl port-forward --namespace polaris svc/polaris-dashboard 8080:80
```