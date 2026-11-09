# Caderno de erros — AWS CLF-C02

- **Prova alvo:** fevereiro/2027
- **Meta:** 70%+ nos simulados

**Formato:** `[Domínio — tema] — motivo` / Heads-up / Palavras-chave
**Motivos:** não sabia | confundi serviços | li errado  
_Motivos dos erros de 26/10 a 09/11 estimados na revisão de 09/11; daqui em diante, anotados na hora do erro._
**Domínios:** 1 Conceitos (24%) · 2 Segurança (30%) · 3 Serviços (34%) · 4 Preços e suporte (12%)

---

## Simulado 1 — Skill Builder: Official Practice Question Set (20 questões)

- **Data:** 26/10/2026 · **Tentativa:** 1 · **Nota:** 60% (12/20)
- **Refazer só a partir de:** 23/11/2026

### Q2 · [3 — Serviços de rede] — motivo: confundi serviços

> Uma empresa precisa de uma conexão criptografada entre seus servidores locais e a AWS. A conexão deve usar a internet que a empresa já tem. Qual solução atende?
>
> A) Amazon Connect · B) AWS Direct Connect · C) AWS Site-to-Site VPN · D) _(conferir na página)_

**Resposta:** C

**Heads-up:**
- **Site-to-Site VPN** = túnel **criptografado** passando **pela internet**. Rápido de montar e barato.
- **Direct Connect** = link **dedicado** e privado. **Não** usa a internet e não é criptografado por padrão.

**Palavras-chave:** "internet existente" + "criptografado" → VPN · "dedicado", "latência consistente", "sem internet" → Direct Connect

---

### Q3 · [2 — Segurança / criptografia] — motivo: confundi serviços

> Qual serviço AWS deve ser usado para implementar criptografia em trânsito?

**Resposta:** AWS Certificate Manager (ACM)

**Heads-up:**
- Em **trânsito** = dado viajando pela rede → TLS/SSL → precisa de **certificado** → **ACM**
- Em **repouso** = dado guardado (disco, S3, banco) → precisa de **chaves** → **KMS**

**Palavras-chave:** "in transit", "TLS", "HTTPS", "certificado" → ACM · "at rest", "chaves", "criptografar S3/EBS" → KMS

---

### Q4 · [3 — Armazenamento] — motivo: confundi serviços

> Quais funcionalidades são características do Amazon S3? (escolha 2)
>
> A) Object store · B) Armazenamento durável · C) Sistema de arquivos de rede · D) Sistema de arquivos global · E) Armazenamento local

**Resposta:** A e B _(eu marquei A e D)_

**Heads-up:** S3 = armazenamento de **objetos**, muito **durável** (11 noves: 99,999999999%). É **regional**: só o **nome** do bucket é global.

| Serviço | Tipo | Uso |
|---|---|---|
| **S3** | Objetos | Arquivos via API/URL |
| **EBS** | Bloco | O "HD" de **uma** instância EC2, numa AZ |
| **EFS** | Arquivos (NFS) | Compartilhado entre várias EC2 |

**Palavras-chave:** "objeto", "durável", "11 noves" → S3 · "file system", "compartilhado" → EFS · "disco da EC2" → EBS

---

### Q7 · [4 — Preços / Lambda] — motivo: não sabia

> Como a AWS cobra pelo uso do AWS Lambda depois que o free tier é excedido? (escolha 2)
>
> A) Pelo número de versões da função · B) Pelo total de funções na conta · C) Pelo número de requisições · D) Pelo tempo de execução da função · E) Pela linguagem de programação

**Resposta:** C e D _(eu marquei B e C)_

**Heads-up:** Lambda cobra por **requisições** + **duração** da execução (em GB-segundo: tempo × memória configurada). Ter muitas funções paradas **não custa nada**: sem execução, sem cobrança.

| Serviço | Como cobra |
|---|---|
| **Lambda** | Requisições + tempo de execução (× memória) |
| **EC2** | Tempo em que a instância fica **ligada**, mesmo ociosa |

