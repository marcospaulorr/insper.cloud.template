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

- [ ] Print do comando `juju status` com todos os serviços ativos
    ![image](https://github.com/user-attachments/assets/84eecc40-ae18-419a-9d45-314dd71c3733)
- [ ] Print do Dashboard do MAAS com as máquinas e IPs
    ![image](https://github.com/user-attachments/assets/0f71a47d-7850-4eca-9ed6-4d96fb1b79c9)
- [ ] Print da aba "Compute > Overview" no Horizon
   ![image](https://github.com/user-attachments/assets/20c82165-c65b-4347-b7a7-8183a3d101c6)
- [ ] Print da aba "Compute > Instances" no Horizon
      ![image](https://github.com/user-attachments/assets/81af6e7d-18c0-441e-b3d7-ca441bed9289)
- [ ] Print da aba "Network > Topology" no Horizon
      ![image](https://github.com/user-attachments/assets/f227ac7e-9436-40f5-9bfe-c2eae3cb2f2f)


## Tarefa 2 - Configuração da Nuvem e Primeira Instância
![image](https://github.com/user-attachments/assets/6e249c86-37d4-4749-ae26-897740f7536f)
![image](https://github.com/user-attachments/assets/7085127c-07ee-403a-bb2f-0921f51703ad)
![image](https://github.com/user-attachments/assets/35acbeae-e321-4bb0-9358-6572a5ca5f43)
![image](https://github.com/user-attachments/assets/b2e300e0-048d-4b88-86f1-e47a53d8af57)
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

subgraph Insper
  A[Usuário na rede Wi-Fi do Insper] -->|SSH via túnel (porta 22)| B[Main (cloud)]

subgraph Nuvem Privada
  B -->|Túnel SSH| C[Dashboard Horizon (porta 80/443)]
  B -->|Túnel SSH| D[Instância Load Balancer - NGINX]
  D -->|Proxy Reverso| E1[Instância API 1]
  D -->|Proxy Reverso| E2[Instância API 2]
  E1 --> F[Instância Banco de Dados]
  E2 --> F

subgraph Redes Virtuais OpenStack
  D ---|Rede Interna (192.169.0.0/24)| E1
  D --- E2
  D --- F
  D -->|Floating IP via Rede Externa (172.16.7.0/24)| G[Usuário Externo (Acesso via IP Público)]

## Tarefa 4 - Deploy de Aplicação em VMs

Foram criadas 4 VMs com a seguinte topologia:
- 2 x API (backend)
- 1 x Banco de dados (PostgreSQL)
- 1 x Load Balancer (NGINX)

Endereçamento:
- Rede privada: 192.169.0.0/24
- IPs flutuantes atribuídos a cada instância

![image](https://github.com/user-attachments/assets/2e190fac-d944-4e3d-8f0d-05a51ef98642)
![image](https://github.com/user-attachments/assets/b8579103-0e3a-49c7-9a6f-866cb19f7bbd)
![image](https://github.com/user-attachments/assets/b9681201-b4dd-44ae-90ec-8954027227bb)
![image](https://github.com/user-attachments/assets/e40795a7-04ce-4314-b22b-f404e4f4645b)

## Conclusão

Esse roteiro permitiu compreender como montar uma nuvem privada completa com OpenStack, incluindo armazenamento distribuído, rede SDN, gerenciamento de instâncias, dashboard de gestão e deploy de aplicações reais.

Ao final, a estrutura permite escalabilidade, balanceamento de carga, separação de serviços e controle centralizado, replicando a arquitetura de nuvens comerciais com recursos locais gerenciados via MAAS e Juju.

