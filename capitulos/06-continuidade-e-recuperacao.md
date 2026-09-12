# 6. Continuidade e recuperação

O backup rodava havia dois anos e o painel estava verde o tempo todo. Na
primeira vez que alguém precisou restaurar de verdade, descobrimos que o job
pegava o banco e não pegava o armazenamento de documentos. O banco voltou em
quarenta minutos. Os documentos, que eram justamente o que a operação precisava
mostrar, não estavam em lugar nenhum do backup.

Ninguém tinha mentido no relatório. O job fazia exatamente o que tinha sido
configurado para fazer, e o que ele fazia era menos do que todo mundo
acreditava.

Este capítulo é o que tem menos código dos anteriores e mais evidência. A
maior parte do trabalho aqui é provar que uma coisa funciona, e prova é o que
falta na maioria dos planos de continuidade que eu vi.

## 1. Backup nunca restaurado é hipótese

Job que termina sem erro prova que o job terminou sem erro. Não prova que o
dado está lá, que está íntegro, que está completo, e que alguém consegue
usá-lo.

O teste de verdade é restaurar num ambiente separado e conferir. Contagem por
tabela contra a origem, amostra de registro aberta e lida, arquivo binário
baixado e verificado. Leva algumas horas e é a única coisa nesta lista que
muda uma hipótese em fato.

O que o teste precisa deixar registrado, e essa é a parte que a auditoria pede:

- A data e quem executou.
- De qual ponto no tempo a restauração foi feita.
- Quanto tempo levou, medido, do início ao sistema utilizável.
- O que foi conferido e como.
- O que faltou ou não funcionou, mesmo que tenha sido resolvido depois.

Teste sem registro é teste que não aconteceu, do ponto de vista de quem
pergunta. E o item mais valioso da lista é o último, porque um histórico de
testes que nunca encontrou nada parece bom e costuma significar que a
conferência é rasa.

Vale um segundo cenário além do desastre completo: recuperar um registro só. O
cliente diz que um documento sumiu, ou uma correção apagou o que não devia. Se
a única forma de responder é restaurar a base inteira num ambiente paralelo,
isso vai levar dias e vai acontecer justamente quando o prazo estiver curto.

## 2. O inventário de recuperação não é o inventário de backup

Este é o ponto que eu colocaria primeiro se pudesse escolher um só.

Faça a lista do que precisa existir para o sistema voltar a operar, e depois
compare com o que o backup cobre. As duas listas nunca batem na primeira vez.
O que costuma ficar de fora:

- Arquivo e documento em armazenamento de objeto, que quase sempre está em
  outro lugar e em outro job.
- Mensagem em fila que ainda não foi processada.
- Segredo, chave e certificado, que às vezes só existem no cofre e às vezes só
  existem na cabeça de alguém.
- Configuração feita na mão no console do provedor, que não está em nenhum
  repositório.
- Registro de DNS e a conta que consegue alterá-lo.
- Dado que vive num serviço de terceiro, onde o backup é problema do
  fornecedor e você só descobre a política dele quando precisa.

A pergunta que organiza isso: se a conta do provedor sumisse hoje, com tudo
dentro, o que você teria em outro lugar? A resposta honesta costuma ser
desconfortável, e é ela que define o trabalho do trimestre.

## 3. O número do papel e o número do teste

Todo plano tem um tempo de recuperação prometido e um ponto de perda aceitável.
Os dois costumam ter sido escritos por alguém que nunca cronometrou nada.

Quem audita compara o número do papel com o do último teste. Se o papel diz
quatro horas e o teste mediu onze, o problema não é o onze. É a distância entre
os dois, porque ela mostra que o plano não descreve o sistema.

Duas coisas ajudam a manter isso honesto. A primeira é medir o tempo até o
sistema estar utilizável, com integração de pé e fila drenada, e não até o
banco subir. A segunda é lembrar que a perda real inclui o tempo de detecção:
se o backup é de hora em hora mas o problema levou seis horas para ser
percebido, a janela é de sete, não de uma.

Se o número medido não cabe no que foi prometido, tem duas saídas honestas,
melhorar o sistema ou corrigir o papel. A saída desonesta é manter os dois
diferentes e torcer para ninguém comparar.

## 4. O caso comum não é o desastre

Plano de continuidade costuma ser escrito para o cenário grande, o data center
inteiro fora do ar. O que acontece de verdade, com frequência bem maior, é a
dependência externa que fica indisponível por algumas horas. O provedor de
assinatura, o serviço de consulta, a instituição do outro lado da integração,
o gateway de mensagem.

Aqui a decisão é de engenharia e de produto ao mesmo tempo, e precisa estar
escrita antes: quando a dependência cai, o sistema recusa a operação ou aceita
e reprocessa depois?