**Palavras-chave:** "serverless", "paga só quando executa" → Lambda · "número de funções", "versões", "linguagem" → **não** influenciam o preço

---

### Q8 · [2 — Segurança / governança] — motivo: confundi serviços

> Qual serviço AWS identifica security groups que permitem acesso irrestrito aos recursos do usuário?
>
> A) IAM · B) Amazon CloudWatch · C) AWS CloudTrail · D) AWS Trusted Advisor

**Resposta:** D _(eu marquei A)_

**Heads-up:** Trusted Advisor = "consultor" que **verifica a conta** e recomenda boas práticas: custo, segurança, desempenho, tolerância a falhas e limites de serviço. Porta aberta para o mundo (0.0.0.0/0) num security group é um dos checks dele.

| Serviço | Responde à pergunta |
|---|---|
| **IAM** | **Quem pode** fazer o quê? (permissões) |
| **CloudWatch** | **Como está** o desempenho? (métricas, logs, alarmes) |
| **CloudTrail** | **Quem fez** o quê? (auditoria de chamadas de API) |
| **Trusted Advisor** | **O que está fora** das boas práticas? (recomendações) |

**Palavras-chave:** "recomendações", "boas práticas", "acesso irrestrito", "otimizar custo" → Trusted Advisor · "permissões", "usuários", "políticas" → IAM · "auditoria", "quem fez" → CloudTrail · "métricas", "alarme" → CloudWatch

---

### Q9 · [1 — Migração] — motivo: não sabia

> Uma empresa tem um servidor Linux local com um banco Oracle. Ela quer migrar o servidor do banco para rodar numa instância EC2 na AWS. Qual serviço usar?
>
> A) AWS Outposts · B) AWS Application Migration Service (MGN) · C) AWS Schema Conversion Tool (SCT) · D) AWS Database Migration Service (DMS)

**Resposta:** B _(eu marquei A)_

**Heads-up:** Migrar o **servidor inteiro** para EC2, do jeito que está = **lift-and-shift** (rehost) = **MGN**.

| Serviço | O que faz |
|---|---|
| **MGN** | Copia o **servidor inteiro** (SO + apps + banco) para EC2. Lift-and-shift |
| **DMS** | Migra os **dados** de um banco para outro banco (ex.: Oracle local → RDS) |
| **SCT** | **Converte o schema** entre engines diferentes (ex.: Oracle → PostgreSQL). Costuma andar junto com o DMS |
| **Outposts** | O **contrário** de migrar: leva hardware da AWS para **dentro** do seu datacenter |
| **Snow Family** | Migra **grandes volumes de dados** fisicamente, num dispositivo enviado pelo correio |

**Palavras-chave:** "servidor" + "EC2" + "como está" → MGN · "banco para banco", "RDS", "dados" → DMS · "trocar de engine", "converter schema" → SCT · "rodar AWS on-premises" → Outposts

---

### Q11 · [4 — Preços / modelos de compra EC2] — motivo: não sabia

> Qual opção permite comprar capacidade EC2 não utilizada, geralmente com desconto?
>
> A) Dedicated Instances · B) Spot Instances · C) On-Demand Instances · D) Reserved Instances

**Resposta:** B _(eu marquei C)_

**Heads-up:** Spot = **sobra** de capacidade da AWS, com até **90% de desconto**. Em troca, a AWS pode **interromper** a instância (com aviso de 2 minutos) quando precisar da capacidade de volta.

| Modelo | Desconto | Quando usar |
|---|---|---|
| **On-Demand** | Nenhum (preço cheio) | Uso imprevisível, curto prazo, sem compromisso |
| **Spot** | Até ~90% | Tarefas que **aguentam interrupção** (batch, processamento, testes) |
| **Reserved Instances** | Até ~72% | Uso estável e previsível, compromisso de **1 ou 3 anos** |
| **Savings Plans** | Até ~72% | Compromisso de gasto **por hora** (1 ou 3 anos), mais flexível que RI |
| **Dedicated Instances / Hosts** | Nenhum (mais caro) | **Compliance** ou **licença** de software atrelada a hardware físico |

