## NGINX

# This repo is a documentation of the most common topics of Nginx features, and configurations.

# Folders with examples

- Basic structure
- Dynamic (php-fpm)
- Performance optimization
- Proxy reverse

# Linux useful comands

- Listar processos filtrados

  > ps -aux | grep nginx

  > ps -ef | grep nginx

- Achar recorrencias dentro do linux, como caminho do php-fpm.sock.

  > find / -name "\*fpm.sock"

- Monitorar os logs na tela, tempo real.

  > tail -f /var/log/nginx/\*

  /var/nginx/log/

Os logs podem ser customizados para ter um output diferente na tela, basta configurar no nginx.conf (global), ou no virtual block (sites-available).

# Evite dar restart antes de testar a configuracao

Depois te uma alteração no conf, primeiro rode o teste, depois um reload. Assim você garante que nao irá parar o servidor por erro no codigo.

> nginx -t

> nginx -s reload

> systemctl reload nginx

> service nginx restart

# Location

Como funcionam os tipos de location

location optional_modifier location_match {
}

location /contato {
\*vai pegar tudo dentro de contato, inclusive as subpastas
}

location = /contato {
\*so vai pegar a pasta contato, e nao suas subpastas
}

location ~/contato {
regex case sensetive
}

location ~\* /contato {
regex case insensetive
}

Ordem de preferência:

1- exact match = URI
2- preferential prefix match ˆ URI
3- REGEX ˜\* URI

# RETURN

return é um recurso mais simples, caso o site tenha mudado de endereço dá para retornar o novo endereço com o resto da url. e tem que ficar dentro do server {} ou location {}

www.antigo.com/contato -> www.novo.com/contato

return 301 $scheme://www.novo.com/$request_uri

# REWRITE

é o modo mais completo, permite manipular a url, inclusive com regex.

sintax: rewrite regex URL [flags]

você cria um return como:

location /welcome {
return 301 /images/background.jpg;
}

depois você cria um rewrite:

\*coloque o rewrite acima do location com return
rewrite ˆ/guest/\w+ /welcome

\*agora tudo que tiver /guest vai bater em /welcome

rewrite ˆ/user/(\w+) /welcome/$1;

location = /welcome/tiago {
return 200 'welcome tiago'
}

muito útil para redirecionar 404, 500

# try_files

serve para checar se o arquivo, ou pasta existe.

# Performance

Worker process e worker connections.

worker processes são processos criados pelo master process, nos quais usando I/O ele controla conexões e processamento.

worker connection são conexões controladas pelos worker processes.

    Exemplo:  master process -> worker processes -> worker connections

cada worker process por default suporta 768 conexoes, sendo que o browser abre pelo menos 2 conexoes, entao esse numero cai pelo menos pela metade.

> ulimit -n -> Exibe numero de operações

mostra quantos operacoes I/O nos podemos operar simultâneas no cpu, fica atrelado ao numero de conexoes que podemos abrir, enquanto os workers sao processos que podemos executar.

é interessante deixar em auto, assim o hardware vai controlar o desempenho de forma natural.

> ps -aux –forest | grep nginx

> lscpu -> checa dados do cpu

> nproc -> faz a mesma coisa

Se você tem apenas um CPU, e configura 4 workers, cada worker vai trabalhar com 25% de limite.

# BUFFER

Espaço de memoria volatil, que se for pequeno obriga o nginx a gravar os dados no disco que gera mais I/O, deixando tudo mais lento.

client_body_buffer_size - usually form posts sent to the server

client_header_buffer_size - é uma meta informacao do body

client_max_body_size - é o corpo da postagem, que se for muito pesado dá erro 413 erro - rquest entity too large

large_client_header_buffer - meta informacao muito pesada

timeout - body ou header, error 408 nao foi capaz de processar o pedido no tempo definido

keepalive_timeout - tempo que deixa a conexao aberta com o cliente.

send_timeout - se o cliente nao responder no tempo definido no servidor, a conexao é fechada

gzip_compression - comprime o dado a ser enviado para ficar mais leve e transmitir mais rapido, sendo que quanto maior a compressao, maior é o uso do cpu para comprimir no servidor, e descomprimir no cliente.

# REVERSE PROXY

É um proxy na frente do servidor que vai filtrar as requisições, podendo distribuir em vários computadores gerando mais agilidade, podendo proteger de ataques DDOS, fazer cache e direcionar o request para esse cache, podendo direcionar as requisições, são vários recursos que um proxy reverso da.

pode criar uma maquina para SSL encription e decription, e usar o proxy reverso para jogar para maquina que vai fazer essa encriptacao, para depois jogar na maquina da aplicacao.

como fica um proxy reverso que joga para 2 maquinas

Conferir na pasta: /reverse-proxy/nginx-2-different-machines.conf

