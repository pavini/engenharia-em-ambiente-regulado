# 1. Por que isso chega em você tarde demais

Auditoria não chega na engenharia no começo do projeto. Chega quando o sistema já
está em produção, já tem cliente, e já tem dois anos de dado dentro.

O motivo é banal. Compliance e jurídico conversam com a diretoria, a diretoria
conversa com produto, e produto traz pro time uma demanda que já vem traduzida
em funcionalidade. "Precisamos de um relatório de movimentações." "Precisamos
guardar os documentos por cinco anos." Isso chega como card no board e é tratado
como card no board.

O problema é que a maior parte do que a auditoria vai pedir não é funcionalidade.
É propriedade do sistema. E propriedade de sistema não se adiciona depois num
sprint, porque depende de decisão que já foi tomada lá atrás.

## O que dá pra resolver depois

Boa parte, honestamente. Relatório, exportação, tela de consulta, retenção de
arquivo, política de acesso. Tudo isso é trabalho e custa tempo, mas é trabalho
que sempre pode ser feito.

## O que não dá

Três coisas, e é nelas que vale gastar atenção no design.

**Dado que foi sobrescrito não volta.** Se o seu registro de proposta tem um
campo `status` que o sistema atualiza no lugar, você não tem o histórico. Você
tem o presente. Quando pedirem quando aquele status mudou, quem mudou e o que
tinha antes, a resposta vai ter que sair do log da aplicação, se ele existir,
se tiver sido retido, e se alguém conseguir correlacionar.

**Evento que não foi registrado não existe.** Você não consegue reconstruir
retroativamente que o cliente recebeu um aviso em março. Ou o envio foi
registrado na hora, ou aquilo aconteceu num lugar onde ninguém olha, como o
log do provedor de e-mail, que tem retenção curta e não é seu.

**Contexto que ficou fora do sistema também não volta.** A conversa no WhatsApp,
o e-mail que o comercial mandou, o áudio no celular do vendedor, a decisão que
foi tomada numa reunião. Se a operação passou por ali e nada disso entrou no
sistema, a trilha tem um buraco no meio, e é sempre nesse buraco que a pergunta
cai.

## O padrão que se repete

Em todas as vezes que passei por isso, a pergunta era alguma variação da mesma
coisa: **me mostre tudo que aconteceu com essa operação, em ordem, com data,
com autor, e me prove que ninguém mexeu depois.**

Não é uma pergunta sobre relatório. É uma pergunta sobre como você guardou.

Se o seu sistema responde isso, o resto da auditoria é trabalho chato mas
tranquilo. Se não responde, nenhuma quantidade de relatório novo resolve,
porque o dado não está lá.

## O que fazer com isso

Se você está começando um sistema que vai operar em ambiente regulado, ou que
tem chance de passar a operar, o [capítulo 2](02-trilha-de-auditoria.md) é
sobre as decisões que precisam ser tomadas agora.

Se o sistema já existe e você está lendo isso porque a auditoria já foi marcada,
o [checklist](99-checklist.md) é o lugar de começar: ele separa o que dá pra
resolver em semanas do que vai precisar de uma conversa honesta sobre prazo.
