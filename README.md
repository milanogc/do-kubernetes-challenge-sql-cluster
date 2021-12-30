# Digital Ocean Kubernetes Challenge

## Create Kubernetes cluster on Digital Ocean

1. [Create a Kubernetes cluster on Digital Ocean](https://cloud.digitalocean.com/kubernetes/clusters/new), default values are ok, keep the node count as 3 at least.
2. After the cluster creation, download its config file (`.yaml`) from Digital Ocean. I placed it on `$HOME/Downloads/kubeconfig.yaml`.

## Install and configure `kubectl` command

1. Install the [Kubernetes official client (kubectl)](https://kubernetes.io/docs/tasks/tools/#kubectl) on your system.
2. Then, a way to use the previously downloaded Kubernetes config file with `kubectl` is through the `KUBECONFIG` environment variable:
```bash
export KUBECONFIG=$HOME/Downloads/kubeconfig.yaml
```

3. Listing the nodes from the remote cluster:

```bash
kubectl get nodes
```

## Install Kubegres operator

Up to now we have an empty Kubernetes cluster, so we need to a way to deploy our scalable Postgres.

Kubegres is a Kubernetes Operator that eases the deployment of scalable Postgres databases and is completely free, so I chose it for the Digital Ocean's challenge.

Everything I needed for adoption of Kubegres operator is documented at [Kubegres' Getting Started document](https://www.kubegres.io/doc/getting-started.html).

1. Install Kubegres Operator

```bash
kubectl apply -f https://raw.githubusercontent.com/reactive-tech/kubegres/v1.14/kubegres.yaml
```

## Configure a Postgres cluster

1. Create a secret resource for the Postgres instances, file `my-postgres-secret.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mypostgres-secret
  namespace: default
type: Opaque
stringData:
  superUserPassword: postgresSuperUserPsw # change it
  replicationUserPassword: postgresReplicaPsw # change it
```

2. Apply the secret resource to the cluster:

```bash
kubectl apply -f my-postgres-secret.yaml
```

3. Create a Kubegres resource that configures the Postgres cluster, file `my-postgres.yaml`:

```yaml
apiVersion: kubegres.reactive-tech.io/v1
kind: Kubegres
metadata:
  name: mypostgres
  namespace: default
spec:
  replicas: 3
  image: postgres:14.1
  database:
    size: 1Gi
  env:
    - name: POSTGRES_PASSWORD
      valueFrom:
        secretKeyRef:
          name: mypostgres-secret
          key: superUserPassword
    - name: POSTGRES_REPLICATION_PASSWORD
      valueFrom:
        secretKeyRef:
          name: mypostgres-secret
          key: replicationUserPassword
```

Only thing that is different from the [Kubegres' Getting Started document](https://www.kubegres.io/doc/getting-started.html) is the field `spec.database.size` that changed from `200Mi` to `1Gi`, this is due to Digital Ocean's minimal volume size being `1Gi`.

Without this change the following error happens: `TODO`.

1. Apply the Kubegres resource to the cluster:

```bash
kubectl apply -f my-postgres.yaml
```

Now we should already have our scalable Postgres cluster working.
