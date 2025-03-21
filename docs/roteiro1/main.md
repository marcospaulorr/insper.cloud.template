## Objetivo

O Roteiro 1 tem como objetivo introduzir o gerenciamento de hardware Bare-Metal usando a plataforma MaaS.

**Principais atividades:**
- Instalação e configuração do MaaS para gerenciar servidores físicos
- Preparação da rede do ambiente com roteador, switch e IPs fixos
- Padronização e cabeamento dos dispositivos para garantir comunicação entre os componentes

**Infraestrutura utilizada**
- 6 NUCs com diferentes configurações de CPU, RAM e armazenamento
- Switch e roteador

## Montagem do Roteiro

Os pontos "tarefas" são os passos que devem ser seguidos para a realização do roteiro. Eles devem ser claros e objetivos. Com evidências claras de que foram realizados.

### Tarefa 1

Instalando o MAAS e o UBUNTU:

Primeiramente, foi realizado a instalação do Ubuntu Server 22.04 na máquina NUC Main. Durante o processo, foi definido o hostname como main, o usuário como cloud e a senha clouds. Foi atribuido o IP fixo 172.16.0.3 e o DNS foi configurado 172.20.129.131. Após isso, foi instalado o MaaS.

<!-- termynal -->

``` bash
sudo snap install maas --channel=3.5/Stable
```
![image](https://github.com/user-attachments/assets/a55b0de4-efd1-4675-85ce-833c42f7fbe1)

/// caption
Dashboard do MAAS
///

Conforme ilustrado acima, a tela inicial do MAAS apresenta um dashboard com informações sobre o estado atual dos servidores gerenciados. O dashboard é composto por diversos painéis, cada um exibindo informações sobre um aspecto específico do ambiente gerenciado. Os painéis podem ser configurados e personalizados de acordo com as necessidades do usuário.

### Tarefa 2

Fazendo o acesso remotamente:

Após a instalação do MaaS, acessei remotamente o MaaS dentro da rede local:
``` bash
ssh cloud@172.16.0.3
```
Foi escolhido esse endereço pois ele pertence à classe de IPs privados, que compreende o intervalo 172.16.0.0 - 172.31.255.255. Essa faixa é reservada para uso interno em redes privadas e não pode ser roteada diretamente para a internet, garantindo segurança. O roteador do kit foi configurado com o seguinte esquema de rede:
- IP do roteador (Gateway da rede): 172.16.0.1
- IP do Switch: 172.16.0.2
- IP escolhido para o NUC Main: 172.16.0.3

Em seguida, com o MaaS inicializado, foi acessado o painel web via navegador no seguinte endereço:
``` cpp
http://172.16.0.3:5240/MAAS
```
A configuração do intervalo de IPs reservados no DHCP do MaaS foi feita utilizando o seguinte intervalo:
``` 
172.16.11.1 - 172.16.14.255
```
Esse foi o intervalo pois a máscara de sub-rede é /20, o endereço de rede é 172.16.0.0 e o IP de broadcast é 172.16.15.255.

Para permitir o acesso externo ao servidor Main, foi configurado um NAT no roteador (que permite que dispositivos dentro de uma rede privada se comuniquem com redes externas) para encaminhar conexões externas via porta 22 (porta da Main). No NAT foi escolhido o seguinte IP para acessar romotamente o nosso kit: 10.103.1.28.
``` 
0.0.0.0/0
```

## App

### Tarefa 1
Primeiramente, foi mudado o DNS Server para 172.20.129.131. Esse endereço foi utilizado pelo fato de ser um servidor DNS interno do Insper.
![image](https://github.com/user-attachments/assets/c4552b0c-27f6-424b-9f9a-38615c21953b)
O print acima mostra a verificação do status do PostgreSQL com o comando "sudo systemctl status postgresql" que mostra que o PostgreSQL está ativo. Em seguida, foi realizado o teste de conexão local ao PostgreSQL com o comando "psql -U cloud -h 172.16.15.0 -d tasks" (posteriormente o IP da server 1 foi mudada para 172.16.0.50), esse comando se conecta ao usuário cloud, depois se conecta ao IP do server 1 e depois se conecta ao banco de dados tasks. Para testar a conexão remota ao PostgreSQL via telnet com o comando "telnet 172.16.15.0 5432", o telnet testa se conseguimos abrir na porta 5432 do IP 172.16.15.0. A porta 5432 pelo fato de ser a porta padrão do PostgreSQL.
![image](https://github.com/user-attachments/assets/c883a7d8-4d14-4474-974c-4132dfff80d7)
No print acima foi executado o comando "nmap localhost" que serve para escanear as portas abertas no localhost (server1) que fornece as portas 22 (SSH) e a porta 5432 (PostgreSQL).
![image](https://github.com/user-attachments/assets/beec6260-0767-40ee-8df1-c1869d58f412)
O print acima mostra o comando "ping 127.0.0.1" que serve para testar a conectividade com o próprio servidor. (127.0.0.1 é o IP do localhost).
Os prints abaixo representam os mesmos de cima, mas com os demais servers.
![image](https://github.com/user-attachments/assets/ff3e185f-7383-4e02-b86f-067b9fcf2ce1)
![image](https://github.com/user-attachments/assets/2c980c0e-7494-4dcb-8128-84b452f509b9)
![image](https://github.com/user-attachments/assets/81fa09f6-3e0e-4ccb-bc8a-bc3d0381eca9)
![image](https://github.com/user-attachments/assets/5657c28c-8b11-4a8c-9e31-ee7368014180)
![image](https://github.com/user-attachments/assets/a73bc286-e60f-44eb-95fd-c7dd4a2dc0b7)
![image](https://github.com/user-attachments/assets/b9b4fc25-9752-4f71-b675-74cc948525b6)
![image](https://github.com/user-attachments/assets/565365ad-8fa8-4e50-9cb6-7fb4f52733c9)



### Tarefa 2

Exemplo de diagrama

```mermaid
architecture-beta
    group api(cloud)[API]

    service db(database)[Database] in api
    service disk1(disk)[Storage] in api
    service disk2(disk)[Storage] in api
    service server(server)[Server] in api

    db:L -- R:server
    disk1:T -- B:server
    disk2:T -- B:db
```

[Mermaid](https://mermaid.js.org/syntax/architecture.html){:target="_blank"}

## Questionário, Projeto ou Plano

Esse seção deve ser preenchida apenas se houver demanda do roteiro.

## Discussões

Quais as dificuldades encontradas? O que foi mais fácil? O que foi mais difícil?

## Conclusão

O que foi possível concluir com a realização do roteiro?
