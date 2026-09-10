# Prompt de Teste: Primeiro Contato com Estrutura Minimalista

Como você está terminando a sessão e vai testar como os AI tools reagem à estrutura do Free Wings, aqui estão algumas opções de prompts com **níveis mínimos de contexto**. Escolha o que preferir:

## Nível 1: Apenas a estrutura de pastas (contexto quase nulo)

>Acesse este diretório e me diga o que este projeto parece ser, quais são os arquivos mais importantes e o que você faria em seguida. Apenas observe a estrutura, não tente rodar nada.

**Caminho:** `/caminho/para/free-wings/`

## Nível 2: Estrutura + um arquivo-chave (contexto muito baixo)

>Acesse este projeto. Leia APENAS o primeiro arquivo chamado `FOUNDATION.md` (até a linha 20) e me diga:

1. O que este projeto parece fazer
2. Qual a filosofia parecida que você detecta
3. O que você faria em seguida

**Não leia nem rode o restante dos arquivos.**

**Caminho:** `/caminho/para/free-wings/FOUNDATION.md`

## Nível 3: Estrutura + indication mínima (contexto controlado)

>"Este é um projeto que segue o padrão 'Free Wings'. Acesse e me informe:
>1. Quais arquivos/diretórios chamam sua atenção
>2. Qual a fonte única de verdade que este projeto menciona
>3. O que você espera que aconteça se digitar `/construct` aqui
>4. Quais ferramentas este projeto suporta (ou parece suportar)

**Apenas observe, não execute nada ainda.**

**Caminho:** `/caminho/para/free-wings/`

## Nível 4: O "Pergunta Aberta" Padrão

>"Estou olhando este repositório pelo primeiro vez. Me explique:
>- O que este projeto é
>- O que acontece se eu tentar gerar arquivos de configuração
>- O que parece ser a 'regra de ouro' deste projeto
>- O que eu devo fazer ou não fazer

**Sem qualquer instrução prévia minha sobre este projeto.**

**Caminho:** `/caminho/para/free-wings/`

---

## O Que Você Vai Observar Nestes Testes

Ao testar, preste atenção nestes sinais:

| Sinal | O Que Isso Significa |
|-------|----------------------|
| **AI lê FOUNDATION.md e para lá** | Entendeu a fonte de verdade |
| **AI tenta gerar CLAUDE.md/AGENTS.md imediatamente** | Ainda está no modo "antigo" (pré-Foundation) |
| **AI pergunta "qual tool você usa?"** | Respeitou a filosofia tool-agnostic |
| **AI menciona "hangar/" ou "blueprints"** | Reconheceu a nova estrutura |
| **AI ignora .gitignore e tenta commitar** | Não leu as regras do projeto |
| **AI pergunta "onde está o .claude/?"** | Ainda espera o layout antigo |

---

## Minha Sugestão de Teste

Comece com o **Nível 1** (apenas pastas). É o teste mais puro de "primeiro contato" — o modelo não tem nenhum viés prévio sobre o que aquele projeto é.

Depois, faça o **Nível 3** para ver se o modelo já tem algum conhecimento sobre o padrão Free Wings (provavelmente virá do treinamento dele mesmo).

---

## Depois do Teste

Quando fizer os testes, anote:

1. **Qual prompt usou**
2. **Qual foi a primeira resposta do AI**
3. **O que ele tentou fazer (ou não fez)**
4. **O que mais chamou a atenção**

Isso vai nos dar um mapa de como os tools estão "pensando" sobre a nova estrutura do Free Wings antes mesmo de qualquer explicação nossa.

---

**Boo teste!** Quando voltar, sinta-se à vontade para compartilhar o que cada tool disse. Estou aqui para analisar os resultados (ainda em modo apropriado).

---

Salvo em: `docs/test-prompt-options.md`