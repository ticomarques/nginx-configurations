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

