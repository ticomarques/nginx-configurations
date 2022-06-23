## Como compilar o nginx via source code, e modulos.

1 - Baixar codigo fonte do site nginx.org

2 - Instalar compilador c na sua maquina

3 - Rodar o compilador

4 - Executar make

5 - Depois install

6 - Vai dar um erro de modulos, porque eles não sao compilados porque sao de terceiros

7 - Roda um apt install nesses modulos que apareceram no erro anterior.

8 - Rodar novamente a instalacao que vai funcionar corretamente.

9 - tem que criar um arquivo no systemd, o template consta no site do nginx. 

10 - Depois tem que dar enable no nginx dentro do systemctl


