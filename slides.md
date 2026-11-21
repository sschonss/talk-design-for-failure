---
marp: true
theme: aifa
paginate: true
lang: pt-BR
title: Design for Failure
description: Como assim meu sistema falha?
author: Luiz Schons
footer: 'Design for Failure · luizschons.com'
---

<!-- _class: cover -->
<!-- _paginate: false -->
<!-- _footer: '' -->

# Design for Failure

como assim meu sistema falha?

<br>

**Luiz Schons**

<!--
Palestra baseada no artigo "Design for Failure: como assim meu sistema falha?" em luizschons.com/pt-br/posts/design-for-failure/. Apresentei versões dela na SoftWeek, no PHPeste e no TLC Share Day, da Tech Leads Club, em 2025.
A ideia é simples: dar nome aos padrões que a gente já usa, às vezes sem saber. Retry, fallback, circuit breaker e saga não são invenções recentes. Entender de onde vem cada um ajuda a saber quando usar, e principalmente quando não usar.
Todo código que eu mencionar é didático: existe para mostrar a ideia, não para ir direto para produção. Para isso existem bibliotecas testadas, que eu cito no caminho.
Transição: começar pelo gancho. A apresentação pessoal vem logo depois.
-->

---

<!-- _class: statement -->

> Todo sistema **falha**.

<p class="muted">A pergunta é o que acontece com o resto quando ele cair.</p>

<!--
Deixar a frase assentar uns segundos.
Não é se um serviço vai cair. É o que acontece com o resto quando ele cair. Essa é a conversa de hoje.
Transição: "Antes de seguir, deixa eu me apresentar."
-->

---

<div class="about">
<div>

# Luiz Schons

<p class="role">Senior Software Engineer @ <b>PicPay</b></p>

<div class="chips">
<span>Guarapuava, Paraná, Brasil</span>
<span>DevParaná</span>
<span>UTFPR</span>
<span>PHP</span>
</div>

<p class="site">luizschons.com</p>

</div>
<img class="photo" src="assets/luiz.jpg" alt="Luiz Schons">
</div>

<!--
Apresentação rápida (30 segundos): quem sou, onde trabalho, de onde venho.
Transição: "Vou dividir a conversa em quatro partes, acompanhando uma loja virtual que cresce e começa a falhar."
-->

---

## O caminho

<div class="cols4">
<div class="card"><p class="muted small">Parte 1</p><h3>Pontos de falha</h3></div>
<div class="card"><p class="muted small">Parte 2</p><h3>Retry, fallback, breaker</h3></div>
<div class="card"><p class="muted small">Parte 3</p><h3>Saga</h3></div>
<div class="card"><p class="muted small">Parte 4</p><h3>Juntando tudo</h3></div>
</div>

<!--
Quatro partes, no mesmo fio de uma loja virtual.
- Parte 1: a loja cresce, a infra se espalha, e cada seta vira um ponto de falha.
- Parte 2: o que fazer quando uma chamada falha — retry, fallback e circuit breaker.
- Parte 3: quando a compra é um conjunto de passos, entra a saga e as compensações.
- Parte 4: os padrões juntos, e por que nenhum deles é bala de prata.
Um único exemplo atravessa a palestra: e-commerce, estoque, frete e pagamento.
-->

---

<!-- _class: divider -->

<div class="kicker">Parte 1 · A loja</div>

# Muitos pontos de falha

<!--
Primeiro ato: de um servidor para um sistema distribuído, e por que isso multiplica as formas de falhar.
-->

---

## Uma loja, um servidor

<div class="cols">
<div class="card"><p class="muted small">No começo</p><h3>20%</h3><p class="small">da capacidade. Está tudo bem.</p></div>
<div class="card warn"><p class="muted small">A loja cresce</p><h3>80%</h3><p class="small">É hora de escalar.</p></div>
</div>

<p class="muted">Um servidor, uma aplicação, um banco de dados.</p>

