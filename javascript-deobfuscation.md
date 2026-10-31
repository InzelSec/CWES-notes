# JavaScript Deobfuscation

Quando encontramos códigos fonte Javascript obfuscado (desenvolvedores fazem isso por motivos de segurança, já que o Javascript fica no client-side) e precisamos realizar a deobfusação para conseguir entende-lo, ex de código obfuscado:
`eval(function (p, a, c, k, e, d) { e = function (c) { '...SNIP... |true|function'.split('|'), 0, {}))`

Da mesma forma que para obfuscar utilizam ferramentas, na deobfuscação também.

---

Primeiro pegamos o código-fonte ao Inspecionar a página.

Caso o código esteja inteiro em uma só linha, significa que ele foi minimizado, então usamos o **Beautify**  →  [https://beautifier.io/](https://beautifier.io/)

Para fazer a deobfuscação, usamos o **UnPacker**  →  [https://matthewfl.com/unPacker.html](https://matthewfl.com/unPacker.html)
