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

## 6. Observabilidade

**Estudado em:** 03/11/2026 · **Referências:** OpenTelemetry · Micrometer · Google SRE Book (SLOs e alertas)

### Resumo

- **Monitoramento** responde "tem algo errado?". **Observabilidade** responde "por que, e onde?", inclusive para problemas não previstos.

| Pilar | O que é | Responde |
|---|---|---|
| **Log** | Registro de **um evento específico**, com detalhe | O que aconteceu neste caso? |
| **Métrica** | **Número agregado** no tempo | Quanto e com que frequência? |
| **Trace** | O **caminho de uma requisição** entre serviços, com o tempo de cada etapa | Onde o tempo foi gasto, e onde falhou? |

- Métrica avisa (taxa de erro subiu) → trace localiza (qual serviço) → log detalha (o erro exato).

**Logs:** estruturados (JSON), com **trace ID** em cada linha, **sem dado sensível** (CPF, cartão, senha, token: risco de segurança e LGPD), níveis usados com critério.

| Tipo de métrica | Mede | Exemplo |
|---|---|---|
| **Counter** | Só cresce | Lançamentos criados |
| **Gauge** | Sobe e desce | Conexões no pool, mensagens na fila |
| **Histogram** | Distribuição | Latência p50, p95, **p99** |

- **Percentil, não média:** a média esconde a cauda. 99% em 10ms + 1% em 4s dá média baixa, mas 1 em cada 100 clientes espera 4s.
- **RED:** Rate, Errors, Duration, as três métricas de todo serviço.
- **Cardinalidade:** cada combinação de labels é uma série. Label só com **poucos valores** (`status`, `motivo`). ID específico (`conta_id`) vai no log ou no trace.
- **Trace:** trace = requisição inteira; span = cada etapa. O trace ID viaja no header `traceparent` (W3C) e, no Kafka, nos headers da mensagem. **OpenTelemetry** é o padrão aberto; no Spring Boot, o **Micrometer** faz a ponte.

| Sigla | O que é | Exemplo |
|---|---|---|
| **SLI** | A medição | % de requisições em < 300ms |
| **SLO** | Meta **interna** | 99,9% em < 300ms, por mês |
| **SLA** | **Contrato com o cliente**, com penalidade | 99,5%, ou desconto na fatura |

- SLA mais frouxo que o SLO = **margem de segurança**: o SLO é violado primeiro e o time reage antes da multa.
- **Error budget:** 99,9% permite ~43 min de falha por mês. Sobrou orçamento → features; acabou → estabilidade.
- **Alertar no sintoma, não na causa.** Alerta que acorda alguém precisa significar impacto no cliente e ter ação clara.

### Armadilhas em que caí

- Defini log, métrica e trace de forma genérica e não dei o exemplo do trace: *"este pagamento levou 3s; em qual serviço e etapa?"*
- Disse que o histograma mostra **onde** está a latência. Histograma mostra **quanto e para quantos**; **onde** é trabalho do trace.
- Não expliquei por que média de 50ms convive com reclamação: a **média esconde a cauda**, e é preciso olhar p95/p99.
- No `conta_id` como label, não citei **cardinalidade**, e a pergunta de negócio ("lançamentos por conta") vai para o **banco**.
- Achei que o SLA é mais frouxo "porque tem mais indisponibilidade". É **margem de segurança** antes da penalidade.
- Na pergunta dos alertas, pensei em revisar o SLA. O problema era alertar na **causa** (CPU) e não no **sintoma** (pagamentos falhando).

### Perguntas

<details><summary>Que pergunta só o trace responde bem?</summary>

"Esta requisição demorou 3 segundos: em qual serviço e em qual etapa o tempo foi gasto?" O log mostra cada serviço isolado e a métrica mostra o agregado. Só o trace junta as etapas de uma requisição.
</details>

<details><summary>Latência média de 50ms e clientes reclamando de lentidão. Como é possível?</summary>

A média esconde a cauda: uma pequena parte das requisições pode levar segundos sem mexer muito na média. Olha-se o p95/p99 do histograma, e depois os traces das requisições lentas para achar onde está o tempo.
</details>

<details><summary>Por que não colocar conta_id como label de métrica?</summary>

