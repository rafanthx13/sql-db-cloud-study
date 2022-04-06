# Ubuntu - Instalar e usar MySql

## O que é o MySql

É um banco de dados Relacional

## O que é o MySQL WorkBench

É uma ferramenta visual GUI para gerir os dados do MySQL.

No Ubunut, depois de instlaado, pode ser achado buscando no simbolo `ubuntu` por mysql como um programa comun.

#### Uso `LocalHost`

**Nâo Confunda**: Se vocÊ usar o MySQL local Você deve dar `start` primeiro no mysql ante de usar o workbench. 

Para iniciar o mysql no ubuntu é 

````
systemctl status mysql.service
````

Só então, você sabo o banco de dados e pode se conectar com o WorkBench

**STOP SERVER**

````
systemctl stop  mysql.service
````

#### Uso `Remote`

Não precisa startar o mysql na sua máquina, basta se conectar com o banco externo. Acaba sendo necessário maisconfiguração no WOrkBench como SSH e porta/ip pois são externos.

## Como instalar no Ubuntu

Segue os links

https://www.digitalocean.com/community/tutorials/como-instalar-o-mysql-no-ubuntu-18-04-pt
https://abbysays.wordpress.com/2008/05/20/how-to-startstop-mysql-server-on-ubuntu-804/

Senha padrão: root
