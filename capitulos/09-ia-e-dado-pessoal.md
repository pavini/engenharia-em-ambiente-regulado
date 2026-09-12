# 9. IA no time, e o dado que sai com ela

Quando eu fiz o inventário de fornecedores do
[capítulo 7](07-fornecedores-e-terceiros.md) olhando para o meu próprio time, a
parte que mais me surpreendeu não foi o agregador de log. Foi a quantidade de
ferramenta de IA que já estava em uso sem ter passado por lugar nenhum.
Assistente no editor de cada pessoa, extensão no navegador, um serviço de
transcrição que entrava nas reuniões, e um fluxo interno que alguém montou
chamando uma API de modelo.

Nada disso tinha sido escondido. Entrou como entra qualquer ferramenta de
produtividade, pelo caminho mais curto, porque resolvia um problema real no
mesmo dia.

O erro que eu vejo com mais frequência nessa conversa é tratar tudo isso como
um assunto só. São três usos diferentes, com riscos diferentes e com respostas
diferentes, e misturar os três é o que faz a discussão travar.

## Os três usos

1. **IA como ferramenta do time.** O assistente que completa código, o
   resumidor de reunião, o tradutor. O dado de cliente chega lá por acidente.
2. **IA como parte do produto.** O seu sistema chama um modelo, com dado de
   cliente, por decisão de projeto.
3. **IA decidindo sobre o cliente.** Score, triagem, priorização, recusa.

O primeiro é problema de vazamento. O segundo é problema de terceiro e de
transferência. O terceiro é o que a lei trata de forma específica.

## 1. Ferramenta do time

Duas coisas saem daqui, e a segunda é a que preocupa.

A primeira é código proprietário, que é conversa de propriedade intelectual e
não deste repositório. A segunda é dado de cliente, que sai sem ninguém
decidir: a pessoa cola um trecho do log para entender um erro, e dentro
daquele trecho vai documento, nome e valor. É exatamente a mesma transferência
manual do dump para o fornecedor, do capítulo 7, com a diferença de que aqui
acontece várias vezes por dia e não deixa rastro.

O reflexo é proibir. Proibir não funciona pelo mesmo motivo do acesso de
emergência do [capítulo 4](04-segregacao-de-acesso.md) e da mudança
emergencial do [capítulo 5](05-mudanca-em-producao.md): quando o caminho
oficial não existe, a pessoa usa o caminho de fora. No caso da IA o caminho de
fora é pior do que o normal, porque é a conta pessoal, no plano gratuito, sem
contrato nenhum e normalmente com a opção de usar o conteúdo para treinamento
ligada por padrão.

O que funciona é escolher e bancar uma ferramenta, com conta corporativa, e
conferir duas configurações que quase ninguém lê:

- **Retenção.** Por quanto tempo o fornecedor guarda o que você manda, e quem
  do lado dele consegue ler.
- **Uso para treinamento.** Se o conteúdo enviado alimenta o modelo. Isso
  costuma ser diferente entre plano gratuito e plano pago do mesmo produto.

Some a regra do que nunca vai no prompt, que é a mesma lista de campos que
nunca saem da seção 2 do capítulo 7. E trate a ferramenta de transcrição com o
mesmo cuidado: reunião sobre cliente é dado pessoal, a gravação fica com o
fornecedor, e ela entra no inventário como qualquer outro.

## 2. Parte do produto

Aqui a chamada ao modelo é uma chamada a serviço externo, e vale tudo do
capítulo 7. Entra no inventário de fornecedores, propaga o identificador de
correlação do [capítulo 2](02-trilha-de-auditoria.md), tem tempo limite,
tem comportamento definido para a falha, e a resposta é guardada.

Três coisas são específicas.

**O prompt é payload.** Se você monta o prompt concatenando o registro do
cliente, você enviou o registro do cliente. A regra de mascarar na origem vale
igual, e o teste automatizado que falha quando um documento aparece no que
seria enviado vale mais ainda aqui, porque o prompt costuma ser montado em
vários lugares do código e cresce sem revisão.

**É transferência para terceiro, e quase sempre internacional.** Precisa
aparecer no mapa do [capítulo 8](08-lgpd-no-backend.md), na base legal daquela
finalidade e na resposta sobre com quem o dado foi compartilhado. O
subprocessador do fornecedor do modelo também está na cadeia.

**A resposta não é reproduzível.** Este é o ponto que eu levaria primeiro para
o time, porque ele contraria o instinto de todo mundo que trabalha com sistema
determinístico. O mesmo prompt não devolve a mesma resposta. Reexecutar depois
não reconstrói o que aconteceu, e guardar só o prompt não serve de evidência.

