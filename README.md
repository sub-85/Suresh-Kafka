# Kafka Composite

## Run Locally
- composite is intended to run locally
  - managed resources are created in the same cluster from which they are being managed
- to deploy resources to a remote cluster one has to update provider configs and replace `InjectedIdentity` with necessary credentials 

```bash
kubectl create ns crossplane-system
helm install crossplane --namespace crossplane-system crossplane-stable/crossplane  

# install providers
kubectl apply -f provider/

# additional permissions necessary for testing in local cluster
SA=$(kubectl -n crossplane-system get sa -o name | grep provider-helm | sed -e 's|serviceaccount\/|crossplane-system:|g')
kubectl create clusterrolebinding provider-helm-admin-binding --clusterrole cluster-admin --serviceaccount="${SA}"

SA=$(kubectl -n crossplane-system get sa -o name | grep provider-kube | sed -e 's|serviceaccount\/|crossplane-system:|g')
kubectl create clusterrolebinding provider-kubernetes-admin-binding --clusterrole cluster-admin --serviceaccount="${SA}"

# build,push and install configuration
kubectl crossplane build configuration -f composition --name kafka-configuration
kubectl crossplane push configuration -f composition/kafka-configuration.xpkg 
kubectl crossplane install configuration --package-pull-secrets=<docker-secret> "image.registry.com/.../kafka-composite:v1"

export CLAIM_NAME=xp-kafka

# create kafka stack
envsubst '$CLAIM_NAME' < example/claim.yaml | kubectl apply -f - 

# delete kafka stack
envsubst '$CLAIM_NAME' < example/claim.yaml | kubectl delete -f - 
```