1 – Primeira Etapa

Inicialmente, foi realizada a inicialização dos containers utilizando o comando:
bash

docker-compose up -d

Esse comando é responsável por criar e iniciar os containers definidos no arquivo docker-compose.yml.

2 – Segunda Etapa

Foi realizada a alteração da porta de acesso da aplicação, mudando de localhost:80 para localhost:8080. Essa alteração permite acessar a aplicação pela porta 8080.

3 – Terceira Etapa

Foi configurado o arquivo nginx.conf, adicionando pesos para cada servidor. O primeiro servidor recebeu peso 8, enquanto os demais receberam peso 1. Essa configuração faz com que o primeiro servidor receba uma quantidade maior de requisições.

4 – Quarta Etapa

Foi adicionada a configuração ip_hash no documento de configuração do Nginx. Essa configuração permite que as requisições de um mesmo endereço IP sejam direcionadas preferencialmente ao mesmo servidor.