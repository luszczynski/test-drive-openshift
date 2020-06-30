# Introdução

## O que é isso?

Esse material é usado para capacitação e transferência de conhecimento de clientes e parceiros da Red Hat Brasil em **OpenShift**, aplicável tanto ao [**Red Hat OpenShift Container Platform**](https://www.openshift.com/container-platform/index.html) \(enterprise\) quanto ao [**Openshift OKD**](https://www.okd.io/) \(community\).

## Por que dessa forma?

Inspirados na cultura open-source, acreditamos que, ao disponibilizar o material de forma aberta, podemos **evoluir o workshop de forma colaborativa e alinhado com as necessidades dos nossos clientes, parceiros e a comunidade**.

Este trabalho está licenciado sob a [**Licença Atribuição-NãoComercial-CompartilhaIgual 4.0 Internacional Creative Commons \(CC BY-NC-SA 4.0\)**](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.pt_BR).

## Como usar?

**Não existe uma maneira certa ou errada de usar o conteúdo deste material.** Você poderia segur como um roteiro independente, usando a sua máquina particular como ambiente de experimentações, ou você pode se reunir com colegas e compartilhar a experiência.

**Se você tiver interesse de participar de um workshop com o time Red Hat Brasil, entre em contato com o seu time de contas/parceiro!**


## Observação

**Para o S2I com o Quarkus** funcionar importar o template template-openjdk11-rhel8-s2i.yaml para o namespace **openshift**. Após isso importar também a secret para a service account default em cada projeto para fazer pulling da imagem do **registry.redhat.io**.

## Etherpad

To install etherpad on Openshift go to https://github.com/luszczynski/openshift-etherpad for instructions

## Terminal

If you created your Openshift environment using RHPDS, you problably have a project called `terminal`. Use it.

Or you can also create a terminal inside Openshift by running:

```bash
oc new-project terminal-workshop
oc process -f https://raw.githubusercontent.com/openshift-homeroom/workshop-spawner/develop/templates/terminal-server-production.json --param SPAWNER_NAMESPACE=`oc project --short` --param CLUSTER_SUBDOMAIN=apps.cluster-brasilia-da5c.brasilia-da5c.example.opentlc.com | oc apply -f -
```

## Install logging

### Install ElasticSearch Operator

Create namescape `openshift-operators-redhat`

```bash
oc create -f - <<EOF
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-operators-redhat
  annotations:
    openshift.io/node-selector: ""
  labels:
    openshift.io/cluster-monitoring: "true"
EOF
```

Create Operator Group

```bash
oc create -f - <<EOF
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openshift-operators-redhat
  namespace: openshift-operators-redhat
spec: {}
EOF
```

Create Subscription

```bash
oc create -f - <<EOF
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: "elasticsearch-operator"
  namespace: "openshift-operators-redhat"
spec:
  channel: "4.2"
  installPlanApproval: "Automatic"
  source: "redhat-operators"
  sourceNamespace: "openshift-marketplace"
  name: "elasticsearch-operator"
EOF
```

Create Role-based access control

```bash
oc create -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: prometheus-k8s
  namespace: openshift-operators-redhat
rules:
- apiGroups:
  - ""
  resources:
  - services
  - endpoints
  - pods
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: prometheus-k8s
  namespace: openshift-operators-redhat
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: prometheus-k8s
subjects:
- kind: ServiceAccount
  name: prometheus-k8s
  namespace: openshift-operators-redhat
EOF
```

### Install Cluster Logging CR

Create Namespace for openshift-logging

```bash
oc create -f - <<EOF
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-logging
  annotations:
    openshift.io/node-selector: ""
  labels:
    openshift.io/cluster-logging: "true"
    openshift.io/cluster-monitoring: "true"
EOF
```

Create Operator Group

```bash
oc create -f - <<EOF
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: cluster-logging
  namespace: openshift-logging
spec:
  targetNamespaces:
  - openshift-logging
EOF
```

Create Operator Group

```bash
oc create -f - <<EOF
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: cluster-logging
  namespace: openshift-logging
spec:
  channel: "4.2"
  name: cluster-logging
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF
```

Create instance of Cluster Logging

```bash
oc create -f - <<EOF
apiVersion: "logging.openshift.io/v1"
kind: "ClusterLogging"
metadata:
  name: "instance"
  namespace: "openshift-logging"
spec:
  managementState: "Managed"  
  logStore:
    type: "elasticsearch"  
    elasticsearch:
      nodeCount: 3
      storage: {}
      redundancyPolicy: "SingleRedundancy"
  visualization:
    type: "kibana"  
    kibana:
      replicas: 1
  curation:
    type: "curator"  
    curator:
      schedule: "30 3 * * *"
  collection:
    logs:
      type: "fluentd"  
      fluentd: {}
EOF
```

## Workshopper

This is the documentation every customer/student will see during the labs. It must be deployed as a container inside Openshift.

### Local

If you want to develop and improve the docs, you can run it locally using one of the two methods below:

#### Using podman

If you want to check the documentation locally, run:

```bash
# Clone this project
git clone https://github.com/luszczynski/test-drive-openshift.git && cd test-drive-openshift.git

# Run the workshopper container
podman run -it --rm -p 8080:8080 -v $(pwd)/parte-2-openshift-4x:/app-data \
              -e CONTENT_URL_PREFIX="file:///app-data" \
              -e LOG_TO_STDOUT=true \
              -e WORKSHOPS_URLS="file:///app-data/_workshop1.yml" \
              quay.io/osevg/workshopper
```

If you have any problem regarding permission when using podman, try disabling the selinux running

```bash
setenforce 0
```

#### Using docker

```bash
# Clone this project
git clone https://github.com/luszczynski/test-drive-openshift.git && cd test-drive-openshift.git

# Run the workshopper container
docker run -it --rm -p 8080:8080 -v $(pwd)/parte-2-openshift-4x:/app-data \
              -e CONTENT_URL_PREFIX="file:///app-data" \
              -e LOG_TO_STDOUT=true \
              -e WORKSHOPS_URLS="file:///app-data/_workshop1.yml" \
              quay.io/osevg/workshopper
```

### Install doc on Openshift

Before beginning your workshop, install the documentation in your Openshift environment by running the following commands:

NOTE: Remember to change the URLs below according to your environment.

```bash
# Usually you do not need to change this URLs
WORKSHOP_URLS="https://raw.githubusercontent.com/luszczynski/test-drive-openshift/master/parte-2-openshift-4x/_workshop1.yml"
ISSUES_URL="https://github.com/luszczynski/test-drive-openshift/issues"

# Change these vars according to your environment
OPENSHIFT_MASTER_URL="https://console-openshift-console.apps.cluster-brasilia-d6ec.brasilia-d6ec.example.opentlc.com/"
ETHERPAD_URL="http://etherpad-etherpad.apps.cluster-brasilia-d6ec.brasilia-d6ec.example.opentlc.com/p/workshop"
TERMINAL_URL="https://terminal-terminal.apps.cluster-brasilia-d6ec.brasilia-d6ec.example.opentlc.com/"
OPENSHIFT_API_URL="https://api.cluster-brasilia-da5c.brasilia-da5c.example.opentlc.com:6443"

oc new-project workshopper --display-name="Workshopper"

oc new-app quay.io/osevg/workshopper --name=workshopper \
      -e WORKSHOPS_URLS=$WORKSHOP_URLS \
      -e ISSUES_URL=$ISSUES_URL \
      -e OPENSHIFT_MASTER_URL=$OPENSHIFT_MASTER_URL \
      -e ETHERPAD_URL=$ETHERPAD_URL \
      -e TERMINAL_URL=$TERMINAL_URL \
      -e OPENSHIFT_API_URL=$OPENSHIFT_API_URL \
      -e LOG_TO_STDOUT=true -n workshopper

oc expose svc/workshopper -n workshopper
```
