# 5. Mudança em produção

Pediram a lista de tudo que mudou em produção num trimestre. O time entregou o
histórico do pipeline, que estava bom: cada publicação com data, com o commit e
com quem aprovou. A conversa ia bem até alguém perguntar sobre uma correção de
quatrocentos e poucos registros que tinha acontecido numa terça-feira. Aquilo
não estava na lista, porque não tinha passado pelo pipeline. Foi um script que
alguém rodou direto no banco.

O problema não foi a correção. Correção de dado acontece e vai continuar
acontecendo. O problema foi que a lista entregue estava incompleta e ninguém no
time sabia disso na hora de entregar.

## Comece pelo inventário de caminhos

Antes de controlar mudança, liste por onde a mudança entra. Na maioria dos
sistemas que eu vi, são cinco caminhos, e só o primeiro costuma ter controle
decente:

1. Publicação de código pelo pipeline.
2. Script executado direto no banco de produção.
3. Feature flag ligada ou desligada.
4. Parâmetro de negócio editado numa tela de administração.
5. Alteração no próprio pipeline.

Do ponto de vista de quem audita, os cinco são a mesma coisa: o sistema passou a
se comportar de um jeito diferente ou o dado passou a ser outro. Se você só
consegue listar o primeiro, a sua resposta cobre uma fração do que foi
perguntado.

Vale fazer esse inventário com o time antes de qualquer outra coisa deste
capítulo. Leva vinte minutos e costuma aparecer um sexto caminho que ninguém
tinha lembrado.

## 1. Publicação de código

Esta parte é a mais bem resolvida da indústria e eu não vou repetir tutorial de
pipeline. O recorte que interessa: precisa ser consultável, não reconstruível.

A pergunta que chega é "me mostre tudo que foi publicado em produção entre tal
e tal data, com o que mudou e quem autorizou". Se a resposta exige alguém
cruzando histórico de pipeline com lista de commits e mensagem de canal de
equipe, você tem o dado e não tem a resposta. Guarde publicação como registro
seu, com versão, commit, autor, aprovador, data e resultado, no mesmo lugar onde
mora o resto da trilha.

Aprovação precisa ser de outra pessoa, pelo mesmo motivo do
[capítulo 4](04-segregacao-de-acesso.md). Se o sistema aceita que o autor aprove
a própria publicação, a segregação existe no processo e não no código.

## 2. O script no banco de produção

Este é o caminho que mais gera achado, e é também o mais legítimo. Sistema
gera dado errado, alguém precisa corrigir, e não existe tela para aquilo. Não
adianta proibir. O que funciona é transformar isso num procedimento com forma.

O que dá para exigir sem travar o time:

- **O script é arquivo versionado.** Vai para o repositório antes de rodar, e
  fica lá depois. Script que existiu só no terminal de alguém não pode ser
  mostrado seis meses depois.
- **Outra pessoa revisa.** É a mesma revisão de código de sempre, e o revisor
  está olhando principalmente a cláusula de filtro.
- **A contagem esperada vem antes.** Quem escreveu declara quantas linhas o
  script deve afetar. Roda a contagem, confere, e só então executa. Quando o
  número não bate, a execução para. Isso pega o filtro errado antes do estrago,
  e é o controle mais barato que existe nessa lista.
- **Quem executa não é quem escreveu.** Mesma regra do capítulo 4, e aqui ela
  custa pouco porque a execução é rara.
- **O resultado fica registrado.** Quantas linhas mudaram de fato, por quem,
  quando, e qual chamado ou incidente motivou.

Falta o ponto que quase todo mundo esquece, e é o mais importante dos seis.

**A correção precisa aparecer na trilha do negócio, não só no registro de
operação.** Se você corrigiu o status de quatrocentas propostas, a trilha
daquelas quatrocentas propostas tem que dizer que houve uma correção
administrativa, com o motivo e o autor. Sem isso, a trilha do capítulo 2 passa
a mentir: ela mostra uma mudança de estado sem autor humano, ou pior, não mostra
mudança nenhuma e o valor simplesmente está diferente do que a sequência de
eventos explica.

Quando o auditor escolher uma dessas propostas para examinar, e ele vai
escolher, a diferença entre um evento de correção com justificativa e um pulo
inexplicado no histórico é a diferença entre uma observação e um problema.

## 3. Feature flag e parâmetro de negócio

Aqui mora a mudança que não aparece em lugar nenhum.

Ligar uma flag muda o comportamento de produção sem commit, sem publicação, sem
aprovação e, na maioria dos painéis, sem registro de quem fez. O time trata
isso como configuração. Quem audita trata como mudança em produção, porque o
efeito é exatamente esse.

O mesmo vale, e com peso maior, para parâmetro de negócio editável em tela.
Taxa, limite, prazo, teto, percentual. Em serviços financeiros esses campos têm
efeito direto em dinheiro e costumam ter menos controle do que uma publicação
que mudou a cor de um botão. Já vi tela de parâmetro sem trilha nenhuma
convivendo com pipeline caprichado.

O que precisa existir:

- O painel de flag e a tela de parâmetro são sistemas de produção. Acesso
  nominal, com as regras do capítulo 4.
- Toda mudança registra quem, quando, valor anterior, valor novo e motivo. É a
  mesma estrutura de registro do [capítulo 2](02-trilha-de-auditoria.md), e não
  precisa ser uma tabela nova.
- Parâmetro com efeito financeiro merece a mesma segregação da publicação:
  quem edita não é quem aprova.
