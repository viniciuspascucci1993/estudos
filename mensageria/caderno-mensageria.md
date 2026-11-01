# Caderno de mensageria

Kafka, RabbitMQ e mensageria em sistemas distribuídos: garantias, cenários de falha e armadilhas.

**Como revisar:** leia a pergunta, responda em voz alta e só depois abra a resposta (clique nela).

**Contexto prático:** ironvault-notifications (estudo antes de implementar)

---

## 1. Fila vs log

**Estudado em:** 01/11/2026

### Resumo

| | **Fila** (RabbitMQ, SQS) | **Log** (Kafka) |
|---|---|---|
| Analogia | Caixa de correio: a carta sai quando alguém lê | Diário: as páginas ficam, cada leitor marca onde parou |
| Depois de consumida | É **removida** (após o ack) | **Continua** no log pelo período de retenção |
| Vários consumidores | Cada um precisa da sua fila | Todos leem o mesmo log, cada um com seu **offset** |
| Reprocessar o passado | Não dá | Dá: voltar o offset (**replay**) |
| Ponto forte | Distribuir **tarefas**, roteamento, retry por mensagem | **Stream de eventos**, alto volume, histórico |

| Broker | Modelo | Quando aparece |
|---|---|---|
| **Kafka** | Log distribuído | Eventos de domínio, alto volume, vários consumidores, replay |
| **RabbitMQ** | Fila com roteamento (exchanges) | Filas de trabalho, roteamento flexível, retry e DLQ |
| **SQS / SNS** | Fila / pub-sub gerenciados (AWS) | Zero operação. Cai na CLF-C02: SQS = desacoplar serviços |
| Outros | Redis Streams, NATS JetStream, Pulsar | Leveza, latência baixíssima, multi-tenant |

- O log do Kafka é **append-only**, a mesma ideia das partidas do ledger.

### Armadilhas em que caí

- Disse que no RabbitMQ a mensagem "se perde". Ela é **removida** depois do ack: foi entregue, não perdida.
- Não dei o caso em que o replay importa: **corrigir um bug e reprocessar** o período afetado, ou um **serviço novo ler o histórico**.

### Perguntas

<details><summary>Em que situação o replay do Kafka faz diferença?</summary>

Um bug no consumidor processou eventos errado por 2 horas: corrige-se e volta-se o offset para reprocessar. Ou um serviço novo precisa ler o histórico de eventos desde o início. Com RabbitMQ as mensagens já foram removidas.
</details>

---

## 2. Garantias de entrega

**Estudado em:** 01/11/2026

### Resumo

| Semântica | Significa | Risco |
|---|---|---|
| **At-most-once** | No máximo uma vez | Pode **perder** |
| **At-least-once** | Pelo menos uma vez | Pode **duplicar** |
| **Exactly-once** | Exatamente uma vez | Só existe dentro de limites específicos |

- Regra do mundo real: **at-least-once + consumidor idempotente**.
- O exactly-once do Kafka vale **dentro do Kafka** (ler de um tópico e gravar em outro). Quando o efeito sai dele (e-mail, banco), a duplicidade volta a ser problema da aplicação.

### Perguntas

<details><summary>"O Kafka tem exactly-once, então não preciso de idempotência." Está certo?</summary>

Não. O exactly-once cobre leitura e escrita dentro do Kafka. Efeitos externos (enviar e-mail, gravar no banco) continuam at-least-once e exigem consumidor idempotente.
</details>

---

## 3. Kafka em cenários extremos

**Estudado em:** 01/11/2026

### Resumo

- **Tópico** → **partições** → cada partição **replicada** em vários brokers. Uma réplica é a **líder** (recebe escritas); as outras seguem. As que estão em dia formam o **ISR**.

| Onde | Config | Por quê |
|---|---|---|
| Tópico | `replication.factor=3` | Cada mensagem em 3 brokers |
| Tópico | `min.insync.replicas=2` | Só aceita escrita com 2+ réplicas confirmando |
| Producer | `acks=all` | "Ok" só depois de as réplicas do ISR gravarem |
| Producer | `enable.idempotence=true` | Retry do producer não duplica no tópico (padrão desde o 3.0) |
| Broker | `unclean.leader.election.enable=false` | Réplica atrasada nunca vira líder |

| Cenário | Com `acks=1` | Com a config acima |
|---|---|---|
| Líder morre logo depois de confirmar | **Perde** | Não perde |
| 1 de 3 brokers cai | Segue | Segue |
| 2 de 3 brokers caem | Aceita sem cópia nenhuma | **Recusa escritas**: indisponível, mas sem perda |
| Só sobra réplica atrasada | Pode virar líder e apagar dados | Partição fica fora até voltar uma réplica em dia |

- Em falha grave, Kafka bem configurado **prefere parar a perder dados**.

**Consumo com vários pods:**
- Mesmo `group.id` = **consumer group**. Cada partição é lida por **um único** pod do grupo.
- **Mais pods que partições = pods ociosos** (ficam de reserva para o rebalance). Para escalar, **aumentar partições**.
- **Ordem só dentro da partição** → usar **chave** (ex.: `merchantId`) para manter os eventos de uma entidade em ordem.
- Pod morre → **rebalance** → outro pod continua do último offset **commitado**.

| Commitar o offset | Se o pod cair no meio |
|---|---|
| **Antes** de processar | Mensagem **perdida** (at-most-once) |
| **Depois** de processar | Mensagem **reprocessada** (at-least-once) → consumidor idempotente |

- Sistema crítico: **auto-commit desligado**, commit **depois** de processar.

