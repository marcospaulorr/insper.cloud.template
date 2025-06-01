# Roteiro 4: Infraestrutura como Código (IaC) com Terraform e OpenStack

## Objetivo

Este roteiro tem como objetivo implementar os conceitos de Infraestrutura como Código (IaC) utilizando Terraform para provisionar recursos no OpenStack. O foco é entender como gerenciar infraestrutura através de código declarativo, criar hierarquia de projetos para organização multi-tenant e implementar boas práticas de IaC.

## Parte 1: Entendendo IaC - Visão Geral

### O que é Infraestrutura como Código?

IaC permite gerenciar e provisionar infraestrutura através de arquivos de configuração ao invés de processos manuais. Com IaC, você escreve código que descreve a infraestrutura desejada, garantindo consistência, versionamento e recuperação rápida.

### Por que Terraform?

O Terraform é uma ferramenta de IaC da HashiCorp que funciona com múltiplos provedores (AWS, Azure, OpenStack), usa linguagem declarativa (HCL) e mantém estado da infraestrutura, sendo idempotente.

### Instalação do Terraform

```bash
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform
```

## Parte 2: Criando Hierarquia de Projetos no OpenStack


### Criação via Dashboard Horizon

## Conclusão - Questões

### 1. Você é o CTO de uma grande empresa com sede em várias capitais no Brasil e precisa implantar um sistema crítico, de baixo custo e com dados sigilosos para a área operacional.

#### a) Você escolheria Public Cloud ou Private Cloud?

**Resposta:** Private Cloud.
Para um sistema crítico com dados sigilosos, a Private Cloud oferece controle total sobre a infraestrutura, garantindo:
- Conformidade com regulamentações brasileiras (LGPD)
- Isolamento físico dos dados sensíveis
- Customização específica para requisitos de segurança
- Menor risco de vazamento por vulnerabilidades de terceiros
- Auditoria completa sobre acesso aos dados

Embora o custo inicial seja maior, o TCO (Total Cost of Ownership) se justifica pela criticidade e sigilo dos dados operacionais.

#### b) Agora explique para o RH por que você precisa de um time de DevOps.


"Precisamos de um time DevOps porque nossa infraestrutura crítica exige:

1. **Redução de Erros**: Com IaC (como vimos no Terraform), automatizamos deploys que antes eram manuais, reduzindo erros humanos de 70% para menos de 5%.

2. **Velocidade de Entrega**: Um deploy manual levava 4 horas. Com DevOps e automação, fazemos em 15 minutos.

3. **Disponibilidade 24/7**: O time implementa monitoramento proativo e resposta automatizada a incidentes, garantindo SLA de 99.9%.

4. **Economia**: Apesar do investimento inicial em salários, economizamos em:
   - Menos downtime 
   - Menos retrabalho
   - Melhor uso de recursos

5. **Segurança**: DevSecOps integra segurança desde o início, não como afterthought.

O ROI é de 6 meses, considerando apenas redução de incidentes."

#### c) Monte um plano de DR e HA considerando:

**i. Mapeamento das principais ameaças:**

1. **Falhas de Hardware** (Probabilidade: Alta, Impacto: Alto)
   - Disco rígido
   - Servidor físico
   - Switch/Roteador

2. **Erros Humanos** (Probabilidade: Média, Impacto: Alto)
   - Delete acidental de dados
   - Configuração incorreta
   - Deploy com bugs

3. **Ataques Cibernéticos** (Probabilidade: Média, Impacto: Crítico)
   - Ransomware
   - DDoS
   - Invasão/Roubo de dados

4. **Desastres Naturais** (Probabilidade: Baixa, Impacto: Crítico)
   - Incêndio
   - Inundação
   - Falta de energia prolongada

**ii. Ações priorizadas para recuperação:**

1. **Prioridade 1 - Primeiros 30 minutos:**
   - Ativar site de DR (pré-aquecido)
   - Redirecionar tráfego via DNS
   - Notificar equipe de crise

2. **Prioridade 2 - Primeira hora:**
   - Restaurar serviços críticos do backup mais recente
   - Validar integridade dos dados
   - Comunicar stakeholders

3. **Prioridade 3 - Primeiras 4 horas:**
   - Restaurar serviços secundários
   - Análise de causa raiz
   - Plano de retorno ao site principal

**iii. Política de Backup:**

```
Estratégia 3-2-1:
- 3 cópias dos dados (produção + 2 backups)
- 2 mídias diferentes (disco + tape/cloud)
- 1 cópia offsite (outro datacenter)



Testes:
- Restore parcial: Semanal
- Restore completo: Mensal
- DR completo: Trimestral
```

**iv. Implementação de Alta Disponibilidade:**

```
Arquitetura HA Multi-Camada:

1. Load Balancer (2x HAProxy)
   - Heartbeat entre eles
   - VIP compartilhado
   - Health check a cada 10s

2. Aplicação (4x instâncias)
   - 2 em cada site
   - Auto-scaling baseado em CPU/RAM
   - Session replication

3. Banco de Dados (Cluster 3 nós)
   - 1 Master + 2 Slaves
   - Replicação síncrona
   - Automatic failover < 30s

4. Storage (Ceph com 3 réplicas)
   - Distribuído entre racks
   - Self-healing
   - 99.999% durabilidade

5. Rede (Redundância total)
   - 2 ISPs diferentes
   - Switches redundantes
   - Múltiplos paths (LACP)


```
![image](https://github.com/user-attachments/assets/2fa47a38-017c-4853-a5dd-62c9b946779d)
![image](https://github.com/user-attachments/assets/b943bcd9-774c-4ea1-af1e-5a49b97d93fe)
![image](https://github.com/user-attachments/assets/8f6c4290-0b97-4356-80ef-34ee1759e4ff)
![image](https://github.com/user-attachments/assets/96159f20-eb77-43ad-bc7a-0a23626648aa)
![image](https://github.com/user-attachments/assets/ed1b00ab-f86f-45a9-8374-2c6cdf2e1d66)
![image](https://github.com/user-attachments/assets/331617f2-d249-4986-b265-1473a65f8798)







