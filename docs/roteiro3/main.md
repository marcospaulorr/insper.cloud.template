## Objetivo

Este roteiro tem como objetivo criar, configurar e utilizar uma infraestrutura de nuvem privada utilizando o OpenStack, orquestrado com Juju e provisionado com MAAS. O foco é compreender a criação e gestão de VMs, redes virtuais (SDN) e a implementação de aplicações em ambientes distribuídos com alta disponibilidade.

## Infraestrutura

### Definição dos papéis das máquinas:
- `server1`: controller
- `server2`: reserva (alocado, mas não utilizado inicialmente)
- `server3`, `server4`, `server5`: compute (onde as VMs serão hospedadas)

### Criação do Juju Controller e modelo:

```bash
juju bootstrap --bootstrap-series=jammy --constraints tags=controller maas-one maas-controller
juju add-model --config default-series=jammy openstack
juju switch maas-controller:openstack
```

Com isso, foi criada a base para a orquestração dos serviços do OpenStack via charms do Juju.

### Deploy dos serviços OpenStack com Juju:

Abaixo estão os principais serviços do OpenStack implantados via Juju, com seus respectivos comandos e explicações do papel de cada um:

Os seguintes serviços foram instalados via charms, respeitando a ordem de dependência:

#### Ceph OSD:
**O que faz:** Ceph OSD é o componente responsável pelo armazenamento distribuído de dados. Ele garante alta disponibilidade e resiliência ao armazenar cópias dos dados entre os nós compute.

**Por que é importante:** Essencial para fornecer backend de armazenamento para serviços como Glance (imagens), Nova (discos efêmeros) e Cinder (volumes persistentes).

```bash
juju deploy -n 3 --channel quincy/stable --config ceph-osd.yaml --constraints tags=compute ceph-osd
```

#### MySQL InnoDB Cluster:
**O que faz:** Fornece o banco de dados relacional que armazena informações de configurações e estados de diversos serviços do OpenStack.

**Por que é importante:** Serviços como Keystone, Nova, Neutron e Glance dependem do banco para funcionar corretamente. A configuração em cluster garante alta disponibilidade.

```bash
juju deploy -n 3 --to lxd:0,lxd:1,lxd:2 --channel 8.0/stable mysql-innodb-cluster
```

#### Nova Compute:
**O que faz:** É o serviço que gerencia a criação, execução e destruição de máquinas virtuais (VMs).

**Por que é importante:** Responsável por alocar instâncias em servidores físicos e permitir a escalabilidade da nuvem com múltiplos nós de computação.

```bash
juju deploy -n 3 --to 0,1,2 --channel yoga/stable --config nova-compute.yaml nova-compute
```

#### Vault:
**O que faz:** Gerencia segredos, como certificados TLS, tokens e chaves de acesso entre os serviços.

**Por que é importante:** Garante segurança nas comunicações entre serviços da nuvem. Ele também fornece certificados para uso interno (HTTPS).

```bash
juju deploy --to lxd:2 vault --channel 1.8/stable
juju run vault/leader generate-root-ca
juju integrate mysql-innodb-cluster:certificates vault:certificates
```

#### Neutron Networking:
**O que faz:** É o serviço de rede virtual (SDN) do OpenStack. Controla a criação de redes internas, roteadores, IPs flutuantes e conectividade externa.

**Por que é importante:** Permite que as VMs tenham conectividade entre si e com a rede externa. Sem ele, as máquinas ficariam isoladas.

```bash
juju deploy -n 3 --to lxd:0,lxd:1,lxd:2 --channel 22.03/stable ovn-central
juju deploy --to lxd:1 --channel yoga/stable --config neutron.yaml neutron-api
juju deploy --channel yoga/stable neutron-api-plugin-ovn
juju deploy --channel 22.03/stable --config neutron.yaml ovn-chassis
```

#### Horizon (Dashboard):
**O que faz:** Fornece uma interface gráfica web para gerenciar a nuvem OpenStack de forma intuitiva.

**Por que é importante:** Facilita a visualização e administração de recursos como instâncias, imagens, volumes e redes. Ideal para quem não quer usar apenas CLI.
```bash
juju deploy --to lxd:2 --channel yoga/stable openstack-dashboard
```

#### Keystone:
**O que faz:** É o serviço de autenticação e autorização do OpenStack. Controla usuários, tokens, permissões e projetos.

**Por que é importante:** Todos os outros serviços utilizam Keystone para autenticação. Sem ele, não há controle de acesso na nuvem.

#### RabbitMQ:
**O que faz:** É um sistema de mensageria baseado em filas AMQP. Permite a comunicação assíncrona entre os serviços da nuvem.

