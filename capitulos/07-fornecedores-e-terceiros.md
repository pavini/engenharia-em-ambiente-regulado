# 7. Fornecedores e terceiros

Pediram a lista dos fornecedores que tocam dado de cliente. O time pegou a
relação de contratos que o financeiro tinha e entregou. Estava incompleta e
levou uns dias até alguém perceber.

Faltavam as ferramentas que tinham entrado pelo cartão de alguém e nunca
passaram por contrato. Faltava o serviço de envio de e-mail, que tinha nome,
endereço e o conteúdo da comunicação. E faltava o agregador de log, que recebia
o corpo inteiro das requisições, com documento e valor dentro, havia mais de um
ano.

Nenhum deles tinha sido escondido. Eles simplesmente nunca foram entendidos
como fornecedores, porque quem contratou foi a engenharia e não a área de
compras.

Este capítulo é sobre o que continua sendo seu quando a execução é de outro. A
regra geral, em praticamente toda regulação que eu vi, é que a
responsabilidade perante o cliente e perante o regulador não se transfere junto
com o serviço. Você terceiriza a execução, não a resposta.

## 1. A lista que ninguém tem

O inventário de fornecedores da engenharia quase nunca é igual ao do
financeiro. Ferramenta paga em cartão, plano gratuito, conta criada num
sábado para testar e que ficou, extensão instalada no repositório. Nada disso
aparece numa relação de contratos.

Comece pelos dois critérios, que geram listas diferentes e sobrepostas:

- **Toca dado de cliente.** Recebe, armazena, processa ou consegue ler.
- **Consegue parar a operação.** Se cair, você para, mesmo que nunca tenha
  visto um dado pessoal.

Onde procurar, na prática, porque a planilha vai estar desatualizada:

- Variáveis de ambiente com URL e credencial de serviço externo.
- Regras de saída do firewall, ou o log do proxy. O tráfego de saída sabe mais
  sobre os seus fornecedores do que qualquer documento interno.
- Faturas de cartão corporativo e assinaturas em nome de pessoas do time.
- Integrações autorizadas nas contas de nuvem, de repositório e de e-mail.

Biblioteca que você compila junto não é fornecedor no sentido deste capítulo,
é dependência de código, e o risco dela é outro. Serviço chamado em tempo de
execução é fornecedor, mesmo que o plano seja gratuito e ninguém tenha assinado
nada.

## 2. O log que vaza sem ninguém decidir

Esta é a parte deste capítulo que é puramente de engenharia, e é a que eu
levaria primeiro para o time.

Monitoramento, rastreamento de erro, agregador de log e ferramenta de
desempenho recebem o que a aplicação manda. E o que a aplicação manda,
normalmente, é o corpo da requisição inteiro, o cabeçalho inteiro e o objeto de
contexto inteiro. Dentro disso vai documento, nome, endereço, valor, às vezes
o token de sessão.

Isso é envio de dado pessoal para um terceiro, com frequência para fora do
país, decidido por ninguém. Não passou por avaliação, não está no contrato que
o jurídico revisou, e não aparece em nenhum inventário. Quando o assunto vira
pergunta de auditoria, a conversa fica ruim rápido, porque o volume é grande e
o histórico é longo.

O que resolve:

- **Mascarar na origem.** A aplicação decide o que sai, antes de sair. Máscara
  configurada no painel do fornecedor significa que o dado chegou lá e depois
  foi escondido.
- **Lista de campos que nunca saem**, mantida no código, aplicada por padrão,
  com o oposto da lógica comum: o que não está explicitamente liberado vai
  mascarado.
- **Teste automatizado da máscara.** Essa configuração para de funcionar em
  silêncio quando alguém renomeia um campo ou aninha um objeto. Um teste que
  falha quando um documento aparece no que seria enviado custa pouco e evita o
  ano inteiro de vazamento.
- **A mesma regra para o dump de suporte.** Quando o time exporta um pedaço da
  base e manda para o fornecedor analisar um problema, aquilo é a mesma
  transferência, só que manual. Vale a regra do dado de produção fora de
  produção, do [capítulo 5](05-mudanca-em-producao.md).

## 3. Acesso do fornecedor ao seu ambiente

Consultoria que está tocando um projeto, integrador, suporte do fabricante que
pede acesso para investigar. Vale tudo do [capítulo 4](04-segregacao-de-acesso.md)
e mais duas coisas.

A primeira é prazo. Acesso de fornecedor nasce com data de fim, porque o
projeto tem data de fim. O que eu mais vi foi o acesso do projeto que acabou
há oito meses continuar ativo, e ninguém lembrar de quem era aquela conta.

A segunda é que conta de fornecedor é nominal também. Uma conta usada por
quatro consultores devolve as mesmas respostas ruins que a conta compartilhada
interna: você sabe que a empresa fez, não quem fez. E se um dia precisar
cobrar, a diferença importa.

O que o consultor faz precisa cair na mesma trilha do resto. Não vale um
canal separado de registro, porque a pergunta que chega é sobre a operação, e
ela não distingue quem é da casa.

## 4. A evidência que você pede na contratação

O auditor vai perguntar como você sabe que o fornecedor trata o dado
direito. Contrato é a resposta do jurídico. A resposta da engenharia é o que
você recebe dele de forma recorrente e o que consegue mostrar.

