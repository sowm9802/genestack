# Deploy Blazar


### Create secrets

``` shell
kubectl --namespace openstack \
        create secret generic blazar-rabbitmq-password \
        --type Opaque \
        --from-literal=username="blazar" \
        --from-literal=password="$(< /dev/urandom tr -dc _A-Za-z0-9 | head -c${1:-32};echo;)"

kubectl --namespace openstack \
        create secret generic blazar-secrets \
        --type Opaque \
        --from-literal=service-username="blazar" \
        --from-literal=service-password="$(< /dev/urandom tr -dc _A-Za-z0-9 | head -c${1:-32};echo;)" \
        --from-literal=service-domain="service" \
        --from-literal=service-project="service" \
        --from-literal=service-project-domain="service" \
        --from-literal=db-endpoint="mariadb-cluster-primary.openstack.svc.cluster.local" \
        --from-literal=db-name="blazar" \
        --from-literal=db-username="blazar" \
        --from-literal=db-password="$(< /dev/urandom tr -dc _A-Za-z0-9 | head -c${1:-32};echo;)" \
        --from-literal=secret-key="$(< /dev/urandom tr -dc _A-Za-z0-9 | head -c${1:-32};echo;)" \
        --from-literal=keystone-endpoint="$(kubectl --namespace openstack get secret keystone-keystone-admin -o jsonpath='{.data.OS_AUTH_URL}' | base64 -d)" \
        --from-literal=keystone-username="blazar" \
        --from-literal=default-region="RegionOne"
```

## Run the deployment

``` shell
kubectl --namespace openstack apply -k /etc/genestack/kustomize/blazar/base
```
