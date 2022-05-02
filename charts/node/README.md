# Substrate/Polkadot node helm chart

## Installing the chart

```console
helm repo add parity https://paritytech.github.io/helm-charts/
helm install polkadot-node parity/node
```

This will deploy a single Polkadot node with the default configuration.

### Deploying a node with data synced from a snapshot archive (eg. [Polkashot](https://polkashots.io/))

Polkadot:
```console
helm install polkadot-node parity/node --set node.chainDataSnapshotUrl=https://dot-rocksdb.polkashots.io/snapshot --set node.chainDataSnapshotFormat=7z
```

Kusama:
```console
helm install kusama-node parity/node --set node.chainDataSnapshotUrl=https://ksm-rocksdb.polkashots.io/snapshot --set node.chainDataSnapshotFormat=7z --set node.chainPath=ksmcc3
```
⚠️ For some chains where the local directory name is different from the chain ID, `node.chainPath` needs to be set to a custom value.

### Optional Vault Integration

To integrate this chart with vault:
- Vault agent injector [installed](https://www.vaultproject.io/docs/platform/k8s/injector/installation) on the cluster
- Kubernetes [auth enabled](https://learn.hashicorp.com/tutorials/vault/kubernetes-sidecar#configure-kubernetes-authentication) in your vault instance
- Secrets for either the keys or the nodeKey created in your vault using kv-2
- A policy for each of your secrets configured in your vault
- an authentication role crated in your vault instance (should match the serviceAccount name for this chart) with policies for accessing your keys attached

```
node:
  vault:
    keys:
      - name: aura
        type: aura
        scheme: sr25519
        vaultPath: kv/secret/polkadot-node # path at which the secret is located in Vault
        vaultKey: aura # key under which the secret value is stored in Vault
        extraDerivation: "//${HOSTNAME}//aura" # allows to have unique derived keys for each pod of the statefulset
    nodeKey:
      name: nodekey
      vaultPath: kv/secret/polkadot-node
      vaultKey: nodekey
```

## Parameters

### Common parameters

| Parameter           | Description                                  | Default                        |
|---------------------|----------------------------------------------|--------------------------------|
| `nameOverride`      | String to partially override node.fullname   | `nil`                          |
| `fullnameOverride`  | String to fully override node.fullname       | `nil`                          |
| `imagePullSecrets`  | Labels to add to all deployed objects        | `[]`                           |
| `podAnnotations`    | Annotations to add to pods                   | `{}` (evaluated as a template) |
| `nodeSelector`      | Node labels for pod assignment               | `{}` (evaluated as a template) |
| `tolerations`       | Tolerations for pod assignment               | `[]` (evaluated as a template) |
| `affinity`          | Affinity for pod assignment                  | `{}` (evaluated as a template) |
| `storageClass`      | The storage class to use for volumes         | `default`                      |

### Node parameters

| Parameter                                                                 | Description                                                                                                                                                                                                                                          | Default                                                             |
|---------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------|
| `node.chain`                                                              | Network to connect the node to (ie `--chain`)                                                                                                                                                                                                        | `polkadot`                                                          |
| `node.command`                                                            | Command to be invoked to launch the node binary                                                                                                                                                                                                      | `polkadot`                                                          |
| `node.flags`                                                              | Node flags other than `--name` (set from the helm release name), `--base-path` and `--chain` (both set with `node.chain`)                                                                                                                            | `--prometheus-external --rpc-external --ws-external --rpc-cors all` |
| `node.keys`                                                               | The list of keys to inject on the node before startup (object{ type, scheme, seed }), supercedes `node.vault.keys`                                                                                                                                   | `{}`                                                                |
| `node.vault.authPath`                                                     | Set the vault-agent authentication path (defaults to `auth/kubernetes`)                                                                                                                                                                              | `nil`                                                               |
| `node.vault.authRole`                                                     | Set the vault role to be used by the vault-agent (defaults to the service account name)                                                                                                                                                              | `nil`                                                               |
| `node.vault.authType`                                                     | Set the vault-agent authentication type (defaults to `kubernetes` when not set)                                                                                                                                                                      | `nil`                                                               |
| `node.vault.authConfigType`                                               | Set the vault-agent authentication additional `type` parameter                                                                                                                                                                                       | `nil`                                                               |
| `node.vault.authConfigServiceAccount`                                     | Set the vault-agent authentication additional `service-account` parameter                                                                                                                                                                            | `nil`                                                               |
| `node.vault.keys`                                                         | The list of vault secrets to inject on the node before startup (object{name, vaultPath, vaultKey, scheme, type, extraDerivation}). Note that the vaultKey must be alphanumeric.                                                                      | `{}`                                                                |
| `node.vault.nodeKey`                                                      | The vault secret to inject as a custom nodeKey (the secrets value must be 64 byte hex) (object {name, vaultPath, vaultKey})                                                                                                                          | `{}`                                                                |
| `node.persistGeneratedNodeKey`                                            | Persist the auto-generated node key inside the data volume (at /data/node-key)                                                                                                                                                                       | `false`                                                             |
| `node.customNodeKey`                                                      | Use a custom node-key, if `node.persistGeneratedNodeKey` is true then this will not be used.  (Must be 64 byte hex key), supercedes `node.vault.nodeKey`                                                                                             | `nil`                                                               |
| `node.enableStartupProbe`                                                 | If true, enable the startup probe check                                                                                                                                                                                                              | `true`                                                              |
| `node.enableReadinessProbe`                                               | If true, enable the readiness probe check                                                                                                                                                                                                            | `true`                                                              |
| `node.dataVolumeSize`                                                     | The size of the chain data PersistentVolume                                                                                                                                                                                                          | `100Gi`                                                             |
| `node.replica`                                                            | Number of replica in the node StatefulSet                                                                                                                                                                                                            | `1`                                                                 |
| `node.role`                                                               | Set the role of the node: `full`, `authority`/`validator`, `collator` or `light`                                                                                                                                                                     | `full`                                                              |
| `node.chainDataSnapshotUrl`                                               | Download and load chain data from a snapshot archive http URL                                                                                                                                                                                        | ``                                                                  |
| `node.chainDataSnapshotFormat`                                            | The snapshot archive format (`tar` or `7z`)                                                                                                                                                                                                          | `tar`                                                               |
| `node.chainDataGcsBucketUrl`                                              | Sync chain data files from a GCS bucket (eg. `gs://bucket-name/folder-name`)                                                                                                                                                                         | ``                                                                  |
| `node.chainPath`                                                          | Path at which the chain database files are located (`/data/chains/${CHAIN_PATH}`)                                                                                                                                                                    | `nil` (if undefined, fallbacks to the value in `node.chain`)        |
| `node.chainDataKubernetesVolumeSnapshot`                                  | Initialize the chain data volume from a Kubernetes VolumeSnapshot                                                                                                                                                                                    | ``                                                                  |
| `node.customChainspecUrl`                                                 | Download and use a custom chainspec file from a URL                                                                                                                                                                                                  | `nil`                                                               |
| `node.forceDownloadChainspec`                                             | Force the chainspec download even if it is already present on the volume                                                                                                                                                                             | `false`                                                             |
| `node.collator.isParachain`                                               | If true, configure the node as a parachain (set the relay-chain flags after `--`)                                                                                                                                                                    | `nil`                                                               |
| `node.collator.relayChainCustomChainspecUrl`                              | Download and use a custom relay-chain chainspec file from a URL                                                                                                                                                                                      | `nil`                                                               |
| `node.collator.relayChainDataSnapshotUrl`                                 | Download and load relay-chain data from a snapshot archive http URL                                                                                                                                                                                  | `nil`                                                               |
| `node.collator.relayChainDataSnapshotFormat`                              | The relay-chain snapshot archive format (`tar` or `7z`)                                                                                                                                                                                              | `nil`                                                               |
| `node.collator.relayChainPath`                                            | Path at which the chain database files are located (`/data/polkadot/chains/${RELAY_CHAIN_PATH}`)                                                                                                                                                     | `nil`                                                               |
| `node.collator.relayChainDataKubernetesVolumeSnapshot`                    | Initialize the relay-chain data volume from a Kubernetes VolumeSnapshot                                                                                                                                                                              | `nil`                                                               |
| `node.collator.relayChainDataGcsBucketUrl`                                | Sync relay-chain data files from a GCS bucket (eg. `gs://bucket-name/folder-name`)                                                                                                                                                                   | `nil`                                                               |
| `node.collator.relayChainFlags`                                           | Relay-chain node flags other than `--name` (set from the helm release name), `--base-path` and `--chain`                                                                                                                                             | `nil`                                                               |
| `node.resources.limits`                                                   | The resources limits (cpu/memory) for nodes                                                                                                                                                                                                          | `{}`                                                                |
| `node.podManagementPolicy`                                                | The pod management policy to apply to the StatefulSet, set it to `Parallel` to launch or terminate all Pods in parallel, and not to wait for pods to become Running and Ready or completely terminated prior to launching or terminating another pod | `{}`                                                                |
| `node.pruning`                                                            | The amount of blocks to retain. Set to a number or set to 0 for `--pruning=archive` .                                                                                                                                                                | `nil`                                                               |
| `node.resources.requests`                                                 | The resources requests (cpu/memory) for nodes                                                                                                                                                                                                        | `{}`                                                                |
| `node.serviceMonitor.enabled`                                             | If true, creates a Prometheus Operator ServiceMonitor                                                                                                                                                                                                | `false`                                                             |
| `node.serviceMonitor.namespace`                                           | Prometheus namespace                                                                                                                                                                                                                                 | `nil`                                                               |
| `node.serviceMonitor.internal`                                            | Prometheus scrape interval                                                                                                                                                                                                                           | `nil`                                                               |
| `node.serviceMonitor.scrapeTimeout`                                       | Prometheus scrape timeout                                                                                                                                                                                                                            | `nil`                                                               |
| `node.serviceMonitor.targetLabels`                                        | transfers labels on the Kubernetes Pod onto service monitor                                                                                                                                                                                          | `[nodes]`                                                           |
| `node.tracing.enabled`                                                    | If true, creates a jaeger agent sidecar                                                                                                                                                                                                              | `false`                                                             |
| `node.subtrateApiSiecar.enabled`                                          | If true, creates a substrate api sidecar                                                                                                                                                                                                             | `false`                                                             |
| `node.perNodeServices.setPublicAddressToExternalIp.enabled`               | If true sets the `--public-addr` flag to be the NodePort p2p services external address                                                                                                                                                               | `false`                                                             |
| `node.perNodeServices.setPublicAddressToExternalIP.ipRetrievalServiceUrl` | The external service to return the NodePort IP                                                                                                                                                                                                       | `https://ifconfig.io`                                               |

### Other parameters

| Parameter                          | Description                                                                                            | Default             |
|------------------------------------|--------------------------------------------------------------------------------------------------------|---------------------|
| `image.repository`                 | Node image name                                                                                        | `parity/polkadot`   |
| `image.tag`                        | Node image tag                                                                                         | `latest`            |
| `image.pullPolicy`                 | Node image pull policy                                                                                 | `Always`            |
| `initContainer.image.repository`   | Download-chain-snapshot init container image name                                                      | `crazymax/7zip`     |
| `initContainer.image.tag`          | Download-chain-snapshot init container image tag                                                       | `latest`            |
| `googleCloudSdk.image.repository`  | Sync-chain-gcs init container image name                                                               | `google/cloud-sdk`  |
| `googleCloudSdk.image.tag`         | Sync-chain-gcs init container image tag                                                                | `slim`              |
| `googleCloudSdk.serviceAccountKey` | Service account key (JSON) to inject into the Sync-chain-gcs init container using a Kubernetes secret  | `nil`               |
| `ingress.enabled`                  | If true, creates an ingress                                                                            | `false`             |
| `ingress.annotations`              | Annotations to add to the ingress (key/value pairs)                                                    | `{}`                |
| `ingress.rules`                    | Set rules on the ingress                                                                               | `[]`                |
| `ingress.tls`                      | Set TLS configuration on the ingress                                                                   | `[]`                |
| `podSecurityContext`               | Set the pod security context for the substrate node container                                          | `{ runAsUser: 1000, runAsGroup: 1000, fsGroup: 1000 }`|
| `jaegerAgent.image.repository`     | Jaeger agent image repository                                                                          | `jaegertracing/jaeger-agent` |
| `jaegerAgent.image.tag`            | Jaeger agent image tag                                                                                 | `1.28.0`            |
| `jaegerAgent.ports.compactPort`    | Port to use for jaeger.thrift over compact thrift protocol                                             | `6831`              |
| `jaegerAgent.ports.binaryPort`     | Port to use for jaeger.thrift over binary thrift protocol                                              | `6832`              |
| `jaegerAgent.ports.samplingPort`   | Port for HTTP sampling strategies                                                                      | `5778`              |
| `jaegerAgent.collector.url`        | The URL which jaeger agent sends data                                                                  | `nil`               |
| `jaegerAgent.collector.port   `    | The port which jaeger agent sends data                                                                 | `14250`             |    
| `extraContainers   `               | Sidecar containers to add to the node                                                                  | `[]`                |   
| `serviceAccount`                   | ServiceAccount used in init containers                                                                 | `{create: true, createRoleBinding: true,  annotations: {}}` |
