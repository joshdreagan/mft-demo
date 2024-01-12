# MFT Demo

## Pre-requisites

Install the Data Grid operator. This can be done within your target namespace (ie, 'datagrid'), or globally across all namespaces.

Install the Camel K operator. This can be done within your target namespace(s) (ie, 'camel' & 'dc2-camel'), or globally across all namespaces.

After installing the Camel K operator, you can download the `kamel` CLI for your platform via the OpenShift web console. Click on the <img height="24" src="img/question-mark.png"/> -> "Command Line Tools" link in the upper-right corner. Unzip the `kamel` CLI executable into the "${PROJECT_ROOT}/bin" directory.

## Set up DC1

__Environment__

```
#
# Set your env variables.
export PROJECT_ROOT="$(pwd)"
export PATH="${PROJECT_ROOT}/bin:${PATH}"
export SFTP_USER="<SFTP_USER>"
export SFTP_PASS="<SFTP_PASS>"
```

__Data Grid__

```
#
# Create/configure a Data Grid server.
cd "${PROJECT_ROOT}/datagrid"
oc new-project datagrid
oc -n datagrid apply -f ./infinispan-config.yaml
oc -n datagrid apply -f ./infinispan.yaml
oc -n datagrid apply -f ./in-progress-repo-cache.yaml
```

__Camel__

```
#
#
cd "${PROJECT_ROOT}/camel"
oc new-project camel
# Be sure to update the secrets and configmaps with your valid credentials and coordinates.
oc -n camel create secret generic aws-credentials --from-file=aws-secret.properties
oc -n camel create secret generic sftp-credentials --from-file=sftp-secret.properties
oc -n camel create configmap file-poller-properties --from-file=application.properties
kamel run --namespace camel --config configmap:file-poller-properties --config secret:aws-credentials --config secret:sftp-credentials file-poller.camel.yaml

#
# Optional: Scale the integration.
oc -n camel scale it file-poller --replicas 2
```

## Testing the code

Generate some test files.

```
mkdir /tmp/mft-demo
for i in {1..100}; do echo "hello $i" > /tmp/mft-demo/file-$i.txt; done;
```

Upload the files to the SFTP server.

```
scp /tmp/mft-demo/* $SFTP_USER@$SFTP_HOST:./
```
