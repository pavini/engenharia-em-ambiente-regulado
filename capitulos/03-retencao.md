# 3. Retenção, e o conflito com a LGPD

Retenção parece o capítulo mais simples e é onde mora a contradição mais chata
da área: a LGPD dá ao titular o direito de pedir eliminação dos dados dele, e a
regulação setorial obriga a guardar boa parte desses mesmos dados por anos.

As duas coisas são verdade ao mesmo tempo. O time de engenharia costuma
descobrir isso quando implementa o botão de excluir conta.

## A resolução, em uma frase

A LGPD prevê que a obrigação legal ou regulatória de guarda é base para manter
o dado mesmo depois de um pedido de eliminação. Ou seja: você não apaga tudo.
Você apaga o que não tem obrigação de reter, e mantém o resto com acesso
restrito, fora do uso comum.

Na prática isso vira uma decisão por categoria de dado, não uma decisão única.

## Classifique antes de decidir

Não dá pra definir política de retenção sobre "os dados". Separe:

**Dado de operação** — a proposta, o contrato, a movimentação, o documento que
embasou a decisão. É o que tem prazo regulatório e é o que você vai reter,
independente de pedido de eliminação.

**Trilha de auditoria** — o registro de que as coisas aconteceram. Segue o prazo
da operação a que se refere, normalmente o mais longo de todos. É append-only,
então não é candidata a eliminação seletiva de qualquer jeito.

**Dado cadastral e de contato** — nome, telefone, e-mail, endereço. Parte segue
a operação, parte é só conveniência de marketing e relacionamento. Essa segunda
parte é a que realmente sai num pedido de eliminação.

**Dado de comportamento e navegação** — analytics, sessão, clique. Quase nunca
tem obrigação de retenção e quase sempre é guardado pra sempre por inércia.
É o primeiro lugar onde você reduz risco de graça.

**Log técnico** — stack trace, requisição, métrica. Não é trilha. Prazo curto,
semanas ou poucos meses. O problema é que log técnico costuma vazar dado
pessoal dentro do payload, e aí ele herda um problema que não era dele.

## Onde os prazos moram

Os prazos não são uma lista universal. Vêm de três lugares diferentes e se
sobrepõem:

- a regulação do seu setor, que é quem dá o prazo mais longo;
- a legislação geral, incluindo prazos prescricionais de matéria cível,
  tributária e trabalhista;
- o contrato com o cliente e com o parceiro, que às vezes pede mais do que a lei.

**Isso você não decide sozinho.** Pegue com o jurídico ou o compliance a lista
por categoria, por escrito, e guarde essa lista junto do código. Quando a
auditoria perguntar por que aquele dado foi apagado, a resposta precisa apontar
para um documento, não para a memória de alguém.

## O que implementar

**Data de expurgo calculada, não idade do registro.** O prazo conta a partir de
um marco de negócio, normalmente o encerramento da operação, não da criação da
linha. Uma operação que ficou aberta por quatro anos começa a contar depois.
Guarde a data de expurgo como coluna, calculada quando o marco acontece, e
deixe o expurgo ser uma consulta simples em cima dela.

**Marcação de retenção legal.** Operação em disputa, questionamento judicial ou
pedido de regulador não expira, mesmo que o prazo tenha vencido. Um campo que
bloqueia o expurgo, com motivo e responsável, resolve. Sem isso alguém vai
apagar exatamente o que estava sendo discutido.

**Eliminação que registra.** Apagar por pedido do titular é, ele próprio, um
evento de trilha: quem pediu, quando, o que foi eliminado, o que foi retido e
com que base. Auditoria vai pedir a prova de que você atendeu, e a prova não
pode ter sido apagada junto.

**Anonimização como meio-termo.** Para dado que você quer manter agregado mas
não precisa identificado, anonimizar resolve os dois lados. Só que anonimizar
de verdade é mais difícil do que trocar o nome por asterisco: se der pra
reidentificar cruzando com outra tabela sua, não está anonimizado.

**Expurgo que roda e deixa rastro.** Job agendado, com relatório do que foi
eliminado em cada execução, e o relatório retido. Política de retenção que
nunca executou é pior do que não ter política, porque você declarou uma coisa
e fez outra, e é isso que aparece no relatório da auditoria.

## O erro comum

Guardar tudo pra sempre parece a opção segura e não é. Aumenta superfície de
vazamento, aumenta custo, e coloca você em desacordo com a própria política
que escreveu. Dado que passou do prazo e não foi eliminado é um achado de
auditoria como qualquer outro.
