# 8. LGPD para quem escreve backend

Quando a LGPD chega no time de engenharia, ela normalmente chega como duas
tarefas: colocar o aviso de cookies e publicar a política de privacidade. As
duas são de front-end e de jurídico, e nenhuma das duas tem a ver com o
trabalho que realmente sobra.

O trabalho que sobra aparece meses depois, na primeira vez que alguém pede uma
coisa simples: diga todo lugar onde está o dado deste cliente, e com quem ele
foi compartilhado. Essa pergunta é de modelagem, e ela não tem resposta rápida
em sistema que não foi construído com ela em mente.

O [capítulo 3](03-retencao.md) já tratou de retenção e do conflito entre o
direito de eliminação e a obrigação de guarda. Aqui é o resto: o que precisa
existir no código.

## 1. O mapa do dado pessoal é um mapa do seu schema

Não dá para responder nada sem saber onde o dado pessoal está. E ele nunca
está só nas tabelas óbvias.

Está no campo de observações, que o operador usa para escrever o que quiser e
onde já vi de tudo. Está no anexo que alguém subiu. Está no índice de busca,
que é uma cópia. Está no cache. Está na fila, em mensagem que ainda não foi
consumida. Está no log da aplicação. Está no relatório que alguém exportou
para uma planilha e guardou numa pasta compartilhada.

O que funciona é tratar isso como parte da modelagem e não como documento de
compliance. Para cada campo que guarda dado pessoal, três informações:

- Qual categoria de dado é.
- Qual a finalidade que justifica ele existir.
- Qual a base legal que sustenta essa finalidade.

Isso pode viver num arquivo ao lado das migrações, num atributo no modelo, ou
numa tabela de metadados. O formato importa menos do que estar perto do
código, porque documento separado desatualiza na primeira sprint e o auditor
vai comparar com o banco.

O campo de texto livre é o caso mais difícil e merece decisão explícita. Ou
você aceita que ele contém dado pessoal imprevisível e trata o campo inteiro
com o maior nível de proteção que você tem, ou você muda a tela para não ter
texto livre naquele ponto. Fingir que ele é neutro é a opção que não funciona.

## 2. Base legal é propriedade do dado, não da empresa

Esse é o ponto que mais confunde quem vem do lado técnico, e tem consequência
direta no código.

A lei traz várias bases que autorizam o tratamento. Consentimento é uma delas,
e é a mais frágil de todas, porque pode ser retirado a qualquer momento. Em
serviços financeiros, a maior parte do dado que sustenta a operação não está
apoiada em consentimento, e sim em execução de contrato e em cumprimento de
obrigação legal ou regulatória.

A consequência prática: se o seu modelo trata consentimento como a chave que
libera o dado da operação, você criou um problema que não precisava existir.
No dia em que o titular retirar o consentimento, o sistema vai tentar apagar
ou bloquear coisa que você é obrigado a manter, e alguém vai ter que resolver
isso na mão.

O desenho que se sustenta guarda a base legal por finalidade, e não por
cliente. O mesmo telefone pode estar sendo tratado para executar o contrato e
para enviar oferta comercial. A retirada do consentimento atinge a segunda
finalidade e não a primeira. Se o seu modelo tem uma única marca de
consentimento por pessoa, ele não consegue expressar isso, e a resposta ao
pedido vira uma reunião em vez de uma consulta.

Quando a finalidade e a base estão registradas, o pedido do titular vira
trabalho de sistema. Quando não estão, vira interpretação humana repetida a
cada pedido, e cada interpretação é uma chance de errar.

## 3. Os direitos do titular viram funcionalidade, com prazo

Confirmação de tratamento, acesso, correção, portabilidade, informação sobre
compartilhamento, eliminação. Cada um vira uma funcionalidade, e a lei fixa
prazo de resposta. O prazo e a forma de contagem são assunto para o seu
jurídico, e vale levantar antes de precisar, porque ele é mais curto do que a
maioria dos times imagina.

Dois merecem atenção especial, porque são os que não se resolvem depois.

**Informar com quem o dado foi compartilhado.** Isso só é possível se cada
compartilhamento tiver sido registrado quando aconteceu. Se o seu sistema
manda dado para um bureau, para um serviço de assinatura e para um provedor de
e-mail, e nada disso foi registrado por titular, a resposta vai ser genérica,
do tipo "compartilhamos com prestadores de serviço". É uma resposta que
sobrevive a um cliente e não sobrevive a uma fiscalização. Depende do
[capítulo 7](07-fornecedores-e-terceiros.md) e da trilha do
[capítulo 2](02-trilha-de-auditoria.md), e não se reconstrói.

