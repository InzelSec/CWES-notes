# IDOR

- Vulnerabilidade de controle de acesso.

---

- Sempre criar duas contas para procurar por IDOR, para entender como funciona essa identificação de usuário. Ver se o site nao esta apenas codificando os IDs. Ex: **`?file=ZmlsZV8xMjMucGRm`** → decodifica = **`file_123.pdf`**. Criar varias contas e comparar a criacao de IDs e etc.
- <u>Basicamente:</u> quando o site passa informação de forma insegura (podemos inspecionar pelo Burp o request), criando vulnerabilidade para manipularmos.
    - estamos logados em um site e clicamos em ver conta e os dados, de alguma forma o site está passando o ID da nossa conta para saber quem somos e passar os dados, então podemos ver primeiramente se está sendo transmitido pela URL (improvável), caso não, podemos abrir inspecionar/network e dar f5, e ver se aparece algum formato JSON que transmite, com isso podemos manipular o ID.
    - No caso do exemplo do site, dentro de um produto ao fazer um comentário de avaliação, enviamos um comentário e vamos no burpsuite para ver o request, e o nome do autor (nesse caso a gnt) aparece dentro do request, ou seja, podemos interceptar esse envio e alterar o autor para outro nome (email de um admin por ex).
    - Ao clicar em editar nosso comentário, no request é especificado nosso ID e a mensagem, porém nosso ID está criptografado, então não conseguimos de primeira mudar o request para o ID de um admin e alterar a mensagem dele, porém podemos dar o Bypass ao dar um like no comentário do admin, pois dessa forma veremos qual o ID criptografado do comentário dele que está sendo passado.
    - Se estiver criptografado, podemos criar duas contas diferentes, e ver qual o padrão que ele encriptografa para tentar entender o padrão.
- Montar scripts em bash (Ex: notamos um IDOR que ao mudar o id conseguimos acesso a pdf de outros usuarios, montamos entao um script que faz requisicoes mudando o id e baixando os pdfs..)
- **Ex de script bash para curl com Fuzzing de ID:**
    
    ```bash
    #!/bin/bash
    
    for i in {10..30}; do
            curl -X POST -d http://83.136.253.105:53623/profile/api.php/profile/$i | grep "role"
        done
    done
    
    ```
    

---

- **Bypassing Encoded References**
    - Quando o ID eh encodado, caso nao seja em base64 ou algo que conseguimos descriptografar, podemos procurar no codigo-fonte ou no JavaScript por pistas sobre a funcao de encodar (procurar por palavras: **`MD5`**..).
- **IDOR in Insecure APIs:**
    - Situacao em que ao interceptar quando vamos editar o perfil, percebemos que as informacoes estao sendo enviadas para uma API, simplesmente TROCAMOS de PUT para GET e alteramos o ID, e a API estara nos enviando as infos desse ID.