<!--
Toda loja virtual começa do mesmo jeito: um servidor, uma aplicação e um banco. Enquanto o servidor usa 20% da capacidade, está tudo bem. Aí a loja cresce, o uso vai para 80%, e é hora de escalar.
Na versão ao vivo eu conto essa história com mais cara: o computador antigo na sala, o "super servidor" de 4 GB, a conta que chega quando a gente só compra máquina maior. Aqui vou direto ao que importa: existem duas formas de escalar.
-->

---

## Duas formas de escalar

<div class="cols">
<div class="card"><p class="muted small">Vertical</p><h3>Melhora o que você já tem</h3><p class="small">Mais CPU, mais memória. É simples, mas tem um teto e fica caro rápido.</p></div>
<div class="card accent"><p class="muted small">Horizontal</p><h3>Aumenta o número de máquinas</h3><p class="small">Mais servidores, containers, um cluster. Não tem um teto tão claro.</p></div>
</div>

<p class="muted">Agora o sistema é distribuído.</p>

<!--
Vertical: você melhora o equipamento que já tem. Mais CPU, mais memória, um servidor maior. É simples, mas tem um teto e fica caro rápido.
Horizontal: você aumenta o número de equipamentos. Mais servidores, containers, um Kubernetes orquestrando tudo. Não tem um teto tão claro, mas agora o sistema é distribuído.
E é no sistema distribuído que você começa a perceber quantos pontos de falha existem.
-->

---

## Nuvem, Docker, Kubernetes

<div class="cols3">
<div class="card"><h3>Nuvem</h3><p class="small">Infra sob demanda</p></div>
<div class="card"><h3>Docker</h3><p class="small">A aplicação encapsulada</p></div>
<div class="card"><h3>Kubernetes</h3><p class="small">Réplicas e orquestração</p></div>
</div>

<p class="muted">Mais setas. Mais pontos de falha.</p>

<!--
A loja vai para a nuvem, empacota a aplicação em containers, e o Kubernetes passa a orquestrar réplicas. Isso tira um pedaço da dor de infra: menos máquina na sala, menos herói acordado de madrugada.
Mas não estamos aqui para falar de infraestrutura. Estamos aqui para falar de como o sistema se comporta quando as coisas dão errado. Porque elas sempre dão. E quanto mais distribuído o desenho, mais setas existem entre as partes.
-->

---

## Cada seta é uma chamada de rede

<svg viewBox="0 0 1100 280" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Duas caixas ligadas por uma seta de rede que pode falhar">
  <defs>
    <marker id="arr1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="8" markerHeight="8" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#3b5bdb"/></marker>
  </defs>
  <rect x="80" y="90" width="280" height="100" rx="18" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
  <text x="220" y="150" text-anchor="middle" font-family="Inter, sans-serif" font-size="32" font-weight="700" fill="#1d1f24">Serviço A</text>
  <line x1="380" y1="140" x2="700" y2="140" stroke="#3b5bdb" stroke-width="4" marker-end="url(#arr1)"/>
  <text x="540" y="118" text-anchor="middle" font-family="Inter, sans-serif" font-size="22" font-weight="600" fill="#d6455d">pode falhar</text>
  <rect x="740" y="90" width="280" height="100" rx="18" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
  <text x="880" y="150" text-anchor="middle" font-family="Inter, sans-serif" font-size="32" font-weight="700" fill="#1d1f24">Serviço B</text>
</svg>

<p class="muted">Timeout, o outro lado caiu, um lock segurando um recurso.</p>

<!--
Uma chamada entre dois serviços pode demorar demais, o outro lado pode ter caído, pode ter um lock segurando um recurso. Cada seta num diagrama de arquitetura é uma chamada de rede, e toda chamada de rede pode falhar.
-->

---

## A jornada da compra

