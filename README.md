## NGINX 

# This repo is a documentation of the most common topics of Nginx features, and configurations.


# Folders with examples

- Basic structure 
- Dynamic (php-fpm)
- Performance optimization
- Proxy reverse

# Linux useful comands

- Listar processos filtrados

   ps -aux | grep nginx
   ps -ef | grep nginx


- Achar recorrencias dentro do linux, como caminho do php-fpm.sock. 

   find / -name "*fpm.sock"

- Monitorar os logs na tela, tempo real.
 
   tail -f /var/log/nginx/*


#Pasta onde ficam os logs

/var/nginx/log/

Os logs podem ser customizados para ter um output diferente na tela, basta configurar no nginx.conf (global), ou no virtual block (sites-available).


# Evite dar restart antes de testar a configuracao

Depois te uma alteração no conf, primeiro rode o teste, depois um reload. Assim você garante que nao irá parar o servidor por erro no codigo.

   nginx -t

   nginx -s reload

   systemctl reload nginx

   service nginx restart


# Location 

Como funcionam os tipos de location


location optional_modifier location_match {
}

location /contato {
	*vai pegar tudo dentro de contato, inclusive as subpastas
}

location = /contato {
	*so vai pegar a pasta contato, e nao suas subpastas
}

location ~/contato {
	regex case sensetive
}

location ~* /contato {
	regex case insensetive
}

Ordem de preferência:

1- exact match = URI
2- preferential prefix match ˆ URI
3- REGEX ˜* URI



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

*coloque o rewrite acima do location com return
rewrite ˆ/guest/\w+ /welcome

*agora tudo que tiver /guest vai bater em /welcome

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

   ulimit -n   -> Exibe numero de operações 

mostra quantos operacoes I/O nos podemos operar simultâneas no cpu, fica atrelado ao numero de conexoes que podemos abrir, enquanto os workers sao processos que podemos executar.

é interessante deixar em auto, assim o hardware vai controlar o desempenho de forma natural.

   ps -aux –forest | grep nginx


   lscpu -> checa dados do cpu

   nproc -> faz a mesma coisa


Se você tem apenas um CPU, e configura 4 workers, cada worker vai trabalhar com 25% de limite.





#REVERSE PROXY

é um proxy na frente do servidor que vai filtrar as requisições, podendo distribuir em varios computadores gerando mais agilidade, podendo proteger de ataques DDOS, fazer cache e direcionar o request para esse cache, podendo direcionar as requisições, são vários recursos que um proxy reverso da.

pode criar uma maquina para SSL encription e decription, e usar o proxy reverso para jogar para maquina que vai fazer essa encriptacao, para depois jogar na maquina da aplicacao.


como fica um proxy reverso que joga para 2 maquinas

http {


include /etc/nginx/mime.types;
access_log /var/log/nginx/access.log;
error /var/log/nginx/error.log;

server {
	listen 80;
	server_name 200.155.123.80;

	location / {
		proxy_pass http://10.10.15.2;
}
}

server {
	listen 80;
	server_name 200.155.123.80;

	location /php {
		proxy_pass http://155.200.55.65;
}
}

}



Pode ser para mesma máquina, mudando apenas a porta de listen

http {


include /etc/nginx/mime.types;
access_log /var/log/nginx/access.log;
error /var/log/nginx/error.log;

server {
	listen 80;
	server_name 200.155.123.80;

	location / {
		proxy_pass http://10.10.15.2;
}
}

server {
	listen 8080;
	server_name 200.155.123.80;

	location / {
		proxy_pass http://155.200.55.65;
}
}

}
