# PostgreSQL no Unbuntu

## Links

https://www.devmedia.com.br/instalacao-e-configuracao-do-servidor-postgresql-no-linux/26184
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-18-04-pt
https://www.liquidweb.com/kb/what-is-the-default-password-for-postgresql/#:~:text=there%20isn't%20a%20default,PostgreSQL%20is%20set%20to%20ident.

## Comandos no Ubuntu

verifica se esta funcionando

````
sudo systemctl status postgresql
sudo systemctl status postgresql.service
````

Usando postgre em CMD (funciona em 04/01/2022)

````
sudo -u postgres psql
````

Dar restart

````
sudo service postgresql restart
````

## Arquivos de configuração do Postgre `pg_hba.conf`

Os arquivos de configuração do PostgreSQL estarão disponível na seguinte pasta: `/etc/postgresql`

`pg_hba.conf`: arquivo responsável por armazenar os usuários que possuem acesso ao servidor PostgreSQL. Este arquivo deverá ser configurado para fornecer o primeiro acesso ao Servidor de banco de dados PostgreSQL. No final do arquivo, localizar as informações abaixo e substituir md5 por trust, conforme exemplo abaixo.

````
# IPv4 local connections:
host    all              all             127.0.0.1/32             trust
# IPv6 local connections:
host    all              all             ::1/128                  trust
````

https://www.devmedia.com.br/instalacao-e-configuracao-do-servidor-postgresql-no-linux/26184

Onde 9.3 é uma versão do psotgre, pode ter vairias, entao faz em todos

`/etc/postgresql/9.3/main/pg_hba.conf`