**Portabilidade.** O dado tem que sair em formato utilizável. É o mesmo teste
que o capítulo 7 manda aplicar nos seus fornecedores, agora apontado para
você. Exporte de verdade uma vez e olhe o resultado antes que um cliente peça.

## 4. Minimização é decisão de design

Todo campo que você coleta é um campo que você vai ter que mapear, proteger,
reter, entregar quando pedirem e apagar quando der. O campo que não existe não
vaza, não aparece no inventário e não precisa de base legal.

Na prática isso aparece em dois lugares.

O primeiro é o formulário. Quando alguém pede um campo novo no cadastro, a
pergunta que vale é qual decisão aquele campo sustenta. Costuma ser
desconfortável e costuma eliminar alguns campos.

O segundo é a API, e esse é mais de engenharia. O objeto serializado inteiro
volta porque o mapeamento fez isso sozinho. A tela usa três campos e a
resposta traz trinta, incluindo documento e data de nascimento que ninguém
pediu. Esses trinta ficam no cache do navegador, no registro do proxy, no
histórico da ferramenta de rede e no agregador de log do capítulo 7. Devolver
só o que a tela usa resolve várias coisas de uma vez, e é trabalho de uma
tarde por endpoint.

## 5. O dado que ficou onde ninguém procura

Eliminar da tabela principal não é eliminar. Vale fazer a lista e conferir um
por um:

- Índice de busca e qualquer réplica de leitura.
- Cache, incluindo o que tem expiração longa.
- Fila e tópico com mensagem retida.
- Log da aplicação e telemetria enviada a terceiro.
- Ambiente de teste alimentado com cópia de produção, do
  [capítulo 5](05-mudanca-em-producao.md).
- Relatório e planilha exportados, que são a parte que você não controla.
- Backup, que é caso à parte e precisa de decisão escrita sobre o que
  acontece quando um dado eliminado volta numa restauração.

Esse último merece nota. Backup normalmente não é expurgado seletivamente, e
não precisa ser, desde que exista controle para não reintroduzir o dado
eliminado quando houver restauração. Isso é uma decisão de engenharia que
precisa estar escrita antes, porque no dia da restauração ninguém vai lembrar.

## 6. Incidente com dado pessoal

O [capítulo 6](06-continuidade-e-recuperacao.md) tratou do incidente do ponto
de vista de disponibilidade. Aqui é o outro lado: quando o incidente envolve
dado pessoal, existe dever de comunicar, e o conteúdo da comunicação exige
coisas que só o sistema pode dizer.

Quais titulares foram atingidos. Quais campos. Em qual janela de tempo. Por
qual caminho o acesso aconteceu.

Se você não tem registro de acesso a dado de cliente, como descrito no
[capítulo 4](04-segregacao-de-acesso.md), a resposta honesta vai ser que não é
possível determinar a extensão. Essa é a pior resposta possível, porque
obriga a tratar o caso pelo pior cenário e comunicar a base inteira.

O prazo de comunicação e para quem comunicar são assunto do jurídico, e a hora
de levantar isso continua sendo antes.

## O teste

1. Liste todos os lugares onde o dado de um cliente específico existe hoje.
   Quantos você lembrou de cabeça e quantos apareceram só depois de procurar?
2. Para três campos do seu cadastro, qual é a finalidade e qual é a base legal?
   Onde isso está escrito?
3. Se um cliente retirar o consentimento de marketing, o que o seu sistema faz
   com o telefone dele que também é usado para executar o contrato?
4. Um cliente pede para saber com quem os dados dele foram compartilhados.
   Você responde com nomes e datas, ou com uma frase genérica?
5. Pegue um endpoint do seu sistema e conte quantos campos ele devolve e
   quantos a tela usa.
6. Se um dado eliminado voltar numa restauração de backup, o que impede ele de
   ser tratado de novo?
7. Num incidente de acesso indevido, você consegue dizer quais titulares foram
   atingidos e em qual janela?

A pergunta 4 é a que separa quem construiu para isso de quem vai ter que
responder com evasiva.

## Onde isso fecha

Com este capítulo o repositório cobre o escopo que eu tinha em mente quando
comecei: por que chega tarde, trilha, retenção, acesso, mudança, continuidade,
terceiros e dado pessoal. O [checklist](99-checklist.md) junta tudo numa lista
para percorrer com o time.

O que vier depois vai vir de discussão. Se você passou por auditoria e viu algo
que não está aqui, ou discorda do que está, abre uma issue.
