# CSRF (Cross-Site Request Forgery)

- Como é uma proteção de token para ataques, vamos procurar em lugares como settings → mudar email, ou algo assim..
- Basicamente precisamos interceptar e ver o request ao tentar mudar o email de nossa conta por exemplo. Ver se existe algum parametro como `csrf_protection` por exemplo (se nao tiver prov eh vuln). Começamos tentando simplesmente retira-lo, se não permitir, podemos trocar o request method para GET e retira-lo novamente, se não permitir, podemos tentar decodar o token (e decodar dnv qlqr coisa, pode ser que resulte em algo como `user_id=1`, ou seja, IDOR).
- SE CONFIRMARMOS, podemos procurar por POC generator e inserir o request para gerar o HTML do POC.

---

### **THM:**

- Em um formulário de Login por ex, caso o request tenha um `cookie Session` e um `loginToken`, ao dar um f5 notamos que os valores mudam, isso significa que o site está sempre atualizando esses valores a cada sessão para controlar o acesso e evitar ataques. Logo podemos usar o Burp Macros → que definimos para sempre manter esses mesmo `cookie` e `loginToken` quando estivermos fazendo um brute-forcing por exemplo, dando um Bypass nisso.
- Nesse caso, também podemos usar o Sequencer do Burp para checar a aleatoriedade da criação do `loginToken`
- Configurar Macro:
    - [https://tryhackme.com/r/room/burpsuiteintruder](https://tryhackme.com/r/room/burpsuiteintruder)  →  Extra Mile Challenge