<svg viewBox="0 0 1100 360" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="E-commerce falando com estoque, frete e pagamento">
  <defs>
    <marker id="arr2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="8" markerHeight="8" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#3b5bdb"/></marker>
  </defs>
  <rect x="40" y="130" width="280" height="100" rx="18" fill="#3b5bdb" stroke="#3b5bdb" stroke-width="2"/>
  <text x="180" y="190" text-anchor="middle" font-family="Inter, sans-serif" font-size="32" font-weight="700" fill="#ffffff">E-commerce</text>
  <line x1="340" y1="160" x2="520" y2="70" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arr2)"/>
  <line x1="340" y1="180" x2="520" y2="180" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arr2)"/>
  <line x1="340" y1="200" x2="520" y2="290" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arr2)"/>
  <rect x="540" y="20" width="280" height="90" rx="18" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
  <text x="680" y="75" text-anchor="middle" font-family="Inter, sans-serif" font-size="30" font-weight="700" fill="#1d1f24">Estoque</text>
  <rect x="540" y="135" width="280" height="90" rx="18" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
  <text x="680" y="190" text-anchor="middle" font-family="Inter, sans-serif" font-size="30" font-weight="700" fill="#1d1f24">Frete</text>
  <rect x="540" y="250" width="280" height="90" rx="18" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
  <text x="680" y="305" text-anchor="middle" font-family="Inter, sans-serif" font-size="30" font-weight="700" fill="#1d1f24">Pagamento</text>
  <rect x="880" y="135" width="180" height="90" rx="18" fill="#e6f6ef" stroke="#e6f6ef" stroke-width="2"/>
  <text x="970" y="190" text-anchor="middle" font-family="Inter, sans-serif" font-size="28" font-weight="700" fill="#0f9d6b">200 OK</text>
  <line x1="840" y1="180" x2="868" y2="180" stroke="#0f9d6b" stroke-width="3" marker-end="url(#arr2)"/>
</svg>

<p class="muted">Se todo mundo responde, o cliente recebe um 200 OK.</p>

<!--
Para o resto da palestra, vamos usar uma loja dividida em serviços. Uma compra passa pelo e-commerce, que fala com o estoque, com o frete e com o pagamento. Se todo mundo responde, o cliente recebe um 200 OK.
Transição: "Mas e se o serviço de frete estiver fora do ar?"
-->

---

<!-- _class: statement -->

> E se o **frete** estiver fora do ar?

<!--
Deixar a pergunta com a plateia um instante.
A resposta mais natural é tentar de novo. É daí que começa a parte 2.
-->

---

<!-- _class: divider -->

<div class="kicker">Parte 2 · Uma chamada que falha</div>

# Retry, fallback, circuit breaker

<!--
Três padrões para uma única chamada: tentar de novo do jeito certo, ter um plano B, e parar de martelar um serviço que já caiu.
-->

---

## Retry: tentar de novo, do jeito certo

<div class="chips accent">
<span>Pacote perdido</span>
<span>Timeout num pico</span>
<span>Container reiniciando</span>
</div>

<p class="muted">Muitas falhas em rede são temporárias.</p>

<!--
A resposta mais natural é tentar de novo. Muitas falhas em rede são temporárias: um pacote perdido, um timeout num momento de pico, um container reiniciando. A segunda tentativa, alguns instantes depois, muitas vezes funciona.
Mas não é sair tentando em loop. Existe um jeito certo de fazer isso, e ele é mais antigo do que parece.
-->

---

## Backoff e jitter

<div class="cols">
<div class="card"><p class="muted small">Backoff exponencial</p><h3>1 s, 2 s, 4 s, 8 s</h3><p class="small">O tempo de espera aumenta a cada tentativa.</p></div>
<div class="card accent"><p class="muted small">Jitter</p><h3>4 s ± 20%</h3><p class="small">Um pouco de aleatoriedade no intervalo.</p></div>
</div>

<p class="muted">A ideia vem da Ethernet, anos 70 e 80.</p>