**Palavras-chave:** "não usada", "pode ser interrompida", "tolerante a falhas" + "mais barato" → Spot · "1 ou 3 anos", "previsível" → Reserved/Savings Plans · "sem compromisso", "curto prazo" → On-Demand · "licença", "compliance", "hardware dedicado" → Dedicated

---

### Q20 · [3 — Serviços de rede] — motivo: confundi serviços

> Uma empresa quer estabelecer uma conexão consistente e privada entre o datacenter local e a AWS. Qual serviço atende?
>
> A) Amazon Connect · B) AWS Site-to-Site VPN · C) AWS Client VPN · D) AWS Direct Connect

**Resposta:** D _(eu marquei C)_. Mesmo tema da Q2.

**Heads-up:** "Consistente" + "privada" → Direct Connect: cabo dedicado, a empresa é a única usuária. A VPN passa pela internet, que é compartilhada, então não é nem consistente nem privada (mesmo criptografada).

| Serviço | Conecta | Passa pela internet? |
|---|---|---|
| **Site-to-Site VPN** | Rede/datacenter inteiro ↔ AWS | Sim (criptografado) |
| **Client VPN** | **Pessoas/laptops** ↔ AWS | Sim (criptografado) |
| **Direct Connect** | Datacenter ↔ AWS por **cabo dedicado** | **Não** |
| **Amazon Connect** | Nada de rede: é **call center** na nuvem | — |

**Palavras-chave:** "consistente", "privada", "dedicada", "baixa latência" → Direct Connect · "internet existente", "rápido/barato" → Site-to-Site VPN · "funcionários remotos", "laptops" → Client VPN

---

## Quiz 1 — Aula de rede e migração (8 questões)

- **Data:** 28/10/2026 · **Nota:** 75% (6/8)

### Quiz Q4 · [3 — Rede / VPC] — motivo: confundi serviços

> Instâncias EC2 numa sub-rede privada precisam baixar atualizações da internet, mas não podem receber conexões vindas da internet. O que usar?
>
> A) Internet Gateway na sub-rede privada · B) AWS Direct Connect · C) NAT Gateway · D) VPC peering

**Resposta:** C

**Heads-up:** NAT Gateway = porta que **só abre de dentro para fora**. A sub-rede privada consegue **sair** para a internet (baixar atualização), mas ninguém de fora consegue **entrar**. Ligar a sub-rede direto num Internet Gateway a transformaria em **pública**.

| Recurso | Faz o quê |
|---|---|
| **Internet Gateway** | Porta de **entrada e saída** entre a VPC e a internet. Sub-rede com rota para ele = pública |
| **NAT Gateway** | Só **saída** para a internet, a partir de sub-rede privada |
| **Direct Connect** | Cabo dedicado datacenter ↔ AWS. Não tem nada a ver com internet |
| **VPC peering** | Conecta **duas VPCs** entre si, de forma privada |

**Palavras-chave:** "sub-rede privada" + "acessar a internet" + "sem receber conexões" → NAT Gateway · "tornar público", "acesso da internet" → Internet Gateway · "conectar duas VPCs" → VPC peering

---

### Quiz Q6 · [1 — Economia de nuvem] — motivo: não sabia

> Uma empresa quer pagar apenas pelos recursos que consome, em vez de comprar servidores antecipadamente. Qual vantagem da nuvem descreve isso?
>
> A) Beneficiar-se de economias de escala · B) Trocar despesa fixa por despesa variável · C) Tornar-se global em minutos · D) Aumentar velocidade e agilidade

**Resposta:** B

**Heads-up:** "Comprar antes" = despesa **fixa** (CapEx). "Pagar conforme usa" = despesa **variável** (OpEx). A pergunta descreve exatamente essa troca. Economia de escala é outra coisa: é o preço **por unidade** cair porque a AWS compra em volume gigante.