- Flag tem dono e data de revisão. Flag que passou de temporária a permanente
  virou configuração do produto e deveria sair do painel e entrar no código,
  onde passa por revisão.

## 4. Quem controla o pipeline controla a produção

O pipeline é quem executa a publicação, então ele tem as credenciais que as
pessoas não têm. Isso resolve um problema e cria outro.

Se o desenvolvedor não tem acesso ao servidor, mas pode alterar o passo de
publicação e fazer isso chegar em produção sozinho, o acesso restrito ao
servidor é decorativo. Ele tem um caminho indireto que faz a mesma coisa, com a
vantagem de ninguém estar olhando.

É o mesmo furo do administrador que edita a própria alçada, no capítulo 4.
Alterar a regra é mais fácil do que burlar a regra.

O que fecha isso: o arquivo de pipeline é código protegido, com revisão
obrigatória de outra pessoa, no mesmo nível do resto. Segredo de produção não
fica acessível a passo que executa comando vindo do repositório sem revisão.
E alteração no pipeline entra na lista de mudanças em produção, porque é uma.

## 5. Mudança emergencial

Vale o desenho do acesso de emergência do capítulo 4 e não vou repetir inteiro.
O caminho precisa existir, porque proibir só empurra para o caminho errado. O
uso gera alerta no momento. A regularização tem prazo curto e produz registro:
o que foi feito, por quem, por que não deu para esperar, e o que foi feito
depois para normalizar.

A frequência continua sendo o termômetro. Mudança emergencial toda semana
significa que o caminho normal está lento demais, e vale resolver isso antes que
alguém de fora aponte.

## 6. Rollback, e o banco que não volta

Todo mundo tem plano de volta no papel. Quase ninguém executou.

Voltar código é a parte fácil. O que trava é o banco. Uma migração que apagou
uma coluna ou mudou o tipo de um campo não tem volta simples, e a versão antiga
do código não roda contra o esquema novo. Aí a decisão na madrugada vira
escolher entre ficar com o defeito ou restaurar backup, que é uma conversa bem
diferente da que estava no plano.

O que resolve é separar as duas coisas. Migração que só adiciona vai antes, em
publicação própria. O código novo passa a usar o campo novo. A remoção do campo
antigo vai depois, numa terceira publicação, quando já não tem código
dependendo dele. Fica mais passo, e em troca toda publicação tem volta possível
sem tocar no banco.

Isso também muda o que você responde quando perguntam sobre capacidade de
reversão. Em vez de descrever um plano, você aponta a última vez que aconteceu.

## 7. Dado de produção fora de produção

O caminho mais rápido para ter ambiente de teste parecido com o real é copiar a
base de produção. É comum, é prático, e cria uma cópia dos dados dos seus
clientes com acesso mais largo, sem retenção definida, fora de todo o controle
do capítulo 4 e provavelmente fora do que a sua política de retenção descreve.

O mínimo é saber que a cópia existe e onde está. O caminho melhor é mascarar na
geração da cópia, e tratar qualquer base derivada de produção com as mesmas
regras de acesso da original enquanto ela tiver dado real.

Isso encosta direto na LGPD, que é um capítulo próprio mais à frente.

## A outra escolha

**Migração em três passos ou janela de indisponibilidade.** Os três passos dão
reversão sem tocar no banco, e custam mais publicações e um período com o
esquema duplicado. A janela é mais simples e exige um horário em que a operação
pode parar. Se o seu sistema tem madrugada sem movimento e o time é pequeno, a
janela é uma escolha legítima, desde que ensaiada.

**Flag em painel ou no código.** O painel existe para mudar em segundos, e é
isso que o torna perigoso. O código passa por revisão e publicação, e é isso
que o torna lento. A divisão que funciona: o que altera comportamento visual ou
liga funcionalidade fica no painel com trilha; o que tem efeito financeiro ou
regulatório fica no código.

**Script revisado ou tela de correção.** Script é mais rápido para o caso
único. Quando o mesmo tipo de correção aparece pela terceira vez, construir a
tela sai mais barato e resolve de vez, porque a tela já nasce com autor,
justificativa e trilha, sem depender de ninguém lembrar do procedimento.

## O teste

Mesma lógica dos capítulos anteriores. Tente responder sozinho:

1. Liste tudo que mudou em produção no último trimestre, considerando os cinco
   caminhos. Quantos lugares você teve que consultar?
2. Pegue a última correção de dado feita por script. Quem escreveu, quem
   revisou, quem executou, quantas linhas mudaram e onde está o registro?
3. Essa correção aparece na trilha das operações que ela afetou?
4. Quem ligou a última feature flag em produção, e quem mudou o último
   parâmetro de negócio? Quando, e com qual valor anterior?
5. Quem pode alterar o passo de publicação do pipeline sem revisão de outra
   pessoa?
6. Qual foi a última vez que vocês voltaram uma versão de verdade? O banco
   voltou junto?
7. Existe dado real de cliente hoje em algum ambiente que não seja produção?

A pergunta 3 é a que costuma surpreender o time. Ela liga este capítulo ao
[capítulo 2](02-trilha-de-auditoria.md), e é onde uma trilha bem feita começa a
perder valor por causa de uma correção mal registrada.

## Onde isso continua

Publicação, correção e volta atrás tratam do dia em que alguma coisa muda por
decisão sua. O próximo capítulo é sobre o dia em que alguma coisa muda sem
ninguém decidir: continuidade e recuperação. Se você quer o recorte rápido
antes disso, a seção de mudança do [checklist](99-checklist.md) tem os itens
que dá para verificar com o time em uma tarde.