<!--
Essa ideia é dos anos 70 e 80, quando as primeiras redes compartilhadas precisavam lidar com colisões. A Ethernet, por exemplo, sorteia quanto esperar antes de transmitir de novo, num intervalo que dobra a cada colisão. É daí que vêm os dois ingredientes de um bom retry.
Backoff exponencial: o tempo de espera aumenta a cada tentativa (1 s, 2 s, 4 s, 8 s). Assim você não fica martelando um serviço que já está com dificuldade.
Jitter: um pouco de aleatoriedade nesse tempo, por exemplo 4 s ± 20%. Parece detalhe, mas é o que evita que todos os clientes tentem de novo no mesmo instante.
Sem jitter, mil clientes falham juntos, esperam 1 segundo e tentam de novo juntos. O serviço, que estava tentando se recuperar, recebe mil requisições no mesmo instante e cai de novo. Isso tem nome: thundering herd, a manada trovejante. Com ±20%, o pico cai; com full jitter, cai ainda mais. É o mesmo número de tentativas, só que espalhadas.
No exemplo didático, o timeout da chamada é de 2 segundos, e a gente só retenta falha de conexão e erro 5xx. Um 4xx, como 400 ou 422, significa que o pedido está errado: mandar o mesmo pedido três vezes só gera três erros.
-->

---

## Três cuidados

<ol class="steps">
<li>Só repita o que é <b>idempotente</b></li>
<li>Conte o <b>tempo total</b></li>
<li>No PHP-FPM, esperar <b>custa um processo</b></li>
</ol>

<p class="muted">Timeout 2 s + esperas 1 s e 2 s = 9 s no pior caso.</p>

<!--
Três cuidados que valem para qualquer retry.
1. Só repita o que é idempotente. Retentar uma consulta de frete é seguro. Retentar uma cobrança, sem uma chave de idempotência, pode cobrar o cliente duas vezes, porque a primeira tentativa pode ter funcionado e só a resposta se perdeu.
2. Conte o tempo total. Com timeout de 2 s e esperas de 1 s e 2 s, o pior caso são 9 segundos. Alguém está esperando do outro lado.
3. No PHP-FPM, esperar custa um processo. Aquele usleep segura um worker parado sem fazer nada. No Swoole e no Hyperf, a mesma chamada só pausa a corrotina, e o processo segue atendendo outras requisições.
O retry resolve falhas rápidas. Mas e quando o frete fica fora do ar por dez minutos?
-->

---

## Fallback: uma resposta alternativa

<div class="chips accent">
<span>Valor padrão</span>
<span>Resposta em cache</span>
<span>Valor calculado antes</span>
</div>

<p class="muted">O sistema não para inteiro. Graceful degradation.</p>

<!--
Tentar três vezes e devolver erro para o cliente significa que nenhuma compra fecha nesse período, por causa de um único serviço.
É aqui que entra o fallback, uma ideia que ganhou força nos anos 90 e 2000 junto com alta disponibilidade e graceful degradation: quando um serviço crítico falha, o sistema não para inteiro, ele devolve uma resposta alternativa. Pode ser um valor padrão, uma resposta em cache ou um valor calculado antes.
No exemplo da loja, se o frete não responde, dá para usar o último valor calculado para aquele CEP ou um frete padrão, por exemplo R$ 29,90 e 10 dias. A compra continua. O cliente talvez pague um frete um pouco diferente, mas consegue comprar.
-->

---

## Essencial × não essencial

<div class="cols">
<div class="card good"><p class="muted small">Frete</p><h3>Pode ter fallback</h3><p class="small">Último valor daquele CEP, ou um frete padrão.</p></div>
<div class="card bad"><p class="muted small">Pagamento</p><h3>Não tem “pagamento padrão”</h3><p class="small">Aqui o fallback não é devolver um valor. É parar com segurança.</p></div>
</div>

<p class="muted">Fallback é decisão de negócio, não só técnica. E precisa de alerta.</p>