Pode ser para mesma máquina, mudando apenas a porta de listen

conferir na pasta: /reverse-proxy/nginx-same-machine.conf

# X-Real-ip

Quando um proxy reverso direciona para outras maquinas, normalmente as maquinas pegam o ip do proxy, e não do cliente que executou a requisição. É possível pegar o ip do cliente que solicitou a requisição, e se necessário pode armazenar ambos ips, do proxy e do cliente.

- Verificar se o modulo está rodando, as instalações feitas de forma padrão já trazem o módulo ativado.

Verificar se o modulo realip está ativo.

> nginx -V

Gerar log customizado para os ips reais.

log_format specialLog '$http_x_real_ip - $remote_user [$time_local] '
'"$request" $status $body_bytes_sent '
                          '"$http_referer" "$http_user_agent" $remote_addr';

access_log /var/log/nginx/access-special.log specialLog;

# NGINX Cache

Cliente cache é diferente de server cache, o servidor não tem controle de como o cliente vai lidar com cache, no maximo podemos passar algumas informações para orientar o client via headers.

> Cache-Control Header - é o responsavel por ligar e desligar o caching no browser.

Sem ele, o browser vai ficar resolicitando os arquivos sempre.

> Cache-Control: public - deixa inclusive o proxy cachear os dados.

> Cache-Control: bypasses o proxy e passa direto para o client.

> Cache-Control: public, max-age= 3660000 - controla quanto tempo é valido o cache

location = /bundle/main.js {
add_header Cache-Control public;
add_header Pragma public;
add_Header Vary Accept-Encoding;
return 200 'Public cache';
expires 2M;
}

Vamos gerar uma regra para varios arquivos

location ˜\* \.(css|js|jpg|png){
}

# NGINX gzip header compression

Comprimir os dados enviados é util para otimizar o download e renderizacao do site no cliente.

Adidionar essas regras dentro da directive http {}

> gzip on;
> gzip_comp_level 4;
> gzip_files text/plain

             text/css
             application/json
             application/x-javascript
             application/javascript
             text/xml
             application/xml
             application/xml+rss
             text/javascript;

# NGINX Micro caching

Micro caching é uma tecnica util para cachear paginas dinamicas por segundos, ou poucos minutos. O objetivo é otimizar usando caches para dados dinamicos, gerando menos processamento para o servidor já que o dado é o mesmo, não foi alterado. o cache é salvo em disco por pequeno espaco de tempo.

FastCGI protocolo de interface interativa como PHP com servidores, o NGINX é um exemplo. O FCGI gerencia gerencia multiplas requisições em apenas um processo.

> fastcgi_cache_path - usado dentro do http{} no arquivo de configuracao do nginx.
> Parametros para diretiva: local onde salvar o cache, levels - level de hierarquia do cache, keys_zone - , incative - tempo de inatividade para deletar o cache, max_size - tamanho maximo do cache.

> fastcgi_cache_key - usado para definir uma key identificadora para o cache. A key de request tem que ser a mesma key de response, é quase um token para garantir autenticidade dos dados (md5).
> Parametros: $scheme - request scheme, HTTP ou HTTPS, $request_method - usually GET, or POST, $HOST hostname ou Header enviado, $request_uri - original URL solicitada.

http {

fastcgi_cache_path /tmp/cache_nginx levels=1:2 keys_zone=ZONE_1 inactive=60m;
fastcgi_cache_key "$scheme$request_method$host$request_uri";
add_header X-Cache $upstream_cache_status;

location ˜\.php$ {
include fastcgi.conf;
fastcgi_pass unix:/run/php/php8.1-fpm.sock;

fastcgi_cache ZONE_1;
fastcgi_cache_valid 200 60m;
}

}

# Benchmark tests

Ferramenta usada é o apache benchmark (ab), como instalar e usar abaixo:

1 - instalar no ubuntu ou debian based:
apt-get -y install apache2-utils

2 - Check instalation
ab

3 - Para rodar um teste, rode no terminal:
ab -n 200 -c 20 <DEFINE YOUR URL>

# Enable SSL on NGINX

https://marketplace.digitalocean.com/apps/lemp?utm_medium=marketplace&utm_source=marketplace&utm_campaign=marketplace&utm_content=no_image#getting-started

# Prevent DDOS or Limit of service

1 - instalar siege

> apt install siege

2 - Rodar comando para testar estresse

> siege -V -r 2 -c 20 https://www.url.com

3 - Adicionar no conf:

> limit_req_zone $request_uri zone=test_zone:10m rate=1r/s

> location / {
> limit_req zone=test_zone burst=5 nodelay;
> }

4 - nginx -t

5 - nginx -s reload

6 - recomendado achar um numero de equilibrio entre connections e requests, para colocar na configuracao.

# Load Balance

# Cache system

# NGINX Acess Control
