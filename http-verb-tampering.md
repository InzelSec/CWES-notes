# HTTP Verb Tampering

- Uso de **métodos HTTP alternativos (verbs → PUT, HEAD..)** para **burlar validações**
- **Verbs Comuns → [https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Methods)**

---

- **Bypassing Basic Authentication**
    - Analisamos o site e pegamos as paginas que nao temos permissao e que pede por credenciais.
    - Pegar a URL que coletamos, rodar  **`curl -i -X OPTIONS http://IP:PORT/`**  para ver quais metodos sao aceitos e guardamos (Ex: **`GET`**, **`POST`**, **`HEAD`**). **OBS: nem sempre retornara os metodos, depende da configuracao do servidor, se nao retornar, testar cegamente pelo Burp direto.**
    - Agora interceptamos o request pelo Burp, jogamos no Repeater e clicamos com o botao direito e alteramos o metodo de **GET** para **POST** e vice-versa (para mudar os outros, usar o request do **POST** e alterar somente o nome, Ex: **HEAD**, **PUT**..) com os que salvamos que o site aceita, e ao ir trocando enviamos e vemos se conseguimos acessar diretamente.
    
    ---
    
- Podemos usar essa vulnerabilidade em conjunto com Command Injection → Ex: um site que preenchemos um campo e ele cria uma pasta/arquivo com esse nome, ele pode ter filtros para bloquear command injection, mas se mal-configurado podemos simplesmente interceptar e mudar o metodo para outro e enviar e talvez nao bloqueie.