<!--
Duas coisas importantes sobre fallback.
Ele é uma decisão de negócio, não só técnica. Um frete padrão pode sair mais caro para a loja do que o real, e alguém precisa concordar com isso. E nem tudo tem fallback: não existe "pagamento padrão".
Essa é a diferença entre o que é essencial e o que não é. O frete pode degradar e a compra segue. O pagamento não pode inventar uma resposta: se ele cai, a operação precisa parar com segurança.
E passar pelo fallback nunca é um bom sinal. Ele mantém o sistema de pé, mas esconde o problema. Por isso o log ali dentro não é enfeite: todo fallback precisa de métrica e alerta, senão o frete pode ficar fora do ar uma semana sem ninguém perceber.
-->

---

## Circuit breaker

<p class="muted" style="margin-bottom: 24px;">Parar de bater em cachorro morto</p>

<div class="cols3">
<div class="card"><h3>Retry</h3><p class="small">Três tentativas, espera o timeout.</p></div>
<div class="card"><h3>Fallback</h3><p class="small">O cliente é atendido.</p></div>
<div class="card bad"><h3>Por baixo</h3><p class="small">O frete continua apanhando.</p></div>
</div>

<!--
Com retry e fallback, o cliente é atendido. Mas olhe o que está acontecendo por baixo: a cada compra, o e-commerce tenta o frete três vezes, espera os timeouts, e só aí vai para o fallback. Multiplique isso por todas as compras e o serviço de frete, que caiu, continua recebendo uma enxurrada de chamadas. É bater em cachorro morto: ninguém ganha nada, e ainda atrapalha a recuperação.
O ideal seria o serviço de frete poder dizer "para de tentar aí, ainda tô me recuperando". Como ele não pode, quem chama precisa perceber sozinho. Essa é a função do circuit breaker, inspirado nos disjuntores elétricos e popularizado pelo livro Release It!, de Michael Nygard, em 2007. Quando a falha passa de um limite, o disjuntor "desarma" e corta as chamadas por um tempo.
-->

---

## Três estados

<div class="flow">
<div class="ok">CLOSED<br><span class="small">As chamadas passam</span></div>
<span>→</span>
<div class="bad">OPEN<br><span class="small">Vai direto ao fallback</span></div>
<span>→</span>
<div class="wait">HALF-OPEN<br><span class="small">Uma chamada de teste</span></div>
</div>

<p class="muted">Inspirado nos disjuntores. Release It!, 2007.</p>

<!--
CLOSED: o normal. As chamadas passam, e as falhas são contadas.
OPEN: as falhas chegaram ao limite. Nenhuma chamada sai; tudo vai direto para o fallback, sem esperar timeout. O serviço do outro lado ganha um respiro.
HALF-OPEN: passou o tempo de espera. Uma chamada de teste é liberada. Se der certo, o circuito fecha e a vida segue. Se der errado, abre de novo e a contagem recomeça.
No exemplo enxuto: 5 falhas seguidas abrem o circuito, e o cooldown é de 60 segundos até liberar um teste.
Com o circuito aberto, a compra não espera mais nove segundos para cair no fallback: ela vai direto. O cliente é atendido mais rápido e o frete para de apanhar.
-->

---

## Onde o estado mora?

<div class="cols">
<div class="card warn"><p class="muted small">PHP-FPM</p><h3>Redis ou APCu</h3><p class="small">Cada requisição começa do zero. Em memória, o breaker nunca abre.</p></div>
<div class="card good"><p class="muted small">Swoole e Hyperf</p><h3>Memória por worker</h3><p class="small">O processo continua vivo. E já existem componentes prontos.</p></div>
</div>

<p class="muted">Um circuito abrindo precisa aparecer no monitoramento.</p>

<!--
Um detalhe que pega muita gente no PHP: onde esse estado mora? No PHP-FPM, cada requisição começa do zero, então um objeto CircuitBreaker em memória esquece tudo ao final de cada requisição e nunca abre. O estado precisa ficar num lugar compartilhado, como Redis ou APCu. No Swoole e no Hyperf, o processo continua vivo entre requisições, e o estado em memória funciona por worker. O Hyperf, inclusive, já tem componentes prontos para isso: o hyperf/retry e o hyperf/circuit-breaker.
E, como no fallback, um circuito abrindo é um evento que alguém precisa ver. Ele deve gerar alerta e aparecer no seu monitoramento.
Até aqui, cada padrão lidou com uma chamada que falha. Mas uma compra é um conjunto de passos.
-->

