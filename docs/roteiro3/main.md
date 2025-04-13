## Objetivo
Esse roteiro tem como objetivo criar, configurar e utilizar uma infraestrutura de nuvem privada usando o OpenStack, orquestrado pelo Juju e provisionando via MAAS.
O foco é compreender como escalar recursos computacionais com VMs, configurar redes virtuais (SDN) e hospedar aplicações completas com alta disponibilidade.
Até o roteiro 2, foi trabalhado com o MAAS e aplicações como Juju, mas para ambientes corporativos usam virtualização para otimizar recursos e escalabilidade. O OpenStack 
permite gerenciar essas VMs.

## Infra
Primeiramente, foi definido os papéis das máquinas.

server1: controller (coordena os serviços principais da nuvem, como dashboard, autenticação, agendamente, etc)

server2: reserva

server3, server4, server5: compute (hospeda as VMs dos usuários, quanto mais servidores compute, mais capacidade de criação de VMs)

Após isso, foi criado o Juju Controller, pois o Juju precisa de uma máquina exclusiva para atuar como controller que é responsável por gereciar o modelo de deploy, orquestrar
os charms e manter o estado das aplicações. Foi usado o seguinte código:
``` bash
juju bootstrap --bootstrap-series=jammy --constraints tags=controller maas-one maas-controller
```
Em seguida, foi criado o modelo "openstack", pois com a criação de um modelo chamado openstack permite separar os recursos e serviços da infraestrutura OpenStack. Isso
garante que todos os serviços que forem implatados fiquem agrupados dentro do modelo openstack.
``` bash
juju add-model --config default-series=jammy openstack
juju switch maas-controller:openstack
```
Depois disso, foi realizado o deploy dos serviços OpenStack com Juju. Os seguintes serviços são instalados em containers LXD:

- Ceph : Sistema de armazenamento distribuído altamente disponível.
- Nova : Serviço que provisiona as máquinas virtuais (VMs) no OpenStack.
- MySQL InnoDB Cluster : Cluster de banco de dados relacional.
- Vault : Gerenciador de certificados e segredos (senha, tokens, chaves TLS).
- Neutron : Serviço de rede virtual (SDN).
- Keystone : Sistema de autenticação e autorização.
- RabbitMQ : Sistema de mensageria (fila de mensagens AMQP).
- Nova Cloud Controller : Componente que gerencia centralmente o Nova.
- Placement : Serviço que gerencia onde as VMs serão alocadas.
- Horizon : Interface web (dashboard) do OpenStack.
- Glance : Serviço de gerenciamento de imagens.
- Cinder : Serviço de disco em bloco (volumes persistentes).
- Ceph RADOS Gateway : Interface S3/Swift compatível do Ceph.

