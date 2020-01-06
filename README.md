# Introdução

## O que é isso?

Esse material é usado para capacitação e transferência de conhecimento de clientes e parceiros da Red Hat Brasil em **OpenShift**, aplicável tanto ao [**Red Hat OpenShift Container Platform**](https://www.openshift.com/container-platform/index.html) \(enterprise\) quanto ao [**Openshift OKD**](https://www.okd.io/) \(community\).

## Por que dessa forma?

Inspirados na cultura open-source, acreditamos que, ao disponibilizar o material de forma aberta, podemos **evoluir o workshop de forma colaborativa e alinhado com as necessidades dos nossos clientes, parceiros e a comunidade**.

Este trabalho está licenciado sob a [**Licença Atribuição-NãoComercial-CompartilhaIgual 4.0 Internacional Creative Commons \(CC BY-NC-SA 4.0\)**](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.pt_BR).

## Como usar?

**Não existe uma maneira certa ou errada de usar o conteúdo deste material.** Você poderia segur como um roteiro independente, usando a sua máquina particular como ambiente de experimentações, ou você pode se reunir com colegas e compartilhar a experiência.

**Se você tiver interesse de participar de um workshop com o time Red Hat Brasil, entre em contato com o seu time de contas/parceiro!**

Para conseguir aproveitar o material, nossa recomendação é:

* _Reserve entre **4-8 horas** para discussão dos assuntos e execução das atividades._
* _Acesso à **Internet é indispensável**._
* _Caso opte por usar uma VM local recomendamos que os participantes utilizem **um PC capaz de rodar máquinas virtuais**:_
  * _**Processador dual-core de 2GHz**_
  * _**8GB de memória RAM**_
  * _**40GB de disco livre**_
  * _**Virtualizador \(QEMU/KVM, Oracle VirtualBox, VMware Workstation\)**_

## Por onde começar?

Esse material é dividido em 2 partes:

* [**Parte 1 - Linux Containers:**](parte-1-containers/) _Material dedicado à discussão introdutória sobre a tecnologia de containers Linux._
* [**Parte 2 - OpenShift**](parte-2-openshift/)**:** _Material dedicado à discussão sobre o OpenShift na perspectiva de desenvolvimento e também infraestrutura._

## Posso contribuir?

**Aceitamos contribuições de todas as formas!** Você pode fazer uso da funcionalidade de "Issues" direto do repositório fonte no [**Github**](https://github.com/redhat-sa-brazil/workshop-openshift). Uma outra alternativa é através das funcionalidades de colaboração do próprio [**GitBook**](https://redhat-sa-brazil.gitbooks.io/workshop-openshift). Fique a vontade de usar qualquer um dos dois!


## Observação

**Para o S2I com o Quarkus** funcionar importar o template template-openjdk11-rhel8-s2i.yaml para o namespace **openshift**. Após isso importar também a secret para a service account default em cada projeto para fazer pulling da imagem do **registry.redhat.io**.

## Workshopper

### Local

```bash
podman run -it --rm -p 8080:8080 -v $(pwd)/parte-2-openshift-4x:/app-data \
              -e CONTENT_URL_PREFIX="file:///app-data" \
              -e LOG_TO_STDOUT=true \
              -e WORKSHOPS_URLS="file:///app-data/_workshop1.yml" \
              quay.io/osevg/workshopper
```

### Install on Openshift

```bash
oc new-project workshopper --display-name="Workshopper"
oc new-app quay.io/osevg/workshopper --name=workshopper \
      -e WORKSHOPS_URLS="https://raw.githubusercontent.com/luszczynski/test-drive-openshift/master/parte-2-openshift-4x/_workshop1.yml" \
      -e ISSUES_URL=https://github.com/luszczynski/test-drive-openshift/issues \
      -e OPENSHIFT_MASTER_URL=https://console-openshift-console.apps.cluster-brasilia-d6ec.brasilia-d6ec.example.opentlc.com/ \
      -e ETHERPAD_URL=http://etherpad-etherpad.apps.cluster-brasilia-d6ec.brasilia-d6ec.example.opentlc.com/p/workshop \
      -e TERMINAL_URL=https://terminal-terminal.apps.cluster-brasilia-d6ec.brasilia-d6ec.example.opentlc.com/ \
      -e LOG_TO_STDOUT=true -n workshopper

oc expose svc/workshopper -n workshopper
```

### Convert Markdown to Asciidoc

```bash
doc=configmap-e-secrets; podman run -v `pwd`:/source jagregory/pandoc --atx-headers --normalize --verbose --wrap=none --reference-links -s -S -t asciidoc -o parte-2-openshift-4x/$doc.adoc parte-2-openshift-4x/$doc.md
```

### Etherpad

```bash
oc new-project etherpad --display-name "Shared Etherpad"

oc new-app mysql-persistent --param MYSQL_USER=ether --param MYSQL_PASSWORD=ether --param MYSQL_DATABASE=ether --param VOLUME_CAPACITY=2Gi --param MYSQL_VERSION=5.7 -n etherpad

sleep 15

# oc new-app -f etherpad-template.yaml -p DB_USER=ether -p DB_PASS=ether -p DB_DBID=ether -p DB_PORT=3306 -p DB_HOST=mysql -p ADMIN_PASSWORD=secret
oc new-app -f https://raw.githubusercontent.com/wkulhanek/docker-openshift-etherpad/master/etherpad-template.yaml -p DB_USER=ether -p DB_PASS=ether -p DB_DBID=ether -p DB_PORT=3306 -p DB_HOST=mysql -p ADMIN_PASSWORD=secret -n etherpad
```

* https://github.com/wkulhanek/docker-openshift-etherpad