---

<!-- _class: divider -->

<div class="kicker">Parte 3 · A compra inteira</div>

# Saga

<!--
Terceiro ato: consistência quando não existe uma transação que abrace todos os serviços.
-->

---

## A compra é um conjunto de passos

<div class="flow">
<div>Reservar o estoque</div>
<span>→</span>
<div>Cobrar o pagamento</div>
<span>→</span>
<div class="bad">Agendar a entrega</div>
</div>

<p class="muted">E se a entrega falhar depois da cobrança?</p>

<!--
Reservar o estoque, cobrar o pagamento, agendar a entrega. E se o estoque foi reservado e o pagamento cobrado, mas a entrega falhou?
Num banco de dados só, isso seria uma transação: ou tudo acontece, ou nada acontece. Mas cada serviço tem o seu banco, e não existe uma transação que abrace todos eles.
A resposta para isso é a saga, descrita por Hector Garcia-Molina e Kenneth Salem em 1987: uma sequência de transações locais em que cada ação tem a sua compensação correspondente.
-->

---

## Cada ação tem uma compensação

<svg viewBox="0 0 1100 300" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Passos da saga e suas compensações">
  <defs>
    <marker id="arr3" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="8" markerHeight="8" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#3b5bdb"/></marker>
    <marker id="arr4" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="8" markerHeight="8" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#d6455d"/></marker>
  </defs>
  <rect x="40" y="30" width="280" height="80" rx="16" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
  <text x="180" y="80" text-anchor="middle" font-family="Inter, sans-serif" font-size="28" font-weight="700" fill="#1d1f24">Reservar estoque</text>
  <line x1="340" y1="70" x2="420" y2="70" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arr3)"/>
  <rect x="440" y="30" width="280" height="80" rx="16" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
  <text x="580" y="80" text-anchor="middle" font-family="Inter, sans-serif" font-size="28" font-weight="700" fill="#1d1f24">Cobrar</text>
  <line x1="740" y1="70" x2="820" y2="70" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arr3)"/>
  <rect x="840" y="30" width="220" height="80" rx="16" fill="#fdecef" stroke="#fdecef" stroke-width="2"/>
  <text x="950" y="80" text-anchor="middle" font-family="Inter, sans-serif" font-size="28" font-weight="700" fill="#d6455d">Entrega falhou</text>
  <rect x="40" y="190" width="280" height="80" rx="16" fill="#fff4e0" stroke="#fff4e0" stroke-width="2"/>
  <text x="180" y="240" text-anchor="middle" font-family="Inter, sans-serif" font-size="28" font-weight="700" fill="#c27803">Liberar reserva</text>
  <line x1="420" y1="230" x2="340" y2="230" stroke="#d6455d" stroke-width="3" marker-end="url(#arr4)"/>
  <rect x="440" y="190" width="280" height="80" rx="16" fill="#fff4e0" stroke="#fff4e0" stroke-width="2"/>
  <text x="580" y="240" text-anchor="middle" font-family="Inter, sans-serif" font-size="28" font-weight="700" fill="#c27803">Estornar</text>
  <line x1="820" y1="230" x2="740" y2="230" stroke="#d6455d" stroke-width="3" marker-end="url(#arr4)"/>
  <text x="950" y="240" text-anchor="middle" font-family="Inter, sans-serif" font-size="19" font-weight="600" fill="#d6455d">compensa ao contrário</text>
</svg>

<p class="muted">Para o usuário parece atômico. A consistência é eventual.</p>