**Por que é importante:** Sem ele, os serviços como Nova, Neutron e Cinder não conseguem se comunicar corretamente.

#### Placement:
**O que faz:** Gerencia os recursos físicos disponíveis e informa ao Nova onde as VMs podem ser alocadas.

**Por que é importante:** Fundamental para o agendamento inteligente de instâncias com base em recursos como CPU, RAM e disco.

#### Glance:
**O que faz:** Serviço de armazenamento e gerenciamento de imagens (como Ubuntu, CentOS etc.) para criação de VMs.

**Por que é importante:** Sem ele, não é possível criar instâncias porque não haveria um sistema operacional para bootar.

#### Cinder:
**O que faz:** Fornece volumes de armazenamento persistentes que podem ser conectados às VMs.

**Por que é importante:** Permite o uso de volumes separados, snapshots e persistência de dados independente da vida útil da instância.

#### Ceph RADOS Gateway:
**O que faz:** Interface de acesso ao Ceph via APIs compatíveis com Swift e S3.

**Por que é importante:** Oferece um sistema de armazenamento de objetos, ideal para arquivos, backups e containers em nuvem.

## Tarefa 1 - Verificação Inicial

- [ ] Inserir print do comando `juju status` com todos os serviços ativos
    ![image](https://github.com/user-attachments/assets/84eecc40-ae18-419a-9d45-314dd71c3733)
- [ ] Inserir print do Dashboard do MAAS com as máquinas e IPs
    ![image](https://github.com/user-attachments/assets/0f71a47d-7850-4eca-9ed6-4d96fb1b79c9)
- [ ] Inserir print da aba "Compute > Overview" no Horizon
- [ ] Inserir print da aba "Compute > Instances" no Horizon
- [ ] Inserir print da aba "Network > Topology" no Horizon

## Tarefa 2 - Configuração da Nuvem e Primeira Instância

- Rede externa criada: faixa 172.16.7.0/24
- Rede interna criada: faixa 192.169.0.0/24
- Imagem Ubuntu Jammy importada
- Flavors criados: m1.tiny, m1.small, m1.medium, m1.large

#### Criar flavor:
```bash
openstack flavor create m1.tiny --vcpus 1 --ram 1024 --disk 20
```

#### Importar imagem:
```bash
openstack image create "ubuntu-jammy" --file ubuntu-jammy.img --disk-format qcow2 --container-format bare --public
```

#### Criar rede e roteador:
Executado via Horizon ou comandos OpenStack CLI conforme documentação.

- [ ] Inserir print atualizado do Dashboard do MAAS
- [ ] Inserir print atualizado do Horizon: "Compute > Overview"
- [ ] Inserir print de "Compute > Instances" com a nova instância
- [ ] Inserir print da "Network Topology" com roteador e redes conectadas
- [ ] Responder: Enumere as diferenças entre os prints da Tarefa 1 e 2
- [ ] Responder: Explique como cada recurso foi criado (imagem, flavor, redes, instância)

## Tarefa 3 - Escalando os Nós

- O server2 foi liberado e adicionado como novo compute node:

```bash
juju add-unit nova-compute
```

- Em seguida, foi adicionado como Ceph OSD:

```bash
juju add-unit --to <machine-id> ceph-osd
```

- [ ] Inserir esquema da arquitetura de rede, do Insper até a instância na nuvem

## Tarefa 4 - Deploy de Aplicação em VMs

Foram criadas 4 VMs com a seguinte topologia:
- 2 x API (backend)
- 1 x Banco de dados (PostgreSQL)
- 1 x Load Balancer (NGINX)

Endereçamento:
- Rede privada: 192.169.0.0/24
- IPs flutuantes atribuídos a cada instância

- [ ] Inserir print da arquitetura de rede no Horizon
- [ ] Inserir print da lista de VMs com nome e IPs
- [ ] Inserir print do dashboard da aplicação acessada via NGINX
- [ ] Inserir 4 prints de qual servidor físico cada instância foi alocada
- [ ] Escrever os passos do deploy da aplicação (infra, rede, configuração, testes)

## Conclusão

Esse roteiro permitiu compreender como montar uma nuvem privada completa com OpenStack, incluindo armazenamento distribuído, rede SDN, gerenciamento de instâncias, dashboard de gestão e deploy de aplicações reais.

Ao final, a estrutura permite escalabilidade, balanceamento de carga, separação de serviços e controle centralizado, replicando a arquitetura de nuvens comerciais com recursos locais gerenciados via MAAS e Juju.