| Vantagem da nuvem | Pista na pergunta |
|---|---|
| **Trocar despesa fixa por variável** | "pagar só pelo que usa", "sem investimento antecipado" |
| **Economia de escala** | "preços mais baixos" porque a AWS tem milhões de clientes |
| **Parar de adivinhar capacidade** | "escalar conforme a demanda", "sem sobrar nem faltar servidor" |
| **Velocidade e agilidade** | "recursos em minutos", "experimentar rápido" |
| **Parar de gastar com datacenter** | "foco no negócio", "sem cuidar de rack e refrigeração" |
| **Ser global em minutos** | "clientes no mundo todo", "baixa latência global" |

**Palavras-chave:** "pagar pelo que consome", "em vez de comprar antes" → fixa por variável · "preço menor por volume" → economia de escala

---

## Quiz 2 — Aula de segurança e conformidade (8 questões)

- **Data:** 02/11/2026 · **Nota:** 100% (8/8), sem consultar a aula
- Temas: responsabilidade compartilhada (EC2 vs RDS), root user, IAM role, Macie, CloudTrail, Artifact, Secrets Manager

---

## Quiz 3 — Aula de serviços, parte 1 + revisão (10 questões)

- **Data:** 04/11/2026 · **Nota:** 80% (8/10)
- Temas: computação, armazenamento e bancos de dados, com 3 questões de revisão misturadas

### Quiz Q1 · [3 — Computação / escalabilidade] — motivo: li errado

> O tráfego de uma aplicação web varia muito ao longo do dia. A empresa quer adicionar e remover instâncias EC2 automaticamente conforme a demanda. Qual serviço usar?
>
> A) Elastic Load Balancing · B) Amazon EC2 Auto Scaling · C) AWS Lambda · D) Amazon Lightsail

**Resposta:** B _(eu marquei A, sabendo a resposta)_

**Heads-up:** **Auto Scaling** muda a **quantidade** de instâncias. **Load Balancer** **distribui** o tráfego entre as instâncias que já existem. Os dois costumam trabalhar juntos, mas só o Auto Scaling cria e remove.

| Serviço | Faz o quê |
|---|---|
| **Auto Scaling** | Adiciona e remove instâncias conforme a demanda |
| **Elastic Load Balancing** | Distribui requisições entre as instâncias (ALB = HTTP, NLB = TCP) |

**Palavras-chave:** "adicionar/remover instâncias", "conforme a demanda", "escalar automaticamente" → Auto Scaling · "distribuir tráfego", "alta disponibilidade entre instâncias" → Load Balancer

---

### Quiz Q10 · [3 — Rede / VPC] — motivo: confundi serviços · ⚠️ 2º erro neste tema (ver Quiz 1, Q4)

> Instâncias numa sub-rede privada precisam acessar a internet para baixar pacotes, sem ficarem acessíveis a partir da internet. O que usar?
>
> A) Internet Gateway associado à sub-rede privada · B) NAT Gateway · C) AWS Direct Connect · D) Security Group liberando 0.0.0.0/0

**Resposta:** B _(eu marquei C)_

**Heads-up:** O **NAT Gateway é o porteiro que só deixa sair**: as instâncias privadas vão à internet buscar pacotes, mas ninguém de fora entra. O **Direct Connect** é um **túnel particular entre dois prédios** (a empresa e a AWS) e não tem nada a ver com acessar a internet.

| Serviço | Conecta o quê | Direção |
|---|---|---|
| **NAT Gateway** | Sub-rede privada → internet | Só saída |
| **Internet Gateway** | VPC ↔ internet | Entrada e saída (torna a sub-rede pública) |
| **Direct Connect** | Datacenter da empresa → AWS | Cabo dedicado, sem internet |

**Palavras-chave:** "sub-rede privada" + "acessar a internet" + "sem receber conexões" → NAT Gateway · "datacenter" + "dedicado/privado" → Direct Connect

---

## Quiz 4 — Aula de serviços, parte 2 + revisão (10 questões)

- **Data:** 09/11/2026 · **Nota:** 70% (7/10)
- Acertei as 7 de conteúdo novo (CloudFront, SQS, SNS, Athena, Kinesis, CloudFormation, Textract). Os 3 erros foram de **arquitetura**: como as peças se encaixam.
- Depois, no exercício de recordação (explicar o caminho da requisição sem consulta), acertei tudo.