Cardinalidade: cada valor vira uma série. Um milhão de contas são um milhão de séries, o que deixa o Prometheus lento, consome memória e pode derrubá-lo (e encarece ferramentas pagas). "Quantos lançamentos por conta" é pergunta de negócio, respondida pelo banco.
</details>

<details><summary>SLO vs SLA: qual a diferença, e por que o SLA é mais frouxo?</summary>

SLO é a meta interna; SLA é o contrato com o cliente, com penalidade. O SLA é mais frouxo para dar margem: o SLO é violado primeiro e o time reage antes de chegar à multa.
</details>

<details><summary>Alerta de CPU dispara toda madrugada e é ignorado; uma queda de pagamentos de 20 min não alertou. O que está errado?</summary>

Alertas na causa (CPU) em vez do sintoma (impacto no cliente), gerando fadiga de alerta. Redesenho: alertas de sintoma ligados ao SLO (taxa de erro, p99, burn rate do error budget); CPU vai para o dashboard ou ignora a janela do batch; todo alerta que acorda alguém precisa significar impacto no cliente e ter ação clara.
</details>

---

## 7. Incidente: pool de conexões esgotado

**Praticado em:** 05/11/2026 · Exercício de investigação de incidente (observabilidade na prática)

### Resumo

**O cenário:** alerta de SLO de latência no `POST /pagamentos`, sexta às 20h12.

| Etapa | Sinal | Leitura |
|---|---|---|
| 1. Métricas RED | Rate normal · p50 normal (85ms) · p99 6,2s · erros 2,3% | A **maioria está normal**; uma **fatia de ~2%** está lenta e vira erro. Não é carga |
| 2. Traces lentos vs rápidos | `HikariPool.getConnection` = 5,98s; a query leva 28ms | O banco **não** está lento: a requisição está **na fila esperando conexão** |
| 3. Gauge do pool + logs | Conexões ativas 10/10, pendentes até 30; "Connection is not available"; deploy às 19:50 | **Pool esgotado**, começando logo após um deploy |
| 4. Trace do estorno | `@Transactional` com HTTP de 4,7s dentro | A transação **segura a conexão** enquanto espera a API externa |

- **Gatilho:** API do banco parceiro lenta (fora do nosso controle).
- **Causa raiz:** **chamada HTTP externa dentro de transação de banco**, introduzida no deploy.
- O timeout do pool (6000ms) explica o p99 parado em ~6s e os erros.

**Mitigação imediata:** perguntar **"o que mudou?"** → **rollback** do deploy ou **feature flag**. Primeiro estancar, depois investigar.

**Correção definitiva:**
- Transações **curtas**: commit antes da chamada externa, nova transação depois
- Ou **assíncrono**: outbox + worker com retry e **Idempotency-Key** para o parceiro
- **Timeout** no cliente HTTP, **circuit breaker** e **bulkhead** (limitar a concorrência dos estornos)

### Armadilhas em que caí

- Li o p99 alto como "afeta todo mundo". Com **p50 normal**, a maioria está bem; o problema é a **cauda**.
- Pensei em usar métrica para o tempo de **cada** requisição. Isso é trabalho do **trace**.
- Suspeitei de consultas repetidas e cache. A query estava rápida: o problema era o **tempo que a conexão ficava presa**, não a quantidade de consultas.
- Apontei a API externa como causa raiz. Ela é o **gatilho**; a causa raiz é o **nosso design** (HTTP dentro da transação).
- Propus **aumentar o pool** como mitigação e como correção. Só adia o problema e pode esgotar o `max_connections` do Postgres (pods × pool). Mitigação = rollback/feature flag; correção = tirar o HTTP da transação.

### Perguntas

<details><summary>Rate normal, p50 normal, p99 explodiu e erros subiram. O que isso diz?</summary>

A maioria das requisições está normal e não é problema de carga. Uma fatia pequena está muito lenta, e provavelmente estoura timeout virando erro. É algo específico atingindo parte das requisições.
</details>

<details><summary>O trace mostra getConnection lento e a query rápida. O que está acontecendo?</summary>

