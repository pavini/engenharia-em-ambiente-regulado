# 4. Segregação de acesso

A pergunta sobre acesso sempre chegou depois da pergunta sobre trilha, e sempre
foi mais difícil de responder.

A trilha até que estava razoável. Tinha quem, tinha quando, tinha o valor
anterior. O problema apareceu quando alguém olhou a coluna de autor e viu que
boa parte dos registros apontava para um usuário de sistema, e que a senha desse
usuário estava num arquivo de configuração que meia dúzia de pessoas conseguia
ler. A partir dali, aquela trilha inteira parou de provar qualquer coisa sobre
autoria.

Segregação de acesso é o que sustenta o [capítulo 2](02-trilha-de-auditoria.md).
Trilha sem acesso nominal registra que alguém fez, e alguém não é resposta.

Este capítulo trata das três superfícies onde isso se decide em código: quem
entra na infraestrutura, quem enxerga dado de cliente dentro da aplicação, e
quem pode aprovar aquilo que outro executa. A parte que é política de empresa
eu comento no fim.

## 1. Infraestrutura: três pontos, e o terceiro é o que falta

Sobre acesso a produção já existe material bom e não vou repetir tutorial. O
recorte que interessa aqui é o que a auditoria pergunta e o que o sistema
precisa ter guardado para responder.

**Nominal em tudo que toca produção.** Banco, console de nuvem, pipeline,
máquina. Conta compartilhada transforma qualquer registro posterior em registro
de grupo. Se hoje existe uma conta assim no seu ambiente, esse é o primeiro
item, porque todo o resto depende dele.

**Conta de serviço também tem dono.** Ela não vai ser nominal no sentido de
pessoa, mas precisa ter um responsável humano registrado, um motivo escrito de
por que existe e um escopo do que ela pode fazer. Credencial que ninguém sabe
de quem é continua válida por anos, sobrevive ao desligamento da pessoa que
criou e é o caminho mais curto para o acesso que ninguém revoga.

**Permissão precisa de histórico, não só de estado.** Este é o ponto que
costuma faltar. A tabela de permissão da maioria dos sistemas é atualizada no
lugar: alguém ganha acesso, a linha muda, alguém perde acesso, a linha some. Ela
responde quem tem acesso hoje.

A pergunta que chega é outra. Quem tinha acesso ao banco de produção em março,
quem concedeu esse acesso, quando, e com qual justificativa. Se a sua resposta
sai de uma consulta na tabela de hoje, você respondeu uma pergunta parecida e
diferente.

Concessão e revogação de permissão são mudança de estado, e valem a mesma regra
do capítulo 2: registre como evento append-only, com quem concedeu, para quem,
o que, quando e por quê. É uma tabela a mais e resolve de uma vez a pergunta
histórica, a revisão periódica e a prova de revogação.

## 2. Quem enxerga dado de cliente

O suporte abre a conta de um cliente para entender uma reclamação. O
back-office consulta um cadastro para conferir um documento. Isso é acesso
legítimo, é rotina, e é onde a auditoria mais gasta tempo, porque é volume e
porque é o lugar onde o vazamento costuma nascer.

**Leitura de dado de cliente é ação registrável.** A maioria dos sistemas
registra escrita e ignora leitura. Aqui a leitura precisa entrar na trilha, com
quem consultou, qual cliente, quando e por onde. Se alguém perguntar se um
funcionário andou olhando a conta de um conhecido, ou o sistema responde, ou a
empresa vai ter que responder que não sabe.

**Justificativa é texto que a pessoa escreveu.** Já vi campo de justificativa
resolvido com uma lista de três opções, e o resultado previsível: quase
todo registro com "consulta de rotina" selecionado. Isso ocupa espaço em
disco e não responde nada. Texto livre obrigatório dá trabalho para quem
consulta, e esse trabalho é parte do controle.

**A tela decide o que você vai conseguir provar.** Uma tela que lista todos os
clientes e deixa clicar é diferente de uma tela que exige o identificador do
cliente antes de mostrar qualquer coisa. Na segunda, toda consulta tem alvo e
tem intenção declarada. Na primeira, não existe como separar trabalho de
curiosidade depois. Isso parece detalhe de interface e é a decisão que mais
muda a sua capacidade de resposta.

**Exportação é a saída silenciosa.** A pessoa filtra uma lista, clica em
exportar, e leva quarenta mil linhas para a máquina dela. Se a trilha registrou
"consultou a lista", você perdeu o evento que importava. Registre exportação
como ação própria, com o filtro aplicado, a contagem de registros e as colunas
levadas. Vale ter alçada aqui também: acima de um volume, a exportação pede
aprovação de outra pessoa.

## 3. Quem aprova e quem executa

Isso chega no time como regra de negócio e vira regra de autorização no
servidor. Quem cadastra o fornecedor não pode ser quem libera o pagamento. Quem
solicita o estorno não pode ser quem confirma. A versão do mesmo princípio
dentro da engenharia: quem abre a alteração não aprova a própria alteração, e
quem escreve o script não é quem roda o script em produção.

Três coisas quebram na prática.

**A regra que existe só no processo.** Está no manual, o time sabe, e o sistema
permite. Quando o auditor perguntar como você garante, a resposta vai ser que a
equipe tem o cuidado de não fazer. Isso não passa, e não deveria passar. Se a
segregação importa, o servidor recusa a segunda ação quando o autor é o mesmo
da primeira.

**A regra implementada na interface.** O botão some para quem não pode. O
auditor mais chato vai chamar a rota direto, e em ambiente regulado ele faz
isso. A decisão precisa estar no ponto que executa, com o teste automatizado
que prova que ela está lá.

