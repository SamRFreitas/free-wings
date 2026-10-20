# Prompt — lessons from Shadow Glass for the harness (2026-10-20)

> Temporary file. Delete it once every topic below has been handled
> (folded into the harness, or decided against).

Cole o texto abaixo numa sessão do `the-architect` no Free Wings.

---

Trago do Shadow Glass (`~/Lab/shadow-glass`) quatro assuntos que
apareceram usando o `programmer` durante o degrau 1 da Fase 1: seis
sessões, de 2026-10-06 a 2026-10-20, para um programa pequeno
(`server-windows/src/adapters_test.cpp`). Os quatro são do harness, não
do Shadow Glass. Quero discutir cada um antes de qualquer mudança;
mostre o texto exato antes de escrever, e diga se algum deles é um
Foundation Sync ou merece uma ADR.

Leia primeiro, no Shadow Glass, `docs/specs/` inteira (1 spec e 9
notas) e a seção "Pedagogical approach" do `FOUNDATION.md`. No Free
Wings: `hangar/blueprints/agents/programmer.md`,
`hangar/blueprints/shared/agent-rules.md`,
`hangar/blueprints/agents/deneir.md` e
`docs/decisions/0003-one-home-for-each-fact.md`.

## 1. Não existe lugar para as notas que os agentes produzem

O blueprint diz que o `programmer` escreve código, configuração e
specs em `docs/specs/`. Na prática ele precisou registrar outras coisas
e improvisou arquivos `note-*.md` dentro de `docs/specs/`. Hoje a pasta
tem 1 spec e 9 notas (eram 10; uma foi apagada em 2026-10-20, ver o
assunto 3), de pelo menos seis tipos:

| Tipo | Exemplo | Quem consome |
|---|---|---|
| Material bruto de aprendizado | `note-learning-2026-10-13-pixels-dpi-and-an-experiment.md` (4 arquivos) | o diário (`/write-diary`, `writer`) |
| Recado para outro agente | `note-for-architect-step-1-status-2026-10-07.md` | `the-architect` |
| Proposta de mudança no `FOUNDATION.md` | `note-for-foundation-teaching-approach.md` | `the-architect` |
| Registro de tarefa | `note-todo-fix-commit-author-email.md` (apagada em 2026-10-20) | ninguém, depois de feita |
| Vocabulário do projeto | `note-vocabulary-2026-10-14-desktop.md` | qualquer agente |
| Registro pessoal / plano paralelo | `note-first-cpp-code-2026-10-07.md`, `note-study-plan-algorithms-book.md` | a pessoa |

Decidir: onde cada tipo mora, como se chama, quem o consome. Pontos a
considerar:

- O `deneir` já produz material bruto para o `writer` em
  `docs/observations/`. As notas de aprendizado do `programmer` cumprem
  um papel parecido. É sobreposição, ou são coisas diferentes (o
  `deneir` observa de fora; o `programmer` registra de dentro da aula)?
- O vocabulário pode ser um glossário do projeto (a skill
  `domain-modeling` trabalha com um `GLOSSARY.md`).
- `docs/specs/` deveria voltar a ter só specs.

## 2. Recados entre sessões funcionaram e merecem virar regra

Sessões não compartilham conversa. O que permitiu ao `the-architect`
retomar o trabalho em 2026-10-20, duas semanas depois, sem perder nada,
foi o `note-for-architect-step-1-status-2026-10-07.md`. O que o fez
funcionar:

- a saída real do programa, colada, com o commit em que rodou;
- respostas às perguntas abertas, separando o medido do suposto;
- "o que falta", numerado, fechado item a item;
- onde o trabalho se desviou da spec, e por quê;
- uma seção "para os registros do projeto", com o que o próprio agente
  não pode editar (`FOUNDATION.md`);
- atualizações datadas no fim, em vez de reescrever o texto anterior.