O banco não está lento: a requisição espera na fila por uma conexão livre. O pool está esgotado porque algo segura as conexões por muito tempo. Confirma-se com o gauge de conexões ativas/pendentes e os logs de timeout do pool.
</details>

<details><summary>Por que aumentar o pool não resolve?</summary>

Só adia: com mais lentidão ou mais carga, o pool maior esgota igual. E pode estourar o max_connections do banco, já que cada pod tem seu pool. A causa (conexão presa) continua.
</details>

<details><summary>Primeira ação num incidente que começou logo após um deploy?</summary>

Rollback do deploy ou desligar a funcionalidade por feature flag. Primeiro restaurar o serviço, depois investigar a causa com calma.
</details>

<details><summary>Como corrigir uma chamada HTTP externa dentro de uma transação?</summary>

Separar em transações curtas (commit antes da chamada, nova transação depois) ou tornar assíncrono com outbox + worker com retry e Idempotency-Key. Somar timeout no cliente HTTP, circuit breaker e bulkhead.
</details>

---

## 8. Persistência e migrations

**Praticado em:** 06/11 e 08/11/2026 · **No projeto:** ledger-service, semana 2 (branch `feat/persistencia-lancamento`)

### Resumo

**Flyway**
- Arquivo em `src/main/resources/db/migration/`, nome `V1__descricao.sql` (V maiúsculo, versão, **dois** underscores).
- **Migration aplicada não se edita.** O Flyway grava um checksum em `flyway_schema_history`; mudar o arquivo impede a aplicação de subir. Correção = migration nova (`V2__...`). Por isso a revisão vem **antes** de rodar.
- Com `ddl-auto: validate`, o Hibernate confere na subida se as entidades batem com as tabelas. O Flyway manda no schema; o Hibernate só verifica.

**O schema também comunica o negócio**
- `CHECK (valor_em_centavos > 0)` e `CHECK (tipo IN ('DEBITO', 'CREDITO'))` repetem regras do domínio: **defesa em profundidade**. O domínio Java não é o único que escreve na tabela (SQL manual, scripts, outro serviço, bug).
- Sem coluna `atualizado_em`: lançamento é **imutável**; correção é estorno.
- Chave estrangeira no banco (`REFERENCES lancamentos (id)`), mesmo sem relacionamento no JPA.
- Índices em `partidas(conta_id)` (cálculo de saldo) e `partidas(lancamento_id)`. A chave primária já tem índice automático.
- **Moeda** (ADR-001): no **lançamento**, para todas as partidas ficarem na mesma moeda e a regra de equilíbrio continuar valendo. Lacuna ainda não implementada.

**Domínio com identidade**

| Método | Para quê | id e data | Valida? |
|---|---|---|---|
| `criar` | Lançamento **novo** | Gera `UUID.randomUUID()` e `Instant.now()` | Sim |
| `reconstruir` | Lançamento que **já existe** no banco | **Preserva** os valores do banco | Não (já foi validado ao entrar; regra nova não pode quebrar dado antigo) |

- `Instant` é o par natural do `TIMESTAMPTZ`.
- Teste: `assertNotNull` para valor aleatório (criar), `assertEquals` para valor controlado (reconstruir).

**Arquitetura hexagonal**
- O port (`application/port/out/LancamentoRepository`) é uma **interface** que a aplicação define.
- O service **chama** o port; o adapter JPA **implementa** o port. O adapter depende da aplicação, nunca o contrário: **inversão de dependência**.
- Ganhos: testar regra sem banco (repositório falso) e trocar a persistência sem tocar no domínio.
- `Optional` no `buscarPorId` deixa explícito na assinatura que pode não existir, e força quem chama a tratar (ex.: `orElseThrow` → 404). `null` não avisa nada.
- Criar só os pacotes que vão ser usados agora. Pasta vazia não ajuda.

**Entidades JPA**
- Separadas do domínio: o domínio fica sem anotação; o adapter traduz um no outro.
- Construtor vazio `protected` (exigido pelo JPA) e sem setters (entidade imutável; com `@Id` no campo, o JPA acessa os campos direto).
- `@Enumerated(EnumType.STRING)`: grava `"DEBITO"`. Sem isso, grava a posição (0/1) e o `CHECK` recusa.
- `open-in-view: false`: com ele ligado, a conexão fica presa durante a requisição inteira (mesma família do incidente de pool).

