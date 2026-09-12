# Checklist: se a auditoria bater amanhã

Para percorrer com o time em uma tarde. A ideia não é passar em tudo. É saber
onde você está antes que alguém de fora descubra.

Marque honestamente. "Mais ou menos" conta como não.

## Trilha

- [ ] Existe um identificador de operação que atravessa todos os serviços,
      filas e jobs envolvidos.
- [ ] A trilha é append-only, e o usuário de banco da aplicação não tem
      permissão de `UPDATE` nem `DELETE` nela.
- [ ] Todo registro tem quando (com fuso), quem, o quê, e o valor anterior
      quando houve mudança de estado.
- [ ] Ação feita por delegação, suporte ou operador em nome de terceiro está
      identificada como tal.
- [ ] Ações do sistema dizem qual sistema e o que as disparou.
- [ ] O vocabulário de ação é de negócio e estável, não rota de API.
- [ ] Existe alguma forma de demonstrar que a sequência não foi alterada depois.

## Canal

- [ ] Todo canal por onde a operação passa tem o registro do lado de cá.
- [ ] Anexo recebido por canal externo é armazenado por você, não só
      referenciado no provedor.
- [ ] Comunicação enviada ao cliente é registrada no envio, não apenas no
      provedor de e-mail ou mensagem.

## Retenção

- [ ] Existe uma lista escrita de prazo por categoria de dado, vinda do
      jurídico ou do compliance.
- [ ] O prazo é contado a partir de um marco de negócio, não da criação
      do registro.
- [ ] Existe marcação que impede expurgo de operação em disputa ou sob
      questionamento.
- [ ] O expurgo roda de fato, e cada execução deixa relatório retido.
- [ ] Pedido de eliminação do titular gera evento de trilha dizendo o que
      foi eliminado e o que foi retido, com a base.

## Acesso

- [ ] Acesso a produção é nominal. Não existe conta compartilhada.
- [ ] Quem aprova não é quem executa, e o sistema impede, não apenas
      desencoraja.
- [ ] Acesso de suporte a dado de cliente é registrado e tem justificativa.
- [ ] Existe revisão periódica de quem tem acesso a quê, com evidência
      da revisão.
- [ ] Desligamento revoga acesso no mesmo dia, e há como provar isso.

## Mudança

- [ ] Existe a lista dos caminhos por onde produção muda: publicação, script,
      feature flag, parâmetro de negócio e o próprio pipeline.
- [ ] Todo deploy em produção é rastreável até o commit e até quem aprovou.
- [ ] Mudança direta em banco de produção é exceção registrada, não rotina.
- [ ] Existe registro de quem executou script em produção, quando e por quê.
- [ ] Correção de dado por script aparece também na trilha das operações
      que ela afetou.
- [ ] Ligar ou desligar feature flag em produção fica registrado, com quem
      e quando.
- [ ] Parâmetro de negócio editável em tela tem trilha com valor anterior
      e autor.
- [ ] Alteração no pipeline de publicação passa por revisão de outra pessoa.
- [ ] Já houve reversão de verdade, e havia como voltar sem restaurar backup.
- [ ] Não existe dado real de cliente em ambiente que não seja produção, ou
      ele está mascarado.

## Continuidade

- [ ] O plano de recuperação existe por escrito.
- [ ] Ele já foi testado, e existe evidência de cada teste com data e
      resultado.
- [ ] O tempo de recuperação medido no teste bate com o que foi prometido
      no papel.
- [ ] Backup já foi restaurado alguma vez. Backup nunca restaurado é
      hipótese, não backup.
- [ ] Existe a lista do que precisa existir para o sistema voltar, e ela
      já foi comparada com o que o backup cobre.
- [ ] Arquivo, fila, segredo, certificado e configuração feita no console
      estão cobertos, não só o banco.
- [ ] Dá para recuperar um registro ou documento sem restaurar a base
      inteira.
- [ ] Está escrito o que o sistema faz quando a dependência externa cai:
      recusa a operação ou aceita e reprocessa depois.
- [ ] Incidente gera registro com hora da ocorrência, hora da detecção,
      hora do retorno e o que se perdeu.
- [ ] O prazo de comunicação a cliente e a regulador foi levantado com o
      jurídico antes de precisar dele.
- [ ] Reprocessamento é idempotente, e aparece na trilha identificado
      como reprocessamento.

## Terceiros

- [ ] Existe lista dos fornecedores que tocam dado de cliente.
- [ ] Existe também a lista dos que conseguem parar a operação, mesmo sem
      tocar dado.
- [ ] A lista da engenharia foi comparada com a de contratos do financeiro.
- [ ] Para cada um, existe contrato com cláusula de tratamento de dado.
- [ ] Log, telemetria e rastreamento de erro enviados a terceiro passam por
      máscara aplicada na origem, e existe teste que falha se ela quebrar.
- [ ] Amostra ou dump enviado ao fornecedor para análise segue as mesmas
      regras do dado original.
- [ ] Acesso de consultoria ao código e ao ambiente é nominal, temporário
      e revogado ao fim.
- [ ] Nenhum acesso de fornecedor de projeto encerrado continua ativo.
- [ ] O contrato do fornecedor crítico diz em quanto tempo e por qual canal
      ele avisa um incidente.
- [ ] Existe evidência periódica do fornecedor, como relatório de auditoria
      independente ou certificação, e alguém a recebe.
- [ ] Você já extraiu seus dados de lá uma vez, e sabe o que sai e o que
      não sai.
- [ ] Os subcontratados que tocam o seu dado são conhecidos.

## A pergunta final

Pegue uma operação real de seis meses atrás e reconstrua a história dela
inteira, sozinho, em uma tarde, só com consulta ao sistema.

Se conseguir, o resto é trabalho. Se não conseguir, é aí que começa.
