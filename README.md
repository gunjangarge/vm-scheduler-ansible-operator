# Create secret with AWS API credentials
```
kubectl create secret generic aws-secret --from-literal=region=us-east-1 --from-literal=aws-secret-access-key=<secret access key> --from-literal=aws-access-key-id=<secret access key id>
```

# Deploy operator directly on cluster

#### Note: All commands to be executed inside `vm-scheduler-ansible-operator` directory.

## Build operator image and deploy
```
make docker-build docker-push IMG="quay.io/gunjangarge/vm-scheduler-ansible-operator:v0.0.1"
make deploy IMG="quay.io/gunjangarge/vm-scheduler-ansible-operator:v0.0.1"
```

## Apply CRs
```
kubectl apply -f config/samples/aws_v1_awsvmschedulerstop.yaml
kubectl apply -f config/samples/aws_v1_awsvmschedulerstart.yaml
```

## Check CronJobs are present
```
kubectl get cronjobs
```

## Delete CRs
```
kubectl delete -f config/samples/aws_v1_awsvmschedulerstop.yaml
kubectl delete -f config/samples/aws_v1_awsvmschedulerstart.yaml
```

## Delete operator
```
make undeploy
```

# Deploy your Operator with OLM

#### Note: All commands to be executed inside `vm-scheduler-ansible-operator` directory.

## Install OLM (if required)
```
operator-sdk olm install
```

## Build operator image and deploy
```
make docker-build docker-push IMG="quay.io/gunjangarge/vm-scheduler-ansible-operator:v0.0.1"
```

## Bundle your operator, then build and push the bundle image
```
make bundle bundle-build bundle-push BUNDLE_IMG="quay.io/gunjangarge/vm-scheduler-ansible-operator-bundle:v0.0.1" IMG="quay.io/gunjangarge/vm-scheduler-ansible-operator:v0.0.1"
```

## Run your bundle on cluster
```
operator-sdk run bundle quay.io/gunjangarge/vm-scheduler-ansible-operator-bundle:v0.0.1
```

## Check crd and csv
```
kubectl get csv
kubectl get crd
```

## Apply CRs
```
kubectl apply -f config/samples/aws_v1_awsvmschedulerstop.yaml
kubectl apply -f config/samples/aws_v1_awsvmschedulerstart.yaml
```

## Check CronJobs are present
```
kubectl get cronjobs
```

## Delete CRs
```
kubectl delete -f config/samples/aws_v1_awsvmschedulerstop.yaml
kubectl delete -f config/samples/aws_v1_awsvmschedulerstart.yaml
```

## Cleanup installed operator
```
operator-sdk cleanup vm-scheduler-ansible-operator
```