Vale pedir, e vale pedir antes de assinar, porque depois não existe alavanca:

- Relatório de auditoria independente ou certificação, com a data da última
  emissão. Certificado vencido é informação também.
- Compromisso de disponibilidade com relatório de apuração, e não só o número
  no material comercial.
- **Prazo e canal de notificação de incidente.** Este é o mais importante da
  lista e o mais esquecido. Se o fornecedor for invadido, você precisa saber
  em quanto tempo ele avisa e por qual meio. Sem isso escrito, você fica
  sabendo pelo jornal, junto com o seu cliente, e o relógio do seu próprio
  dever de comunicar já estará correndo.
- A lista dos subcontratados dele, porque o fornecedor do seu fornecedor
  também está na cadeia, e a responsabilidade percorre a cadeia inteira.

Nada disso é trabalho de engenharia sozinho. O que é trabalho de engenharia é
dizer quais fornecedores merecem essa exigência, e isso sai do inventário da
seção 1.

## 5. Concentração e saída

Duas perguntas que aparecem cada vez mais.

A primeira é o que acontece se esse fornecedor parar. Se a resposta for que a
operação para junto e não existe alternativa, isso é uma informação que a
diretoria precisa ter antes de a auditoria contar.

A segunda é mais concreta e é a que o time consegue responder: se o contrato
acabar em trinta dias, você consegue tirar o seu dado de lá, num formato
utilizável, sem depender da boa vontade de ninguém?

Responder "acho que tem uma API" não conta. Extraia uma vez, de verdade, e
veja o que sai. É comum descobrir que a exportação não traz o histórico, ou
traz sem os anexos, ou tem limite de volume que faria a extração levar semanas.
Melhor descobrir isso enquanto o relacionamento é bom.

O que costuma prender, na prática: formato proprietário, dado que só existe lá
e nunca foi replicado do seu lado, e integração acoplada ao modelo dele. Dá
para reduzir os três com decisão de design, e todos os três ficam mais caros
de resolver com o tempo.

## 6. O que muda no seu código

Curto, e vale para toda chamada a serviço externo:

- O identificador de correlação do [capítulo 2](02-trilha-de-auditoria.md)
  atravessa a chamada externa. Sem isso, o trecho da operação que passou pelo
  terceiro fica órfão na trilha.
- Toda chamada tem tempo limite e comportamento definido para a falha, que é a
  decisão do [capítulo 6](06-continuidade-e-recuperacao.md).
- **Guarde a resposta do terceiro, não só o resultado dela.** Se a decisão foi
  tomada com base numa consulta externa, o que sustenta a decisão é o retorno
  que você recebeu, com data e conteúdo. Registrar "aprovado" e descartar a
  resposta deixa você sem como mostrar em que se baseou. É a mesma ideia da
  pergunta sobre o que o aprovador estava vendo na hora.
- O registro dessa chamada não pode conter o que a seção 2 manda mascarar.

## A outra escolha

**Mascarar ou não enviar.** Mascarar preserva a capacidade de diagnosticar e
depende de a máscara estar certa para sempre. Não enviar o corpo da requisição
elimina o risco e custa tempo de investigação em cada incidente. Para o campo
que nunca ajuda a depurar, e documento é o caso clássico, não enviar é a
escolha melhor, porque não depende de configuração nenhuma continuar correta.

**Um fornecedor ou dois.** Dois reduzem concentração e multiplicam contrato,
integração, custo e superfície de acesso ao seu dado. Para quase tudo, um só
com saída testada é melhor. A exceção é a dependência que para a operação
inteira, onde vale ter o segundo caminho pronto mesmo sem uso.

**Serviço gerenciado ou hospedado por você.** Hospedar a ferramenta de
observabilidade resolve transferência, retenção e subprocessador de uma vez, e
te dá mais um sistema para operar, com backup, atualização e acesso próprios.
Compensa quando o volume de dado sensível é alto e o time tem folga. Na maioria
dos casos, o gerenciado com máscara na origem sai melhor.

## O teste

1. Liste os fornecedores que tocam dado de cliente. Compare com a lista de
   contratos do financeiro. Quantos aparecem só numa das duas?
2. Pegue o seu agregador de log ou ferramenta de erro e procure por documento,
   endereço ou telefone de cliente nos últimos registros. Achou?
3. Quantos acessos de fornecedor estão ativos hoje, e quantos deles são de
   projeto já encerrado?
4. Para o seu fornecedor mais crítico, em quanto tempo ele é obrigado a avisar
   se sofrer um incidente? Onde isso está escrito?
5. Se ele encerrasse o contrato em trinta dias, você consegue extrair seus
   dados num formato utilizável? Já testou?
6. Quem são os subcontratados dele que tocam o seu dado?
7. Numa decisão que dependeu de consulta externa, você ainda tem a resposta que
   recebeu, ou só o resultado que gravou?

A pergunta 2 costuma ser a mais desconfortável, e é a que dá para verificar em
dez minutos, agora.

## Onde isso continua

Boa parte deste capítulo é sobre dado pessoal saindo do seu sistema para a mão
de outra empresa. O próximo capítulo fecha esse assunto pelo lado de quem
escreve o backend: o que a LGPD exige que exista no código, além do banner de
consentimento e da política de privacidade. A seção de terceiros do
[checklist](99-checklist.md) tem o recorte rápido para percorrer antes disso.
