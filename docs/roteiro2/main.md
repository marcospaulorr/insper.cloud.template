## Objetivo
Esse roteiro tem como objetivo automatizar todo o ciclo de deploy de aplicações bare-metal, usando Juju como orquestrador, integrado com o MAAS, que continua sendo
seu gerenciador de máquinas físicas

# Criação da infraestrutura com o Juju
Foi realizado a instalação do Juju via snap, usando o canal da versão 3.6.
``` bash
sudo snap install juju --channel 3.6
```

Em seguida, foi realizado a verificação se o MAAS aparece como provedor (cloud). Esse código lista as clouds conhecidas pelo Juju, deve existir o maas na lista. Antes de usar o MAAS como backend para deploy, o Juju precisa saber que ele existe como provedor de infraestrutura.
``` bash
juju clouds
```
![image](https://github.com/user-attachments/assets/b695220d-1d6d-4e3a-a144-ae5d8ed36e6e)


Após isso, foi criado o arquivo maas-cloud.yaml, que define uma nova cloud chamada "maas-one" que será conectada ao MAAS via endpoint. É necessário registrar a nossa instância do MAAS para que o Juju consiga provisionar máquinas através dela.

``` bash
clouds:
  maas-one:
    type: maas
    auth-types: [oauth1]
    endpoint: http://172.16.0.3:5240/MAAS/
```

Posteriormente, foi adicionado o MAAS como cloud cliente. É necessário adicionar a cloud manualmente.
``` bash
juju add-cloud --client -f maas-cloud.yaml maas-one
```
Depois foi necessário adicionar as credenciais de autenticação com o MAAS. A API_KEY é obtida no dashboard do MAAS nas configurações do usuário.
```bash
credentials:
  maas-one:
    anyuser:
      auth-type: oauth1
      maas-oauth: <API_KEY>
```
Comando de adicionar as credenciais ao Juju
```bash
juju add-credential --client -f maas-creds.yaml maas-one
```
Após isso a server1 foi marcada com a tag "juju". A partir disso o server1 é a máquina que será usada como controlador do Juju.
Em seguida, foi criado o controlador Juju (chamado maas-controler) usando a máquina com a tag "juju". Essa máquina será usada para orquestrar os próximos deploys com Juju. O bootstrap indica a versão do Ubuntu usada.
```bash
juju bootstrap --bootstrap-series=jammy --constraints tags=juju maas-one maas-controller
```
![image](https://github.com/user-attachments/assets/729c4a4e-d476-4115-b4d3-c66c4950eb4a)

Foi verificado o status das máquinas, aplicações e do próprio controlador da juju com o seguinte código:
```bash
juju status
```
Validando que o controlador já foi criado com sucesso e o ambiente está pronto para deploy.
![image](https://github.com/user-attachments/assets/044d500b-9767-4eef-a1eb-78b2d462fd5b)

# Tarefa 1
Nessa tarefa será usado o Juju para orquestrar o deploy automatizado de duas aplicações: Grafana (visualização) e Prometheus (coleta de dados).

Primeiramente, foi criado a pasta para armazenar os arquivos ".charm" das aplicações, o que facilita o deploy com Juju de charms locais. Em seguida, foi baixado
o charm do Grafana e o charm do Prometheus. Os charms tornam o deploy automatizado e padronizado. Para acompanhar o status do deploy, acompanhar a instalação em tempo real e verificar se as aplicações estão ficando "active" foi utilizado o seguinte código:
```bash
watch -n 1 juju status
```
Depois foi realizada a conexão entre Grafana e o Prometheus, essa integração é necessária para que o Grafana consiga exibir os dados coletados pelo Prometheus
```bash
juju relate grafana prometheus2
```





