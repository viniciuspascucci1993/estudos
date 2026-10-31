# Caderno de fundamentos — Engenharia de Software

Resumos dos estudos de fundamentos, com as armadilhas em que caí e perguntas de entrevista.

**Como revisar:** leia a pergunta, responda em voz alta e só depois abra a resposta (clique em "Resposta").

**Projeto prático ligado:** [ledger-service](https://github.com/viniciuspascucci1993/ledger-service)

---

## 1. Contabilidade de dupla entrada

**Estudado em:** 23/10/2026 · **Fonte:** Modern Treasury, *Accounting for Developers* (Parts I e II)

### Resumo

| Conceito | O que é | Exemplo |
|---|---|---|
| **Conta** | "Balde" que acumula valores de um tipo | Saldo do Cliente A, Caixa, Receita de Tarifas |
| **Lançamento** | O **evento de negócio** registrado | "Tarifa mensal", "PIX de A para B" |
| **Partida** | **Cada linha** do lançamento: conta + débito/crédito + valor | Débito em Saldo do Cliente A, R$ 2 |

- Todo lançamento tem **pelo menos 2 partidas**, e **soma dos débitos = soma dos créditos**.
- Débito e crédito são só **lado esquerdo e lado direito**. Não significam "entrou/saiu" nem "bom/ruim".
- O efeito depende do **tipo da conta**:

| Tipo de conta | Débito | Crédito |
|---|---|---|
| **Ativo** (o que a empresa tem) | aumenta | diminui |
| **Passivo** (o que a empresa deve) | diminui | aumenta |
| **Receita** | diminui | aumenta |

- Motivo: **Ativo = Passivo + Patrimônio**. Cada lado da equação reage de forma oposta.
- **O saldo do cliente é um passivo da empresa**: é dinheiro que ela deve a ele.
- **Saldo é derivado** das partidas, nunca um campo editável. As partidas são a fonte da verdade.
- **Lançamento não se apaga nem se edita**: corrige-se com **estorno** (novo lançamento com as partidas invertidas). O histórico é *append-only*.

### Exemplos

**PIX de R$ 100 de A para B (mesma plataforma):**

| Conta | Tipo | Lançamento |
|---|---|---|
| Saldo do Cliente A | Passivo | Débito R$ 100 (a empresa deve menos a A) |
| Saldo do Cliente B | Passivo | Crédito R$ 100 (a empresa deve mais a B) |

O **Caixa não mexe**: só muda a quem a empresa deve. Por isso PIX interno é instantâneo.

**Tarifa de R$ 2:** Débito em Saldo do Cliente (passivo diminui) · Crédito em Receita de Tarifas (receita aumenta).

### Armadilhas em que caí

- Confundi **lançamento** com o evento que o gera (boleto, empréstimo). O boleto **gera** um lançamento.
- Expliquei débito/crédito por "taxas e dívidas". O que define o efeito é o **tipo da conta**.
- "A plataforma recebe o crédito" → o certo é **"credita a conta de receita"**. Débito e crédito são feitos **em contas**.
- Correção como "marcar como pagamento errado" ainda é **editar**. O certo é **estorno**.

### Perguntas

<details><summary>Por que débitos precisam ser iguais a créditos?</summary>

Dinheiro não surge nem some: sai de um lugar e vai para outro. Débito ≠ crédito significa que o sistema "criou" ou "sumiu" dinheiro, e os livros não fecham.
</details>

<details><summary>Por que o saldo é calculado e não um campo atualizado?</summary>

Um campo à parte pode divergir (bug, concorrência, update que falhou no meio) e não há como provar como chegou ao valor. Calculado das partidas, o saldo é auditável e reconstruível para qualquer data.
</details>

<details><summary>Como se corrige um lançamento errado?</summary>

Com um **estorno**: novo lançamento com as partidas invertidas, ligado ao original. Nada é apagado ou editado.
</details>

---

## 2. Representação monetária

**Estudado em:** 24/10/2026 · **Fonte:** Modern Treasury, *Floats Don't Work for Storing Cents* · **Decisão:** [ADR-001](https://github.com/viniciuspascucci1993/ledger-service/blob/main/docs/adr/001-representacao-monetaria.md)

### Resumo

- **Float/double nunca para dinheiro**: são binários e não representam frações decimais (como 0,10) com exatidão. O erro é estrutural e se acumula.
- `long` em centavos e `BigDecimal` são **ambos exatos**. A escolha entre eles é de **trade-off**, não de precisão.

| Opção | Prós | Contras |
|---|---|---|
| **long em centavos** (escolhida) | Soma/subtração exatas; primitivo, leve, mapeia para BIGINT | Risco de confundir reais com centavos; percentuais exigem arredondamento explícito |
| **BigDecimal** | Multiplicação/divisão com decimais nativas; controle de escala e arredondamento | Objeto: mais lento e mais memória; armadilhas (`new BigDecimal(0.1)`, `equals` compara escala) |

- R$ 12,34 → `1234`. A **moeda** vai em campo separado (ISO 4217, ex.: `BRL`).
- Regras: conversão só **na borda da API**; partida com valor **> 0** (a direção vem do débito/crédito); percentuais arredondados com `HALF_UP`.

### Armadilhas em que caí

- Achei que o artigo recomendava float para finanças. Era o **contraexemplo**.
- Escrevi "long tem mais precisão". Os dois são exatos.
- Escrevi R$ 12,34 → `1,234`. O certo é `1234`: inteiro, sem vírgula.
- "BigDecimal porque é moderno" não é argumento. Stripe e Modern Treasury usam inteiro.

### Perguntas

<details><summary>long em centavos ou BigDecimal: qual é mais preciso?</summary>

Nenhum: os dois são exatos. A diferença está em performance, ergonomia com frações e armadilhas de uso.
</details>

<details><summary>HALF_UP vs HALF_EVEN?</summary>

HALF_UP é o arredondamento "da escola" (0,5 sobe). HALF_EVEN, "do banqueiro", manda o 0,5 para o par mais próximo e evita viés quando se arredonda milhões de vezes.
</details>

---

## 3. TDD e qualidade de testes

**Praticado em:** 24/10 e 30/10/2026 · **No projeto:** regra de equilíbrio do ledger (PR #1)

### Resumo

- Ciclo: **vermelho** (teste falha) → **verde** (implementação mínima) → refatorar.
- Teste de domínio é **JUnit puro**: sem Spring, sem banco.
- Cada teste prova **uma regra**, isolada.
- `assertThrows(ExcecaoEsperada.class, () -> codigo)`: o código que deve falhar fica **dentro** da lambda.
- O **nome** do teste descreve o **comportamento**, não os dados (`deveAceitarLancamentoComMultiplasPartidas`, e não `...Debito100Credito60mais40`).
- `enum` em vez de `boolean`/`String`: torna estados inválidos impossíveis de representar, e o erro vai para o compilador.
- Construtor `private` + métodos de fábrica: a única forma de criar o objeto passa pela validação.

### Armadilhas em que caí

- Teste de valor zero montado com **1 partida só**: passaria por causa da regra de "mínimo 2 partidas", não do valor. **Falso positivo**: o teste precisa falhar e passar pelo motivo certo.
- Criei a partida **fora** do `assertThrows`: a exceção explodiria antes do assert.
- Inventei uma regra ("valor acima de 50") que ninguém pediu. Teste documenta o produto; regra inventada vira mentira.
- Assinatura gerada pela IDE ficou com `@Nullable Object` e `int`. **Revisar o que a IDE gera.**
- Construtor `public` no `Lancamento` furava a validação de equilíbrio.

---

## 4. Transações e concorrência

**Estudado em:** 27/10/2026 · **Fonte:** *DDIA*, cap. 7 · Modern Treasury, *Designing Ledgers with Optimistic Locking*

### Resumo

| ACID | Significa | No ledger |
|---|---|---|
| **A**tomicidade | Tudo ou nada | Débito e crédito gravam juntos, ou nenhum grava |
| **C**onsistência | De um estado válido para outro válido | Constraints nunca violadas |
| **I**solamento | Transações simultâneas não se atrapalham | Dois PIX ao mesmo tempo não corrompem o saldo |
| **D**urabilidade | Commit feito não se perde | Lançamento confirmado não some |

- **Atomicidade é o mecanismo; consistência é a consequência.**
- **Lost update:** duas operações leem o mesmo valor, calculam em cima dele, e a segunda a gravar **sobrescreve** a primeira. A atualização da primeira **se perde**.

| Nível de isolamento | Garante | Observação |
|---|---|---|
| **Read Committed** | Não lê dado não confirmado | **Padrão do Postgres. Não impede lost update** |
| **Repeatable Read** | O que leu não muda na transação | No Postgres, detecta conflito e aborta uma |
| **Serializable** | Igual a rodar uma de cada vez | Mais seguro, mais abortos e retries |

| Lock | Ideia | Como | Bom quando |
|---|---|---|---|
| **Pessimista** | "Vai dar conflito, tranco antes" | `SELECT ... FOR UPDATE` | Muita disputa pela mesma linha |
| **Otimista** | "Conflito é raro, detecto no fim" | Coluna `versao` / `@Version` | Pouca disputa |

- **Hot account** (ex.: Receita de Tarifas recebendo milhares de créditos por segundo): os dois locks vão mal (fila ou tempestade de retries). Solução: **não atualizar linha de saldo**; cada operação é um **INSERT** de partida. Saldo derivado elimina a disputa. Lock só é necessário na conta **debitada**, que precisa checar saldo.

### Armadilhas em que caí

- Respondi "consistência" para as partidas na mesma transação. A letra é **Atomicidade**.
- Descrevi o lost update pela **solução** (versionamento) em vez do **problema**.
- Achei que o Postgres impede lost update no padrão. **Não impede.**
- Escolhi lock pessimista para a hot account. O problema é **uma linha** disputada, e a resposta é modelagem (append-only).

### Perguntas

<details><summary>O que é lost update?</summary>

Duas operações leem o mesmo valor ao mesmo tempo, calculam em cima dele, e a segunda a gravar sobrescreve o resultado da primeira. A atualização da primeira se perde.
</details>

<details><summary>O Postgres no nível padrão impede lost update?</summary>

Não. O padrão é Read Committed. É preciso lock pessimista (`SELECT ... FOR UPDATE`), otimista (versão) ou um nível de isolamento mais alto.
</details>

<details><summary>Como lidar com uma hot account?</summary>

Evitar atualizar uma linha de saldo. Cada operação vira um INSERT de partida (append-only), sem disputa por linha. O saldo é derivado. Conta que só recebe crédito não precisa checar saldo, então não precisa de lock.
</details>

---

## 5. Idempotência

**Estudado em:** 31/10/2026 · **Referências:** Stripe Idempotency-Key · PIX `endToEndId`

### Resumo

- **Retry é inevitável**: o cliente não sabe se a requisição foi processada quando a resposta se perde.
  - Não tentar de novo → arrisca **perder** a operação
  - Tentar de novo sem idempotência → arrisca **duplicar**
- **Idempotente** = executar várias vezes tem o **mesmo efeito** que executar uma vez. É sobre o **estado**, não sobre a resposta.

| Método | Idempotente? |
|---|---|
| `GET`, `PUT`, `DELETE` | Sim |
| `POST` | **Não** (criar lançamento é POST) |

- **Idempotency-Key:** o cliente gera um UUID por operação e o reenvia nos retries. O servidor guarda chave + resposta; chave repetida devolve a resposta guardada sem reexecutar.
- Chave e lançamento **na mesma transação** (atomicidade).
- **Check-then-act** (SELECT + INSERT) falha: as duas requisições fazem o SELECT ao mesmo tempo, antes de qualquer uma inserir, e as duas inserem. Solução: **`UNIQUE` + `INSERT ... ON CONFLICT`**. O banco é o juiz; a perdedora devolve a resposta guardada ou `409` (ainda processando).
- **Mesma chave, corpo diferente** = bug do cliente → **rejeitar** (ex.: `422`). Guardar **hash do corpo** junto com a chave.
- Chaves **expiram** (Stripe: 24h).
- **Kafka** é *at-least-once*: o consumidor guarda os IDs de eventos processados e ignora repetidos.

### Armadilhas em que caí

- Disse que não tentar de novo causaria duplicidade. Causa **perda**.
- Não reconheci o check-then-act como parente do lost update.
- Ia **processar o novo valor** com a mesma chave: o cliente pagaria R$ 250.

### Perguntas

<details><summary>Por que SELECT + INSERT não protege contra requisições simultâneas?</summary>

As duas fazem o SELECT ao mesmo tempo, antes de qualquer uma inserir, e as duas veem que a chave não existe e inserem. Com UNIQUE, o banco só deixa uma inserir; a outra recebe erro de violação.
</details>

<details><summary>Mesma Idempotency-Key com valor diferente: o que fazer?</summary>

Rejeitar explicitamente (ex.: 422). Processar o novo valor duplicaria a operação; devolver a resposta antiga calado enganaria o cliente.
</details>

<details><summary>DELETE retorna 204 e depois 404. Quebra a idempotência?</summary>

Não. Idempotência é sobre o estado final: o recurso está apagado do mesmo jeito.
</details>

---

<!-- Próximo tema: copie a estrutura de uma seção (Resumo / Armadilhas em que caí / Perguntas) -->
