# Using Web Proxies

- **Burpsuite** ou **ZAP**
- Para setar proxy sem ser pelo Browser, podemos setar pro computador inteiro com o **proxychains**: **`/etc/proxychains.conf`.** Ou tambem usar **Metasploit**: **`auxiliary/scanner/http/http_put`.**
- Podemos tambem interceptar a resposta do servidor (e nao so o request), que as vezes pode nos ajudar e podemos alterar o html da pag por exemplo (usar nmap com --proxies e interceptar pelo Burp).
- **Automatic Modification →** modificacoes para todos requests e respostas em certas situacoes (no ZAP tbm tem, pesquisar).
    - Vamos em `Proxy>Options>Match and Replace` e add a modificacao, Ex: Queremos que os **requests** User-Agent tenha um valor especifico sempre → `Type`: `Request header` **/** `Match`: `^User-Agent.*$` **/**  `Replace`: `User-Agent: HackTheBox Agent 1.0`  **/** `Regex match`: `True`. E caso for para um **Response** eh a mesma coisa, Ex: todo **`type number`** queremos mudar para que seja **`type text`** → `Type`: `Response body` **/** `Match`: `type="number"` **/** `Replace`: `type="text"` **/** `Regex match`: `False`.