### Armadilhas em que caí

- Achei que 6 pods para 4 partições "encavalaria". Na verdade **4 trabalham e 2 ficam ociosos**.
- Expliquei o `acks=1` com a definição do `acks=all`. Com `acks=1` só a **líder** confirma, **antes** de replicar: se ela morrer nesse instante, a mensagem confirmada some.
- Achei que o pod caindo entre o envio e o commit mandava a mensagem para a **DLQ**. DLQ é para falha repetida. Aqui ocorre **reprocessamento** pelo rebalance.

### Perguntas

<details><summary>Tópico com 4 partições e 6 pods no mesmo consumer group: o que acontece?</summary>

4 pods recebem uma partição cada; 2 ficam ociosos, de reserva. Para mais paralelismo, aumentam-se as partições.
</details>

<details><summary>Como uma mensagem confirmada com acks=1 pode ser perdida?</summary>

A líder grava e confirma antes de as seguidoras copiarem. Se ela morrer nesse instante, uma seguidora sem a mensagem vira líder e a mensagem some.
</details>

<details><summary>O pod cai depois de enviar o e-mail e antes de commitar o offset. O que acontece?</summary>

Rebalance: outro pod assume a partição a partir do último offset commitado e reprocessa a mensagem, enviando outro e-mail. Evita-se guardando o ID do evento numa tabela com UNIQUE (consumidor idempotente) e, se o provedor permitir, usando idempotency key na API de e-mail.
</details>

<details><summary>Por que o Kafka bem configurado recusa escritas quando 2 de 3 brokers caem?</summary>

Por causa do min.insync.replicas=2: sem duas réplicas para confirmar, ele prefere ficar indisponível a aceitar uma escrita que poderia ser perdida.
</details>

---

## 4. RabbitMQ em cenários extremos

**Estudado em:** 01/11/2026

### Resumo

| Proteção | O que faz |
|---|---|
| **Fila durável + mensagem persistente** | Sobrevive ao restart do broker |
| **Publisher confirms** | O producer sabe que o broker gravou (equivalente ao `acks`) |
| **Ack manual** | Consumidor morreu sem ack → mensagem volta para a fila |
| **Quorum queues** | Fila replicada com consenso (Raft): alta disponibilidade moderna |
| **DLQ** (dead letter exchange) | Mensagem que falha N vezes vai para outra fila |

- **Mensagem venenosa:** sempre falha (ex.: e-mail com formato inválido) e voltaria para a fila para sempre, travando as outras. A DLQ tira ela do caminho.

### Perguntas

<details><summary>Para que serve uma DLQ?</summary>

Para isolar mensagens que falham repetidamente (mensagens venenosas), evitando que voltem à fila para sempre e bloqueiem as demais. Elas ficam numa fila separada para análise.
</details>

---

## 5. Dual write e padrão outbox

**Estudado em:** 01/11/2026

### Resumo

- **Dual write:** gravar no banco e publicar no broker são duas operações separadas. Se o pod cair entre elas, o dado existe e o evento nunca sai.

```
1. grava o pagamento no banco   ✔
2. publica o evento no Kafka    ✖ (pod caiu)
```

- `acks=all` não ajuda: a mensagem **nunca chegou** ao Kafka.
- **Retry não resolve:** roda dentro do pod e morre com ele.
- **Outbox:**

```
Mesma transação:
  INSERT pagamento
  INSERT outbox (evento)          ← atomicidade

Processo separado (relay):
  lê a outbox → publica → marca como enviado
```

- O retry passa a sair de uma **tabela durável**. O relay pode publicar duas vezes → consumidor **idempotente**.
- Alternativa ao relay: **CDC** (ex.: Debezium), que lê o log do banco e publica.
- Estrutura que se repete: **atomicidade** grava junto → **at-least-once** entrega → **idempotência** absorve a duplicata.

### Armadilhas em que caí

- Propus **retry** para resolver o dual write. O pod que faria o retry é o que morreu.

### Perguntas

<details><summary>O payments grava o pagamento e publica no Kafka (acks=all, RF=3), mas às vezes o lojista não é notificado. Causa e solução?</summary>

Dual write: o pod cai entre gravar no banco e publicar, então o evento nunca chega ao Kafka. Solução: outbox. O evento é gravado numa tabela na mesma transação do pagamento, e um processo separado publica depois. Consumidor idempotente para absorver duplicatas.
</details>

<details><summary>Por que o retry não resolve o dual write?</summary>

O retry roda dentro do pod e morre junto com ele. O outbox grava o evento numa tabela na mesma transação, então ele sobrevive à queda, e um processo separado publica.
</details>

---

## 6. Quando usar cada um

**Estudado em:** 01/11/2026

### Resumo

- **Consistência não vem da quantidade de brokers**, e sim de outbox, acks, commit depois de processar e consumidor idempotente. Cada broker a mais é mais operação, monitoramento e custo.
- **Kafka** para **eventos de domínio** (ex.: "pagamento aprovado"), consumidos por vários serviços e com replay.
- **RabbitMQ** como **fila de trabalho** (ex.: dentro do notifications, roteando para e-mail/SMS/push, com retry e DLQ por canal).
- A combinação é comum, mas deve nascer de uma **necessidade**, não ser montada de antemão.

### Armadilhas em que caí

- Pensei em usar Kafka **e** RabbitMQ "para ter mais consistência". Dois brokers mal configurados perdem mensagem do mesmo jeito.

---

<!-- Próximo tema: copie a estrutura de uma seção (Resumo / Armadilhas em que caí / Perguntas) -->
