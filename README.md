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

# Create a catalog source
#### Ensure that ```opm``` utility is installed on the system.

## 1. Initialize a catalog for a file-based catalog:

### a. Create a directory for the catalog:
```
mkdir vm-scheduler-ansible-operator-index
```
### b. Create a Dockerfile that can build a catalog image
```
cat > vm-scheduler-ansible-operator-index.Dockerfile
# The base image is expected to contain
# /bin/opm (with a serve subcommand) and /bin/grpc_health_probe
FROM quay.io/openshift/origin-operator-registry:4.9.0

# Configure the entrypoint and command
ENTRYPOINT ["/bin/opm"]
CMD ["serve", "/configs"]

# Copy declarative config root into image at /configs
ADD <operator_name>-index /configs

# Set DC-specific label for the location of the DC root directory
# in the image
LABEL operators.operatorframework.io.index.configs.v1=/configs

CTRL+D
```

### c. Populate the catalog with your package definition

```
opm init vm-scheduler-ansible-operator --default-channel=preview --description=./README.md --output yaml > vm-scheduler-ansible-operator-index/index.yaml
```

## 2. Add a bundle to the catalog

```
opm render quay.io/gunjangarge/vm-scheduler-ansible-operator-bundle:v0.0.1 --output=yaml >> vm-scheduler-ansible-operator-index/index.yaml
```

## 3. Add a channel entry for the bundle. For example, modify the following example to your specifications, and add it to your vm-scheduler-ansible-operator-index/index.yaml file

```
---
schema: olm.channel
package: vm-scheduler-ansible-operator
name: preview
entries:
  - name: vm-scheduler-ansible-operator.v0.0.1 
```

## 4. Validate the file-based catalog
```
opm validate vm-scheduler-ansible-operator-index
echo $?
```

## 5. Build the catalog image
```
docker build . -f vm-scheduler-ansible-operator-index.Dockerfile -t quay.io/gunjangarge/vm-scheduler-ansible-operator-index:v0.0.1
```

## 6. Push the catalog image to a registry
```
docker login quay.io
docker push quay.io/gunjangarge/vm-scheduler-ansible-operator-index:v0.0.1
```

## Specify above index image while creating catalog.
