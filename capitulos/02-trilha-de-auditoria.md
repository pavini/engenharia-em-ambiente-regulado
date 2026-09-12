# 2. Trilha de auditoria de verdade

Quase todo sistema tem alguma coisa que o time chama de trilha de auditoria.
Normalmente é uma tabela `log` ou `historico`, preenchida por um interceptador,
com `usuario`, `acao`, `data` e um JSON solto. Ela existe, tem milhões de linhas,
e não responde a pergunta.

Não responde porque foi construída para depurar, não para provar.

## A diferença entre log e trilha

Log é para você. Serve para entender o que quebrou, é verboso, é descartável,
e ninguém se importa se o formato mudou entre versões.

Trilha é para outra pessoa, meses depois, que não conhece o sistema e precisa
reconstruir uma sequência de fatos e confiar nela. Isso muda quatro coisas.

## 1. Correlação: um identificador que atravessa tudo

A operação começa num lugar e termina em outro. Passa por três serviços, dois
provedores externos, uma fila e um job noturno. Se cada parte registra com o
identificador que ela conhece, ninguém junta depois.

Gere um identificador de operação no ponto de entrada, propague em tudo, e
registre em toda escrita de trilha. Não o `request_id` do HTTP, que morre no
fim da requisição. Um identificador do processo de negócio, que nasce quando a
proposta nasce e vive até a operação ser encerrada ou recusada.

Isso parece óbvio e é a coisa que mais falta. Sem ela, responder uma única
pergunta vira um trabalho de dias cruzando tabela na unha.

## 2. Imutabilidade: escreve e não mexe mais

Trilha é append-only. Nada de `UPDATE`, nada de `DELETE`, nada de corrigir
registro errado. Se registrou errado, você registra um novo evento dizendo que
o anterior estava errado, com o motivo e o autor.

Na prática:

- Sem `ON UPDATE CASCADE`, sem trigger que reescreve.
- A aplicação escreve com um usuário de banco que só tem `INSERT` e `SELECT`
  naquelas tabelas. Isso é mais importante do que parece: quando o auditor
  perguntar como você garante que ninguém alterou, a resposta ser "a aplicação
  não tem permissão" vale muito mais do que "a gente não faz isso".
- Se o seu banco suporta, particionamento por período e a partição antiga em
  tablespace somente leitura.

Se você precisa de garantia mais forte, encadeamento por hash resolve: cada
registro guarda o hash do anterior. Não é blockchain nem precisa ser. É uma
coluna a mais que permite provar que a sequência não foi editada no meio.
Custa pouco e a pergunta "como você prova que não mexeram" passa a ter resposta
técnica em vez de resposta de confiança.

## 3. Conteúdo: o que cada registro precisa ter

O mínimo que sobrevive a uma pergunta:

- **Quando** — timestamp com fuso explícito. Não `datetime` do servidor sem
  informação de fuso. Você vai ter servidor em região diferente, vai ter
  horário de verão em série histórica antiga, e vai ter discussão sobre se o
  evento foi antes ou depois de outro. Guarde em UTC com o deslocamento, ou
  guarde `timestamptz`.
- **Quem** — o identificador do usuário, não o nome. Nome muda. E quando a ação
  foi do sistema, diga qual sistema e disparado por quê. "usuário: sistema" não
  é resposta.
- **Em nome de quem** — se existe atuação por delegação, operador agindo por um
  cliente, suporte acessando conta alheia, isso precisa estar separado do "quem".
  É a primeira coisa que auditoria procura.
- **O quê** — a ação, em vocabulário de negócio e estável ao longo do tempo.
  `proposta.aprovada`, não `PUT /v2/proposals`. A rota vai mudar, o fato não.
- **O antes e o depois** — para mudança de estado, o valor anterior e o novo.
  Sem isso você sabe que mudou, não o que mudou.
- **De onde** — IP, canal, aplicação. Ajuda mais do que parece quando a pergunta
  é sobre acesso indevido.
- **Por quê** — quando existe justificativa, e em ação sensível deveria existir,
  guarde o texto que a pessoa escreveu.

## 4. Canal: o buraco que quase todo mundo tem

Este é o ponto que mais vi quebrar na prática.

A operação começa num canal de conversa, migra pro e-mail, a decisão sai num
sistema interno e o aviso final volta pro canal de conversa. Cada pedaço está
registrado em algum lugar. Nenhum lugar tem a sequência.

Quando pedem o histórico completo, alguém vai ter que ir no celular de um
vendedor tirar print. Já vi acontecer.

Se a conversa faz parte da operação, ela é parte da trilha. Isso significa
ingerir a mensagem, o remetente, o horário e o anexo pro seu lado, com o mesmo
identificador de correlação de tudo o mais. É trabalho e é caro. Mas a
alternativa é ter um buraco exatamente no trecho que interessa.

Vale a regra prática: **se o dado influenciou a decisão, ele pertence à trilha,
mesmo que tenha entrado por um canal que não é seu.**

## O teste

Pegue uma operação real, de uns seis meses atrás, e tente responder sozinho,
só com consulta ao sistema:

1. Quem iniciou, quando, e por qual canal?
2. Quais documentos entraram, quando, e quem enviou?
3. Em que momento cada mudança de status aconteceu, e quem fez?
4. Quem aprovou, com base em quê, e o que ele estava vendo na hora?
5. O que foi comunicado ao cliente, quando, e por qual meio?
6. Alguém alterou algum desses registros depois? Como você prova?

Se você não consegue responder as seis em uma tarde, sem pedir ajuda pra
ninguém e sem abrir o banco de produção na mão, a trilha ainda não existe.

A pergunta 6 é a que separa log de trilha. Quase todo mundo passa nas cinco
primeiras e trava nela.

## Custo

Trilha completa cresce rápido e cresce pra sempre. Separe do banco
transacional, defina período quente e período frio, e trate arquivamento como
parte do design e não como problema do futuro. Isso é o [capítulo 3](03-retencao.md).