<!--
Reservou estoque? A compensação é liberar a reserva. Cobrou? A compensação é estornar.
Se a entrega falha, o pagamento é estornado e depois o estoque é liberado.
Para o usuário, a operação parece atômica, mas tecnicamente a consistência é eventual. O sistema garante o "tudo ou nada" não por uma transação de verdade, mas pela capacidade de desfazer cada passo se o processo falhar.
Existem duas formas principais de coordenar uma saga.
-->

---

## Duas formas de coordenar

<div class="cols">
<div class="card"><p class="muted small">Coreografada</p><h3>Ninguém coordena no centro</h3><p class="small">Cada serviço reage a eventos e desfaz a própria parte.</p></div>
<div class="card accent"><p class="muted small">Orquestrada</p><h3>Um orquestrador conhece o fluxo</h3><p class="small">Chama na ordem e dispara as compensações na ordem inversa.</p></div>
</div>

<p class="muted">Menos acoplamento de um lado. Um lugar só para ler, do outro.</p>

<!--
Na coreografada, ninguém coordena: cada serviço reage a eventos e publica os seus. Quando algo falha, quem percebe publica um evento, e cada serviço desfaz a própria parte. Na palestra usei o event dispatcher do Hyperf para o exemplo ficar curto. Entre serviços diferentes, esse evento viaja por um broker como Kafka ou RabbitMQ, e o listener vira um consumidor. O ponto forte é o desacoplamento: não existe um serviço central que, se cair, para tudo. O ponto fraco é que o fluxo não está escrito em lugar nenhum. Ele existe espalhado pelos listeners, e entender "o que acontece quando um pedido é cancelado" exige olhar vários serviços.
Na orquestrada, um orquestrador conhece o fluxo inteiro: chama cada serviço na ordem e, se algum falhar, dispara as compensações dos passos que já aconteceram, na ordem inversa. O fluxo está num lugar só, fácil de ler e de testar. Em troca, o orquestrador vira uma peça central: se ele cai, nenhuma saga anda.
Num sistema real, falta uma coisa nesse desenho: guardar o estado da saga. Se o orquestrador cair no meio de uma compensação, ele precisa saber, ao voltar, onde parou. E uma compensação também pode falhar, então ela precisa de retry e precisa ser idempotente — o mesmo evento pode chegar duas vezes. É por isso que, em produção, sagas costumam rodar em cima de ferramentas como Temporal ou de uma tabela de estado bem cuidada.
-->

---

## Três tipos de saga

<div class="cols3">
<div class="card"><p class="muted small">Epic</p><h3>HTTP, atômica, orquestrada</h3><p class="small">O orquestrador chama e espera. Nenhum serviço fala com outro.</p></div>
<div class="card"><p class="muted small">Parallel</p><h3>Kafka, eventual, orquestrada</h3><p class="small">Continua com orquestrador, mas os passos podem correr em paralelo.</p></div>
<div class="card"><p class="muted small">Anthology</p><h3>Kafka, eventual, coreografada</h3><p class="small">Sem orquestrador. Cada serviço trata o próprio rollback.</p></div>
</div>

<p class="muted">Quanto mais à direita, menos acoplamento — e mais difícil de acompanhar.</p>

<!--
O livro Software Architecture: The Hard Parts, de Ford, Richards, Sadalage e Dehghani, organiza as sagas por três escolhas: comunicação síncrona ou assíncrona, consistência atômica ou eventual, e coordenação orquestrada ou coreografada. Na palestra mostrei três das combinações.
A Epic Saga é a mais parecida com o código orquestrado: o orquestrador chama cada serviço por HTTP, espera a resposta, e nenhum serviço fala com outro diretamente. A Parallel Saga mantém o orquestrador, mas conversa por mensagens, o que permite executar passos em paralelo. A Anthology Saga é a coreografada pura: sem orquestrador, cada serviço mantém a sua transação e trata o próprio rollback. Quanto mais para baixo nessa lista, menos acoplamento e mais escala, e mais difícil entender o que está acontecendo.
-->

---

<!-- _class: divider -->

<div class="kicker">Parte 4 · Fechamento</div>

