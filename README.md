# IBQUOTA 3 - CUPS BACKEND (Testado com Ubuntu 16.04.7 LTS)

## 1 - Instalar os pacotes abaixo:

    sudo apt-get install apache2 build-essential cups mysql-server php libdbd-mysql-perl libnet-ldap-perl php-ldap python-minimal
    apt upgrade
    apt update

   
## 1.1 - Instalação manual do contador de páginas pkpgcounter:

    cd /home/nome_do_usuario
    wget http://www.pykota.com/software/pkpgcounter/download/tarballs/pkpgcounter-3.50.tar.gz
    tar -zxf pkpgcounter-3.50.tar.gz
    cd pkpgcounter-3.50
    
nano setup.py -> REMOVA OU COMENTE TRY CATCH DE IMPORT DO PACKAGE 'psyco'.

    python setup.py install


## 2 - Download do template do IBQUOTA 3

    wget https://github.com/valcir/ibquota/archive/master.zip
    
   

## 3 - Configuração do CUPS

    sudo nano /etc/cups/cupsd.conf

Localizar a linha "Listen localhost:631" e alterar para:

    Listen 631

Para dar permissão de acesso via rotas ao CUPS, altere as linhas conforme abaixo:

    #Restrict access to the server...
    <Location />
      Allow all
      Order allow,deny
    </Location>

    #Restrict access to the admin pages...
    <Location /admin>
      Allow all
      Order allow,deny
    </Location>

    #Restrict access to configuration files...
    <Location /admin/conf>
      Allow all
      AuthType Default
      Require user @SYSTEM
      Order allow,deny
    </Location>

## 4 Criação do banco mysql:
 
Criar o DB:

     sudo mysql -u root -p
     
   confirmar senha: *****
   
     mysql> CREATE DATABASE ibquota3;
     mysql> exit

Importar a estrutura de tabelas através do script 'ibquota3.sql'
    
     cd /home/nome_usuario/ibquota3/sql
     sudo mysql -u root ibquota3 < ibquota3.sql -p
     
   confirmar senha: *****
   
     

Criar o usuário padrão do ibquota no banco:

    sudo mysql -u root -p
    
  confirmar senha: *****
  
    mysql> GRANT ALL ON ibquota3.* TO ibquota@localhost identified by 'ibquota';
    mysql> FLUSH PRIVILEGES;
    mysql> exit
 
O script principal (ibquota3) deverá ser copiado para dentro da basta backend, do CUPS.

     cd /home/nome_usuario/backend
     sudo cp ibquota3 /usr/lib/cups/backend/
     cd /usr/lib/cups/backend
     sudo chmod 755 ibquota3
     sudo chown root ibquota3


Agora temos que editar o backend:


    sudo nano /usr/lib/cups/ibquota3

    my $DBhost="localhost";
    my $DBlogin="ibquota";
    my $DBpassword="ibquota";
    my $DBdatabase="ibquota3";
    my $DBport=3306;
 
    cd home/nome_usuario/ibquota-master/gg
    sudo mkdir /var/www/html/gg
    sudo cp -r * /var/www/html/gg
    
Lembrar de deletar o arquivo index.html do apache


Neste momento iremos editar o arquivo com as configurações de acesso a banco

    cd /var/www/html/gg
    sudo nano includes/db.php

    define("HOST", "localhost");     // Servidor com o qual voce quer se conectar.
    define("USER", "ibquota");       // Usuário para acessar o banco de dados. 
    define("PASSWORD", "ibquota");   // Senha de acesso ao banco de dados. 
    define("DATABASE", "ibquota3");  // O nome do banco de dados.

## 5 - Reiniciar o CUPS

    sudo /etc/init.d/cups restart

## 6 - Instalando a impressora no servidor

Acesse o CUPS pela URL no padrão abaixo

http://ip_do_servidor:631

Em "Administration", selecione:

    "Show printers shared by other systems"
    "Share printers connected to this system"
    "Allow printing from the Internet"
    "Allow remote administration"
    "Allow users to cancel any job (not just their own)" 

Em "Administration", clique em "Add Printer"

   No meu caso em particular, eu adicionei da seguinte forma:
   Opção 'Other Network Printers:' -> Internet Printing Protocol (http)
   Deletar qualquer texto do campo e utilizar: 
    
    ibquota3:socket://0.0.0.0
    
   onde 0.0.0.0 = ip da impressora.

## 7 - Configuração via site do IBQUOTA 3:

- Acesse o site http://ip_do_servidor/gg 
- Login com "admin" (A senha será resetada)
- Configuração principal (Paths, Base de usuários [SQL ou LDAP/AD], DEBUG)
    
## 8 - Teste a configuração do Backend:

    perl /usr/lib/cups/backend/ibquota3 --check
    
Saída que queremos:

    PATH_PYTHON = /usr/bin/python [OK]
    Python is executable [OK]
    PATH_PKPGCOUNTER = /usr/bin/pkpgcounter [OK]
    Base de Dados: LOCAL SQL [OK]
## 9 - Como saber se funcionou?

1) Crie um grupo de nome qualquer:
Cadastro > Grupos

2) Crie uma política com seus critérios.
Avançado > Políticas de impressão

3) Dentro da política, atribua a ela um grupo.

4) Adicione o(s) nome(s) da impressora(s) dentro da política.

5) Por fim, cadastre um usuário, e atribua à ele um dos grupos.
Faça o teste da impressão. Caso na aba "Relatórios>Impressões" vier com status "OK - Impresso com sucesso", está tudo certo.

Se não, veja no log de erro dentro do IBQUOTA. 
Relatórios>Impressões com erro
# Observações finais:
1) Não testei as configurações com LDAP/AD integrados no IBQUOTA. Fiz a importação do AD e inseri manualmente no mysql.
2) Pode acontecer do 'setup.py' do pkpgcounter, jogar o arquivo 'pkpgcounter' para a pasta '/usr/local/bin/', se for este o caso, copie o arquivo para a pasta correta


        sudo cp /usr/local/bin/pkpgcounter /usr/bin/pkpgcounter
    

Obrigado pelo interesse no IBQUOTA!