**A alçada que o próprio aprovador edita.** Este é o furo mais bonito e o mais
comum. O sistema impede que a mesma pessoa aprove o que cadastrou, e o perfil
de administrador consegue alterar a tabela de limites e a lista de aprovadores.
Se alterar a regra é mais fácil do que burlar a regra, a segregação é
decorativa. Mudança em alçada e em perfil precisa da mesma trilha, e de
preferência da mesma segregação, do que ela controla.

## 4. Acesso de emergência

Toda a régua acima encontra a madrugada de incidente. O serviço caiu, alguém
precisa entrar no banco agora, e a pessoa de plantão não tem esse acesso no dia
a dia, justamente porque você fez a lição de casa.

Existem duas saídas ruins e as duas eu já vi. Uma é não existir caminho
nenhum, e aí a pessoa usa a credencial da aplicação, que ela consegue ler no
arquivo de configuração, e essa credencial fica no histórico do terminal dela
para sempre. A outra é existir um acesso permanente guardado "para emergência",
que em três meses vira o acesso que todo mundo usa porque é mais rápido.

O desenho que funciona tem cinco partes:

- A credencial existe e está separada, com escopo amplo de propósito.
- O uso não depende de aprovação prévia. Se depender, às três da manhã ninguém
  vai acordar o aprovador, e o caminho errado será usado de novo.
- O uso dispara alerta no momento, para mais de uma pessoa, e pelo menos uma
  delas não pode ser quem usou.
- A sessão é gravada. Comando executado, não apenas "fulano acessou".
- A credencial morre no uso. Rotação automática depois de cada acionamento. Sem
  isso, quebrar o vidro vira ter a chave.

Some a revisão obrigatória depois, com prazo curto, e o registro dessa revisão
guardado junto do incidente. Uso de acesso de emergência sem revisão registrada
é achado de auditoria quase garantido.

A frequência também diz alguma coisa. Acesso de emergência acionado uma vez por
semestre é o desenho funcionando. Acionado toda semana significa que o acesso
normal de alguém está errado, e vale olhar quem aciona e para quê antes que a
auditoria olhe.

## 5. Revogação, e o acesso que sobra

O acesso principal costuma cair no mesmo dia do desligamento, porque está no
login central e o RH avisa. O que sobra é o resto: a chave de API que a pessoa
gerou para um teste, o usuário de banco que ela criou, o acesso ao repositório
de um fornecedor, o grupo no provedor de nuvem que foi concedido fora do
processo, a integração que ainda usa o token dela.

Duas coisas resolvem a maior parte disso. A primeira é ter inventário de onde
existe acesso, porque sem lista a revogação é feita de memória e a memória
esquece o sistema secundário. A segunda é o histórico de concessão da seção 1:
se toda concessão virou evento, a lista do que revogar é uma consulta, e a
prova de que foi revogado é outra.

Vale a pena testar isso com um desligamento real dos últimos meses antes que
alguém de fora peça.

## O que vem do outro lado

Matriz de perfil aprovada em comitê, política de acesso assinada, treinamento
anual, termo de confidencialidade. Isso nasce em compliance, jurídico e RH, e
não é trabalho de engenharia discutir o conteúdo.

O que sobra para o time é fazer o sistema produzir a evidência sem ninguém
montar planilha na mão na véspera. Quando a revisão de acesso sai de uma
consulta e vem com data, autor e resultado, essa parte da auditoria deixa de
consumir semanas do time.

## A outra escolha

**Justificativa em toda consulta ou só fora do padrão.** Em operação de volume
alto, exigir texto em cada consulta produz preenchimento automático e ruído que
ninguém lê. A alternativa é exigir justificativa em acesso fora do padrão, como
conta sem chamado vinculado ou volume acima do normal, e alertar sobre o resto.
Ganha quando o suporte atende milhares de casos por dia. Perde quando o volume
é baixo, porque aí o texto obrigatório é barato e vale mais.

**Credencial selada ou elevação temporária aprovada.** A credencial que se
quebra o lacre funciona quando o plantão é de uma pessoa. Se o plantão sempre
tem duas, a elevação aprovada pelo outro plantonista é melhor: mesmo efeito,
com segregação preservada no momento mais sensível.

**Segregação no código ou em motor de fluxo.** Implementar a regra no serviço é
direto e fica junto do que ela protege. Um motor de fluxo centraliza e facilita
auditar, ao custo de mais uma peça na arquitetura e de a regra viver longe do
código. Com poucas regras, no serviço. Com dezenas, e alçada que muda por
produto, o motor começa a compensar.

## O teste

Mesma lógica do capítulo 2. Pegue casos reais e tente responder sozinho, só com
consulta ao sistema:

1. Quem tinha acesso ao banco de produção numa data de seis meses atrás, quem
   concedeu e com qual justificativa?
2. Um funcionário consultou o cadastro de um cliente específico no último ano?
   Quantas vezes, e o que ele escreveu como motivo?
3. Alguém exportou mais de mil registros de cliente no último trimestre? Quem,
   quando, com qual filtro?
4. A regra de que quem cadastra não aprova está implementada onde? Existe teste
   automatizado que falha se ela for removida?
5. Quando foi o último acesso de emergência, quem usou, o que executou, e onde
   está a revisão posterior?
6. Pegue um desligamento dos últimos seis meses e mostre a data de revogação em
   cada sistema onde a pessoa tinha acesso.

A pergunta 6 é a que costuma travar. Ela não trava por falta de controle, trava
por falta de inventário.

## Onde isso continua

O acesso a produção encosta direto em mudança em produção, que é o próximo
capítulo: quem faz deploy, quem aprova, e o que acontece quando alguém precisa
rodar um script no banco. Se o seu sistema já está em operação e você quer o
recorte rápido, a seção de acesso do [checklist](99-checklist.md) resume os
itens que dá para verificar em uma tarde.