Hoje isso é improviso do `programmer`. Proposta: uma regra em
`agent-rules.md` ("ao parar um trabalho que outra sessão ou agente vai
continuar, deixe um recado no repositório"), com esse formato mínimo e
um lugar definido (ligado ao assunto 1).

## 3. Notas precisam de ciclo de vida, não só de criação

Cada arquivo custa espaço e tokens a todo agente que o lê. Regra
proposta pela pessoa: toda nota existe para atender uma necessidade.
Quando essa necessidade é atendida e registrada no lugar certo (o
`FOUNDATION.md`, o diário, o código), a nota é condensada ou apagada.
Notas que não precisam persistir não persistem.

Casos reais:
- `note-for-foundation-teaching-approach.md`: as regras já foram
  incorporadas ao `FOUNDATION.md` em 2026-10-20; falta só a
  retrospectiva entrar no diário.
- `note-todo-fix-commit-author-email.md`: a tarefa foi feita em
  2026-10-07, e a nota foi apagada em 2026-10-20 — o primeiro caso da
  regra aplicada, feito à mão pelo `the-architect` com aprovação.

Decidir: quem revisa (proposta: o `the-architect`, ao ler os registros
num ponto de checagem, decide incorporar, condensar ou apagar, sempre
mostrando antes); quando revisa; e como isso se liga à ADR 0003 ("one
home for each fact"), da qual é a continuação natural: um fato que já
tem casa não precisa de uma segunda cópia numa nota.

## 4. O blueprint do `programmer` pode descrever melhor como ajudar quem aprende

O blueprint já tem "check understanding" e "let them attempt a piece".
A prática no Shadow Glass foi mais longe, e as regras abaixo falam da
pessoa aprendendo, não do Shadow Glass. Estão no `FOUNDATION.md` do
Shadow Glass, "Pedagogical approach", desde 2026-10-20:

- A pessoa escreve o código primeiro; o `programmer` lê o diff e aponta
  cada problema com o motivo, sem corrigir.
- Antes de explicar, perguntar o que a pessoa já sabe e partir dali.
- Uma ideia por vez. Quando uma aula empilha ideias e a pessoa trava,
  recomeçar em pedaços menores noutro dia (aconteceu duas vezes).
- Uma pergunta deve fazer aplicar a ideia num caso novo; se responde
  lendo a tabela de cima, não ensina nada.
- Analogias do mundo da pessoa só quando são exatas; valores de exemplo
  novos a cada explicação.
- Quando a pessoa propõe ir além do exercício, tratar como um plano
  melhor, não como um desvio (o experimento de DPI de 2026-10-13).
- Cada pedaço roda na máquina real antes do próximo; os pedaços podem
  ser menores do que a spec previu, e o desvio é registrado.
- Sugestões de prompt da ferramenta desligadas durante o estudo.

Pontos a decidir:

- **Tensão com o blueprint atual.** Ele diz que o ensino continua "nos
  comentários do código". Quando quem escreve é a pessoa, o
  `programmer` só comenta quando pedido. O blueprint precisa
  acomodar esse modo, em que o código é da pessoa.
- **Onde moram regras sobre a pessoa.** O blueprint manda as regras do
  projeto para o `FOUNDATION.md` do projeto. Mas estas são sobre quem
  aprende e valeriam no próximo projeto (Swift, por exemplo). Falta
  talvez uma camada "da pessoa", entre o harness e o projeto.
- **Opcional — verificação.** No experimento de DPI, o assistente
  chamou de "verificado" o resultado de duas execuções que diferiam em
  duas coisas ao mesmo tempo. Regra candidata para o `programmer` e o
  `tester`: num experimento, mudar uma coisa de cada vez antes de
  afirmar que algo foi verificado.

---

Verificado: tudo citado acima existe nos arquivos indicados, lido em
2026-10-20. Suposto: que as regras do assunto 4 valem fora do Shadow
Glass. Isso é para a pessoa confirmar.
