
# HA-DGEG-Combustiveis
Node-RED flow que importa o preço dos combustiveis do site da DGEG para base de dados MariaDB, permitindo depois obter os preços mais baratos com base em coordenadas GPS, criando vários sensores no Home Assistant

## Como funciona?

![mermaid](/images/mermaid.PNG "mermaid")

## Requisitios:
- Home Assistant
- Addons:
	- NodeRED
		- Instalar [node-red-contrib-stackhero-mysql](https://flows.nodered.org/node/node-red-contrib-stackhero-mysql)
	- MariaDB
- Custom Components:
	- Node-RED Companion Integration (HACS)

## Instalação
Todos os requisitos têm de estar instalados e configurados.
***
**MariaDB**

Na configuração do Addon MariaDB é necessário adicionar uma nova base de dados e um login para a mesma.

> o nome da nova base de dados tem mesmo de ser `combustiveis`
> 
> não esquecer de atribuir os `rights` para o login criado

Deverá ficar algo deste género
```
databases:
  - homeassistant
  - combustiveis
logins:
  - username: homeassistant
    password: my_really_long_password
  - username: nodered
    password: my_really_long_password
rights:
  - username: homeassistant
    database: homeassistant
  - username: nodered
    database: combustiveis
```

- Reiniciar o addon MariaDB
***
**NodeRED**

Importar o json que se encontra na pasta `nodered` [ha-dged-combustiveis.json](/nodered/ha-dged-combustiveis.json "ha-dged-combustiveis.json")

Este json irá criar um flow **`HA-Combustiveis`** e vários sub-flows de suporte

- É necessário configurar o node `ha-combustiveis`

![node-ha-combustiveis](/images/node-ha-combustiveis.PNG "node-ha-combustiveis")

![node-ha-combustiveis-config](/images/node-ha-combustiveis-config.PNG "node-ha-combustiveis-config")

Nos campos `user` e `password` colocar os dados do utilizador criado anteriormente no addon MariaDB

No campo `database` colocar `combustiveis`, que deverá ser a database criada no addon MariaDB

- Fazer **`Deploy`** no NodeRED e após isso iniciar o node `One Time Only`, que irá criar a estrutura de tabelas necessária na base de dados
	> Este processo é só necessário correr a primeira vez após a instalação
	
- Após a estrutura de tabelas ser criada, devemos iniciar o `Update Distritos`
- Esperar que fique no estado `green`<br>![node-distritos-finish](/images/node-distritos-finish.PNG "node-distritos-finish")
***
- Agora iniciamos o flow `Update Postos` e aguardamos que o também fique no estado `Finished`
	>Enquanto este flow está a correr, temos a informação do distrito que está a ser processado
	>
	>Este flow pode demorar alguns minutos a correr
	

***
- Quando o flow anterior terminar, podemos então iniciar o `Update Precos from DGEG`
> O flow `Update Precos from DGEG` demora vários minutos a correr e podemos acompanhar o estado do mesmo vendo quantos postos já foram atualizados
> 
> ![node-update-precos](/images/node-update-precos.PNG "node-update-precos")


***
***Sensores Home Assistant***

Neste momento já temos toda a informação atualizada na nossa base de dados, vamos configurar os sensores a serem criados no Home Assistant

É logo criado um sensor `sensor.marcas_combustiveis` que tem como atributo a lista de todas as marcas possíveis de serem configuradas nos descontos.

Para configurar os sensores, apenas temos de alterar os nodes `CONFIG`

![node-config](/images/node-config.PNG "node-config")

![node-config-open](/images/node-config-open.PNG "node-config-open")

|Const          |Exemplo       |Descrição|
|---------------|--------------|---------|
|entity_id      |zone.home     |Entidade do Home Assistant que tenha Longitude e Latitude como atributos (ex: zonas e device_trackers)
|combustivel    |DIESEL        |Qual o tipo de combustível que queremos usar (DIESEL, GASOLINA, GPL)
|distancia      |4             |Raio de distância (em km) com base nas coordenadas do `entity_id`
|not_marcas     |BP,GALP       |Lista de marcas a excluir dos resultados, separada por virgulas
|descontos      |`JSON`        |Lista em formado JSON com possíveis descontos em combustível

**Descontos**
|Campo       |Exemplo    |Descrição  |
|------------|-----------|-----------|
|marca       |REPSOL     |Marca de combustível (ver `sensor.marcas_combustiveis`)
|desconto    |0.12       |Desconto por litro em euros

---

**ToDo**

- Adicionar exemplo de ações e notificações mobile (iOS e Android)<br>![notification-ios](/images/ios-notification.png "notification-ios")
- ~~Adicionar lista para excluir Marcas dos resultados~~
- Adicionar opção de carregadores electricos

---
**NOTA:** Este projecto não tem qualquer relação com a DGEG e apenas irá funcionar enquanto o formato da API não for alterado



## Thank you

<a href="https://www.buymeacoffee.com/denkyem" target="_blank"><img src="https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png"></a>