As duas respostas são defensáveis, e a escolha muda o que você precisa
construir. Recusar exige mensagem clara e registro da recusa. Aceitar exige
fila durável, controle do que ficou pendente e um limite de quanto tempo você
segura antes de desistir. O que não se sustenta é aceitar sem guardar, porque
aí a operação some no meio e o cliente descobre antes de você.

Em ambiente regulado, aceitar e perder é bem pior do que recusar na hora.

## 5. O incidente é evidência

Quando alguma coisa cai, o que sobra depois é o registro. Ele precisa ter,
no mínimo:

- A hora em que começou, que é diferente da hora em que alguém percebeu.
- A hora da detecção, e como foi detectado, por alerta ou por reclamação de
  cliente.
- O que foi feito, por quem, em ordem.
- A hora do retorno.
- O que se perdeu ou ficou inconsistente, e o que foi feito com isso.
- O que foi comunicado, para quem e quando.

A distância entre a primeira e a segunda linha é o número que mais diz sobre a
operação. Incidente descoberto por reclamação de cliente é uma resposta ruim
para dar, e é a resposta verdadeira em boa parte dos casos.

Sobre comunicação, existe prazo em várias regulações setoriais e ele varia por
setor, por tipo de evento e por porte. Esse é assunto para o seu jurídico, e a
hora de descobrir é antes. Durante o incidente ninguém vai ter tempo de
pesquisar, e o relógio já está correndo desde a primeira linha do registro.

## 6. Reprocessamento sem duplicar

Voltar no ar é metade. A outra metade é acertar o que ficou pendente enquanto
estava fora: a fila que acumulou, o arquivo que não foi processado, a
integração que rejeitou e vai ser reenviada pelo outro lado.

O requisito aqui é idempotência, e ele precisa existir antes do incidente,
porque no dia não dá para implementar. Toda entrada precisa de um
identificador que permita reconhecer que já foi processada, e o reprocessamento
precisa ser seguro de rodar duas vezes.

Em sistema financeiro isso pesa mais do que em outros lugares. Lançamento
duplicado depois de um incidente costuma dar mais trabalho, e mais explicação,
do que a indisponibilidade que o causou. E cada reprocessamento é uma escrita
que precisa aparecer na trilha do [capítulo 2](02-trilha-de-auditoria.md),
identificada como reprocessamento, pelo mesmo motivo da correção por script no
[capítulo 5](05-mudanca-em-producao.md).

## A outra escolha

**Réplica ou backup.** Réplica protege contra perda de máquina e não protege
contra erro: o comando que apagou a tabela chega replicado em segundos. Ela
resolve disponibilidade e não substitui backup. Quem tem só réplica costuma
achar que tem os dois.

**Ambiente de recuperação de pé ou reconstruído na hora.** Manter ambiente
parado custa dinheiro todo mês e entrega tempo de recuperação curto.
Reconstruir sob demanda é barato e transforma o tempo de recuperação numa
promessa que depende de tudo funcionar no pior dia. A escolha é direta: se o
tempo prometido for curto, não existe versão barata.

**Teste completo anual ou parcial rotativo.** O teste completo é o que produz o
número que a auditoria compara com o papel, e é caro. O parcial, restaurando um
sistema por trimestre, encontra mais defeitos ao longo do ano porque acontece
mais vezes. O arranjo que funciona é um completo por ano para medir, e parciais
no meio para achar.

## O teste

Mesma lógica dos capítulos anteriores:

1. Quando foi a última restauração de verdade, em ambiente separado, e onde
   está o registro dela?
2. Quanto tempo levou até o sistema estar utilizável? Esse número bate com o
   que está escrito no plano?
3. Se a conta do seu provedor de nuvem sumisse hoje, o que você teria em outro
   lugar?
4. Um cliente diz que um documento dele sumiu. Você recupera aquele documento
   sem restaurar a base inteira?
5. Quando a integração externa mais importante cai, o sistema recusa ou aceita
   e guarda? Isso está escrito em algum lugar ou é o que o código faz por
   acidente?
6. No último incidente, qual foi a distância entre a ocorrência e a detecção?
7. O reprocessamento depois daquele incidente gerou duplicidade em algum lugar?
   Como você sabe?

A pergunta 3 é a que costuma mudar a prioridade do time. A 4 é a que mais
aparece na prática.

## Onde isso continua

Boa parte do que ficou de fora do seu controle neste capítulo está na mão de
outra empresa: o provedor de nuvem, o serviço de assinatura, a integração do
outro lado. Isso é o próximo capítulo, sobre fornecedores e terceiros. A seção
de continuidade do [checklist](99-checklist.md) tem o recorte rápido para
verificar com o time antes disso.