**Relacionamentos JPA vs referência por ID**

| | `@OneToMany` / `@ManyToOne` | Referência por ID (UUID) |
|---|---|---|
| Ganho | Conveniência: navega `lancamento.getPartidas()` sem query | Controle: você sabe exatamente que queries rodam |
| Custo | Queries escondidas (N+1), `LazyInitializationException`, cascade com efeito colateral, grafos enormes | Escrever as consultas explicitamente |
| Bom para | CRUDs e domínios pequenos | Sistemas grandes; entre agregados (DDD) |

> "Relacionamentos JPA trocam controle por conveniência. Em domínios pequenos compensa; em sistemas grandes prefiro referência por ID e queries explícitas, porque o comportamento no banco fica previsível."

**Lendo erro de conexão**

| Mensagem | Significa |
|---|---|
| `password authentication failed` | Há um Postgres respondendo, mas recusou usuário/senha |
| `Connection refused` | **Não há Postgres** escutando naquela porta (container parado) |

### Armadilhas em que caí

- Coluna `created_at` num schema todo em português. **Um idioma só** no schema.
- `REFERENCES (lancamentos)`: o certo é `REFERENCES lancamentos (id)` (tabela, depois coluna).
- `CHECK ( IN 'CREDITO' | 'DEBITO')`: faltou a coluna, e o separador é vírgula: `CHECK (tipo IN ('DEBITO', 'CREDITO'))`.
- Misturei o `CHECK` do valor com a moeda (`IN ('BRL', 'USD')`). Valor é número: `CHECK (valor_em_centavos > 0)`.
- Criei o índice de `lancamento_id` na tabela `lancamentos`. A coluna mora em `partidas`.
- Respondi que a falta de `atualizado_em` "é vista pelo índice". O motivo é a **imutabilidade** do lançamento.
- Disse que "o JPA chama o repository". É o **service que chama**; o adapter JPA **implementa**.
- Deixei comentários `// ???` no código já resolvido, mais de uma vez.
- Typo `valorEmXentavos` e setters numa entidade imutável.

### Perguntas

<details><summary>Por que uma migration aplicada não pode ser editada?</summary>

O Flyway grava o checksum de cada migration aplicada. Se o arquivo mudar, o checksum não bate e a aplicação para de subir. Correção se faz com uma migration nova.
</details>

<details><summary>Se o domínio já valida valor > 0, por que um CHECK no banco?</summary>

Defesa em profundidade: o domínio não é o único que escreve na tabela (SQL manual, scripts, outro serviço, bugs). O banco garante a regra para qualquer escritor.
</details>

<details><summary>Por que a tabela de lançamentos não tem atualizado_em?</summary>

Lançamento é imutável: nunca é alterado, só estornado com um novo lançamento. Ter atualizado_em sugeriria que editar é permitido.
</details>

<details><summary>Por que criar e reconstruir são métodos separados?</summary>

Criar gera identidade nova (id e data) e valida. Reconstruir preserva o id e a data do banco. Usar criar na leitura mudaria o id a cada leitura e falsificaria o histórico.
</details>

<details><summary>Quem chama e quem implementa o port do repositório?</summary>

O service da aplicação chama a interface; o adapter JPA a implementa. A dependência aponta do adapter para a aplicação (inversão de dependência), o que permite testar sem banco e trocar a persistência.
</details>

<details><summary>Por que @Enumerated(EnumType.STRING)?</summary>

Sem ele o JPA grava a posição do enum (0, 1). Com STRING grava o nome ("DEBITO"), que é legível, não quebra se a ordem do enum mudar e bate com o CHECK da tabela.
</details>

<details><summary>@OneToMany ou referência por ID?</summary>

Relacionamentos trocam controle por conveniência: risco de N+1, lazy loading e cascades. Em domínios pequenos compensam; em sistemas grandes, referência por ID e queries explícitas deixam o comportamento previsível.
</details>

---

<!-- Próximo tema: copie a estrutura de uma seção (Resumo / Armadilhas em que caí / Perguntas) -->