# Juntando tudo

<!--
Último ato: os padrões não aparecem sozinhos, e cada um cobra um preço.
-->

---

## Na arquitetura real, eles aparecem juntos

<div class="cols">
<div class="card accent"><p class="muted small">A chamada ao frete</p><h3>Retry, breaker, fallback e log</h3><p class="small">Backoff e jitter, um disjuntor em volta, o último valor calculado, e um log de cada tentativa.</p></div>
<div class="card"><p class="muted small">A compra inteira</p><h3>Uma saga</h3><p class="small">Desfaz os passos se algo der errado no meio.</p></div>
</div>

<p class="muted">Agora os seus sistemas podem falhar tranquilamente.</p>

<!--
Numa arquitetura real, esses padrões aparecem juntos. A chamada ao frete tem retry com backoff e jitter, um circuit breaker em volta, um fallback com o último valor calculado e um log de cada tentativa e falha. A compra inteira é uma saga, que desfaz os passos se algo der errado no meio.
"Agora os seus sistemas podem falhar tranquilamente."
Deixar um segundo. Depois virar a frase.
-->

---

<!-- _class: statement -->

> Ou talvez **não**.

<!--
Cada padrão desses resolve um problema e cria outro.
-->

---

## Cada padrão cobra um preço

<div class="cols4">
<div class="card"><p class="muted small">Retry</p><h3>Aumenta a carga</h3><p class="small">Num serviço que já está mal.</p></div>
<div class="card"><p class="muted small">Fallback</p><h3>Esconde a falha</h3><p class="small">E pode custar dinheiro.</p></div>
<div class="card"><p class="muted small">Circuit breaker</p><h3>Precisa de estado</h3><p class="small">E de limites bem calibrados.</p></div>
<div class="card"><p class="muted small">Saga</p><h3>Mais código</h3><p class="small">Consistência eventual e compensações.</p></div>
</div>

<p class="muted">Tudo é trade-off.</p>

<!--
O retry aumenta a carga no serviço que já está mal. O fallback esconde a falha e pode custar dinheiro. O circuit breaker precisa de estado compartilhado e de limites bem calibrados, senão abre à toa ou não abre nunca. A saga troca uma transação simples por consistência eventual, compensações e muito mais código. Tudo é trade-off.
-->

---

<!-- _class: statement -->

> Não existe **bala de prata**.

<p class="muted">Identifique o que faz sentido no seu contexto.</p>

<!--
Então não se apeguem a padrões. Identifique o que faz mais sentido para o seu contexto: qual falha é provável, quanto ela custa, quanto tempo alguém pode esperar, e qual resposta errada é aceitável. Não existe bala de prata.
Transição: o artigo tem o código didático e os detalhes que não cabem no palco.
-->

---

<div class="qr-slide">
<div>

<p class="kicker-dark">A versão escrita desta palestra</p>

# Design for Failure

<p class="muted">luizschons.com/pt-br/posts/design-for-failure</p>

</div>
<figure class="qr">
<img src="assets/qr-artigo-pt.svg" alt="QR code do artigo Design for Failure">
<figcaption>Escaneie para ler o artigo</figcaption>
</figure>
</div>

<!--
Para se aprofundar: o artigo em luizschons.com/pt-br/posts/design-for-failure/ é a versão escrita desta palestra, com mais espaço para os detalhes. Deixar alguns segundos para quem quiser escanear.
Depois, ir para as perguntas.
-->

---

<!-- _paginate: false -->
<!-- _footer: '' -->

# Perguntas?

<div class="contacts">
<div><span class="contact-label">Site</span><span class="contact-value">luizschons.com</span></div>
<div><span class="contact-label">LinkedIn</span><span class="contact-value">linkedin.com/in/luiz-schons</span></div>
<div><span class="contact-label">GitHub</span><span class="contact-value">github.com/sschonss</span></div>
</div>

<!--
Abrir para perguntas. Deixar este slide na tela durante o Q&A.
-->
