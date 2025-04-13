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
