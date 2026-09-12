# 10. A conversa com quem paga

Os nove capítulos anteriores tratam do que construir. Este trata do problema
que vem depois, e que costuma ser o mais difícil dos dois: nada disso aparece
numa tela.

Você vai pedir três sprints para um trabalho que o cliente nunca vai ver, que
não entra em release note, e que existe para o caso de alguém perguntar uma
coisa que ninguém perguntou ainda. Quem decide o orçamento tem outras dez
demandas na mesa, todas visíveis.

Este capítulo é sobre conduzir isso sem virar o técnico que fala em desgraça.

## 1. A ordem de ataque

Quando o sistema já está em produção, tudo está errado ao mesmo tempo e a
tentação é atacar pelo que incomoda mais. A ordem que funciona é outra, e ela
vem de uma distinção simples.

Começar a fazer certo a partir de hoje é barato. Reconstruir o passado é caro,
às vezes impossível, e o preço sobe todo mês. Então a sequência é sempre a
mesma:

1. **Parar de piorar.** Ligar o registro que falta, nem que seja bruto. Uma
   tabela de eventos mal modelada capturando desde hoje vale mais do que a
   modelagem perfeita que entra em produção em abril.
2. **Conseguir responder daqui para a frente.** Fechar as perguntas do
   [mapa](00-o-mapa.md) para operações novas, aceitando que as antigas ficarão
   com resposta parcial.
3. **Recuperar do passado o que der.** Por último, e só onde tem valor real. É
   a parte cara e a que menos gente precisa de fato.

Inverter essa ordem é o erro mais comum. O time passa três meses desenhando a
trilha definitiva enquanto o sistema continua sobrescrevendo status, e no fim
tem uma modelagem bonita e mais três meses de buraco.

## 2. O que não precisa de projeto

Boa parte deste repositório não precisa virar iniciativa própria. Precisa virar
regra em código novo, e aí o custo marginal fica perto de zero.

Coisas que entram assim, sem parar nada:

- Toda tabela nova que guarda estado nasce com histórico.
- Todo endpoint novo devolve só o que a tela usa.
- Toda integração nova propaga o identificador de correlação.
- Todo script de correção nasce versionado e gera evento.
- Toda tela nova que mostra dado de cliente registra a consulta.

Em doze meses de desenvolvimento normal, isso cobre uma fatia grande do
sistema sem nenhuma linha de orçamento. É a parte que você decide sozinho,
dentro da sua alçada técnica, e não precisa apresentar a ninguém.

O que precisa mesmo de projeto é uma lista curta: trilha retroativa, ingestão
de canal externo, mascaramento de telemetria que já está ligada, e refazer
modelo de permissão quando ele nasceu errado. Quatro coisas, não quarenta.

Separar essas duas listas antes da reunião muda o tom da conversa. Deixa de ser
"precisamos parar tudo" e passa a ser "a maior parte eu absorvo no fluxo, e
preciso de decisão em quatro pontos".

## 3. Como dimensionar sem chutar

Não estime o conjunto. Ninguém consegue, e o número que você inventar vai ser
cobrado.

O que funciona é fazer uma ponta a ponta. Escolha uma operação, uma só, e
implemente a trilha completa dela, do primeiro contato ao encerramento. Meça o
tempo de verdade. Esse número vezes o número de fluxos parecidos é uma
estimativa que você consegue defender, porque tem base, e no caminho você
descobre os buracos que não apareciam no papel.

Antes disso, conte o que existe. Quantos fluxos de negócio, quantos pontos de
escrita em estado, quantas integrações externas, quantos sistemas com login
próprio. Leva um dia e é a diferença entre estimativa e chute.

E sempre apresente faixa, com o que faz cair para o mínimo e o que faz subir
para o máximo. Número único vira promessa.

## 4. Quando pedem uma data

Vão pedir uma data para "estar em conformidade", e essa data não existe, porque
conformidade não é um estado binário com linha de chegada.