O que precisa ficar registrado, junto da operação:

- O prompt final que foi enviado, já mascarado.
- A resposta que voltou, inteira.
- O identificador exato da versão do modelo, não o nome comercial.
- Os parâmetros que influenciam a saída.

A versão importa mais do que parece. O fornecedor troca e aposenta versão sem
você pedir. Se a sua trilha guarda só o nome do produto, daqui a um ano você
não consegue nem descrever o que rodou naquela decisão, e essa é a pergunta
que vai chegar.

## 3. Decidindo sobre o cliente

A LGPD dá ao titular o direito de pedir revisão de decisão tomada apenas de
forma automatizada que afete os interesses dele, e de receber informação sobre
os critérios usados. O alcance disso e a forma de responder são conversa com o
seu jurídico. O que cabe à engenharia é deixar isso possível.

Três exigências práticas:

- **Saber quais decisões foram automatizadas.** Uma marca no registro da
  decisão dizendo que ela saiu de um modelo, e qual. Sem isso, quando o pedido
  de revisão chegar, ninguém vai conseguir separar os casos.
- **Guardar o que entrou.** As variáveis que produziram aquela saída, no
  estado em que estavam. Recalcular hoje dá outro resultado, porque o cadastro
  mudou.
- **Ter caminho de revisão humana, e registrá-lo.** Quem revisou, quando, o
  que viu, e se o resultado mudou.

Sobre o último, tem um furo que aparece rápido e vale dizer com todas as
letras. Muita gente resolve a exigência colocando um operador para clicar em
confirmar na tela, e chama aquilo de decisão humana. Se na prática o operador
confirma tudo, porque tem meta de volume e não tem informação para discordar,
a revisão é decorativa. Quem audita olha a taxa de divergência entre a
sugestão do modelo e a decisão final, e uma taxa perto de zero conta a
história sozinha.

Vale também um critério de onde usar. Em ambiente regulado, "o modelo decidiu"
não é uma resposta que se sustenta. Se você não consegue dizer o que pesou
naquela saída, isso é um argumento contra usar modelo naquele ponto
específico, e não um problema para resolver depois com documentação.

## 4. O prompt é o sexto caminho

O [capítulo 5](05-mudanca-em-producao.md) listou cinco caminhos pelos quais
produção muda, e disse que fazendo o inventário com o time costuma aparecer um
sexto que ninguém lembrou. Em sistema que usa modelo, o sexto é o prompt.

Mudar o texto do prompt muda o comportamento do sistema em produção, e na
maioria dos times ele está numa variável de ambiente, num painel ou num campo
de banco, fora do repositório e fora de qualquer revisão. Alguém ajusta uma
frase numa terça e o sistema passa a responder outra coisa, sem commit e sem
aprovação.

Prompt é código. Vive no repositório, passa por revisão de outra pessoa, tem
versão, e a mudança dele entra na lista de mudanças em produção. O mesmo vale
para ligar um recurso de IA por feature flag, que é mudança em produção pelas
regras do capítulo 5.

## O teste

1. Liste as ferramentas de IA em uso no time hoje. Quantas passaram por
   contrato ou por avaliação?
2. Nessas ferramentas, por quanto tempo o fornecedor retém o que vocês mandam,
   e o conteúdo é usado para treinamento? Onde você leu isso?
3. O prompt que o seu sistema monta passa pela mesma máscara que o resto do
   que sai para terceiro? Existe teste que falha se um documento vazar nele?
4. Numa decisão que passou por modelo há três meses, você tem o prompt, a
   resposta, a versão exata do modelo e os parâmetros?
5. Consegue listar quais decisões sobre clientes foram automatizadas no último
   trimestre?
6. Qual é a taxa de divergência entre a sugestão do modelo e a decisão final
   do operador que revisa?
7. Onde está o texto dos prompts de produção, e quem pode alterá-lo sem
   revisão de outra pessoa?

A pergunta 6 é a mais desconfortável da lista, e é a que eu faria primeiro se
estivesse do outro lado da mesa.

## Onde isso continua

Este capítulo não inventa exigência nova. Ele aplica o que os capítulos
anteriores já pediam a um fornecedor que entra pelo caminho mais curto, produz
saída que não se reproduz, e às vezes decide coisa sobre gente. A seção de IA
do [checklist](99-checklist.md) tem o recorte rápido para percorrer com o time.
