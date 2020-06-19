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

## Workshopper

### Local

#### Using podman

```bash
podman run -it --rm -p 8080:8080 -v $(pwd)/parte-2-openshift-4x:/app-data \
              -e CONTENT_URL_PREFIX="file:///app-data" \
              -e LOG_TO_STDOUT=true \
              -e WORKSHOPS_URLS="file:///app-data/_workshop1.yml" \
              quay.io/osevg/workshopper
```

#### Using docker

```bash
docker run -it --rm -p 8080:8080 -v $(pwd)/parte-2-openshift-4x:/app-data \
              -e CONTENT_URL_PREFIX="file:///app-data" \
              -e LOG_TO_STDOUT=true \
              -e WORKSHOPS_URLS="file:///app-data/_workshop1.yml" \
              quay.io/osevg/workshopper
```

### Install on Openshift

Remember to change the URLs below according to your environment.

```bash
oc new-project workshopper --display-name="Workshopper"
oc new-app quay.io/osevg/workshopper --name=workshopper \
      -e WORKSHOPS_URLS="https://raw.githubusercontent.com/luszczynski/test-drive-openshift/master/parte-2-openshift-4x/_workshop1.yml" \
      -e ISSUES_URL=https://github.com/luszczynski/test-drive-openshift/issues \
      -e OPENSHIFT_MASTER_URL=https://console-openshift-console.apps.cluster-brasilia-d6ec.brasilia-d6ec.example.opentlc.com/ \
      -e ETHERPAD_URL=http://etherpad-etherpad.apps.cluster-brasilia-d6ec.brasilia-d6ec.example.opentlc.com/p/workshop \
      -e TERMINAL_URL=https://terminal-terminal.apps.cluster-brasilia-d6ec.brasilia-d6ec.example.opentlc.com/ \
      -e OPENSHIFT_API_URL=https://api.cluster-brasilia-da5c.brasilia-da5c.example.opentlc.com:6443 \
      -e LOG_TO_STDOUT=true -n workshopper

oc expose svc/workshopper -n workshopper
```

### Convert Markdown to Asciidoc

```bash
doc=configmap-e-secrets; podman run -v `pwd`:/source jagregory/pandoc --atx-headers --normalize --verbose --wrap=none --reference-links -s -S -t asciidoc -o parte-2-openshift-4x/$doc.adoc parte-2-openshift-4x/$doc.md
```

### Etherpad

To install etherpad go to https://github.com/luszczynski/openshift-etherpad

### Terminal

If you need a terminal inside Openshift, run:

```bash
oc new-project terminal-workshop
oc process -f https://raw.githubusercontent.com/openshift-homeroom/workshop-spawner/develop/templates/terminal-server-production.json --param SPAWNER_NAMESPACE=`oc project --short` --param CLUSTER_SUBDOMAIN=apps.cluster-brasilia-da5c.brasilia-da5c.example.opentlc.com | oc apply -f -
```