A resposta que se sustenta tem três partes:

- **Data para saber.** O diagnóstico tem prazo curto e você consegue cravar.
  Uma tarde para o checklist, duas semanas para a ponta a ponta.
- **Data para parar de piorar.** Também curta, e é a que mais importa.
- **Faixa para o resto**, com as dependências explícitas.

Nunca dê data para o que depende de terceiro. A lista de prazos por categoria
vem do jurídico, a cláusula de notificação vem da negociação com o fornecedor,
e a matriz de perfil vem de um comitê. Se você assume prazo em cima disso, você
vai ser o responsável pelo atraso de uma coisa que não controla.

Vale registrar o pedido e a sua resposta por escrito, no mesmo dia. Essa
conversa costuma ser revisitada meses depois, e memória de reunião é a pior
fonte que existe.

## 5. Como falar de custo sem soar alarmista

Não fale em multa. Primeiro porque você não sabe o valor, e o jurídico sabe.
Segundo porque soa como ameaça, e ameaça vinda da engenharia é descontada na
hora.

Três coisas funcionam melhor, e todas são verificáveis:

**Quanto custou da última vez.** Se já houve um pedido parecido, some as horas
que o time gastou respondendo. É um número interno, real, e ninguém discute.

**A lista do que hoje não pode ser respondido.** Lista, não adjetivo. "Não
conseguimos dizer quem tinha acesso ao banco em março" é uma frase que produz
silêncio na sala. "Nossa governança de acesso é frágil" não produz nada.

**O que fica mais caro a cada mês.** É aqui que a última coluna da tabela do
[mapa](00-o-mapa.md) trabalha por você. Os itens marcados como não reversíveis
não são uma dívida parada, são uma dívida que cresce sozinha, e essa é a única
parte do argumento que cria urgência legítima.

Se der para escolher uma coisa só, escolha demonstrar em vez de contar. Pegue
uma operação real de seis meses atrás e tente reconstruí-la na frente das
pessoas, ao vivo, com as consultas que existem. Vinte minutos de tentativa
frustrada convencem mais do que qualquer apresentação, e é honesto, porque é
exatamente o que vai acontecer quando o pedido chegar de fora.

## 6. Quem é o dono

Se a resposta for "o time", não vai acontecer. Precisa de um nome, com tempo
alocado, e com autoridade para recusar o atalho quando a pressão de entrega
apertar. Não precisa ser cargo novo, precisa ser responsabilidade nomeada.

E quando a decisão for não fazer, o que é uma decisão legítima e às vezes
correta, ela precisa ficar registrada: o que foi apresentado, o que foi
decidido, por quem e quando. Isso não é para se proteger. É porque essa decisão
vai ser revisitada no dia em que o pedido chegar, e sem registro a conversa
daquele dia vira uma discussão sobre quem avisou o quê, que é a pior conversa
possível para se ter sob pressão.

## O teste

Este capítulo é o único cujo teste não é sobre o sistema. É sobre você.

1. Se pedirem hoje a reconstrução completa de uma operação, quantas
   pessoas-dia isso consome? Você mediu ou está supondo?
2. Quais itens não reversíveis do [mapa](00-o-mapa.md) ainda estão piorando
   todo mês?
3. Qual regra nova o time passou a seguir em código novo por causa disso, e
   desde quando?
4. Existe um nome responsável, ou está distribuído no time?
5. A última vez que a decisão foi adiar, isso ficou escrito em algum lugar?

A pergunta 3 é a que separa quem entendeu de quem concordou. Concordar é de
graça.

## Onde isso termina

Com este capítulo o repositório cobre os dois lados: o que precisa existir no
sistema e como conseguir espaço para construir. O [mapa](00-o-mapa.md) mostra o
conjunto e o que ainda dá para decidir. O [checklist](99-checklist.md) diz onde
você está hoje.

O resto é discussão. Se você passou por isso e conduziu diferente, abre uma
issue contando como foi.