**A imagem que liga os três erros:**

```
                     Usuários (internet)
                            │
                     ┌──────▼───────┐
                     │ Load Balancer│  ← distribui as requisições
                     └──┬────────┬──┘
           ┌────────────┘        └────────────┐
   ┌───────▼────────┐                 ┌───────▼────────┐
   │     AZ  A      │                 │     AZ  B      │  ← duas AZs: se uma cai, a outra segue
   │   EC2   EC2    │                 │   EC2   EC2    │  ← Auto Scaling decide quantas EC2 existem
   │ sub-rede privada                 │ sub-rede privada
   └───────┬────────┘                 └───────┬────────┘
           └──────────►  NAT Gateway  ◄───────┘          ← EC2 privadas SAEM para updates; ninguém ENTRA
                             │
                         internet
```

**Dica de prova:** em questão de rede ou disponibilidade, montar esta imagem mentalmente e ver em que ponto do caminho a pergunta está.

### Quiz Q8 · [3 — Infraestrutura global / disponibilidade] — motivo: não sabia

> Uma aplicação precisa continuar funcionando mesmo se um datacenter inteiro da região falhar. Qual é a prática recomendada?
>
> A) Usar uma instância EC2 maior · B) Implantar em várias zonas de disponibilidade · C) Usar edge locations do CloudFront · D) Ativar o AWS Shield Advanced

**Resposta:** B

**Heads-up:** Cada **AZ** é um ou mais datacenters isolados dentro da região. Rodar em **várias AZs** = alta disponibilidade. Instância maior não ajuda se o datacenter inteiro cair.

| Pergunta fala em | Resposta |
|---|---|
| "falha de um datacenter", "alta disponibilidade" | **Várias AZs** |
| "desastre regional", "perda de uma região inteira" | **Várias regiões** |
| "baixa latência para usuários no mundo" | **Edge locations / CloudFront** |

**Palavras-chave:** "datacenter falhar", "alta disponibilidade", "tolerância a falhas" → multi-AZ

---

### Quiz Q9 · [3 — Computação / balanceamento] — motivo: confundi serviços · ⚠️ inverso do Quiz 3, Q1

> Várias instâncias EC2 rodam a mesma aplicação e a empresa quer distribuir as requisições HTTP dos usuários entre elas. Qual serviço?
>
> A) Amazon EC2 Auto Scaling · B) Application Load Balancer · C) Amazon Route 53 · D) AWS Global Accelerator

**Resposta:** B

**Heads-up:** **Load Balancer distribui** entre as instâncias que já existem. **Auto Scaling dimensiona** (muda quantas existem). Os dois trabalham juntos, mas fazem coisas diferentes.

**Palavras-chave:** "distribuir requisições/tráfego entre instâncias" → Load Balancer · "adicionar/remover instâncias conforme a demanda" → Auto Scaling

---

### Quiz Q10 · [3 — Rede / VPC] — motivo: confundi serviços · ⚠️ 3º erro neste tema (ver Quiz 1 Q4 e Quiz 3 Q10)

> Servidores de banco numa sub-rede privada precisam baixar atualizações de segurança da internet, mas nenhuma conexão vinda da internet pode alcançá-los. O que usar?
>
> A) AWS Direct Connect · B) Internet Gateway na sub-rede privada · C) NAT Gateway · D) AWS Site-to-Site VPN

**Resposta:** C

**Heads-up:** O NAT só existe **porque** as instâncias estão em sub-rede **privada**: ele é o porteiro que deixa **sair** e não deixa **entrar**. Direct Connect e Site-to-Site VPN ligam o **datacenter da empresa** à AWS, não têm nada a ver com acessar a internet.

**Palavras-chave:** "sub-rede privada" + "acessar a internet / baixar atualizações" + "sem conexões de entrada" → NAT Gateway

---

<!-- Próximas: copie o bloco de uma questão acima e preencha -->
