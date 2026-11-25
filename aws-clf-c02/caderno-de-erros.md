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

## Quiz 5 — Aula de faturamento e suporte + revisão (10 questões)

- **Data:** 11/11/2026 · **Nota:** 80% (8/10)
- Acertei as 2 de revisão, incluindo o **NAT Gateway** (depois de 3 erros) e Inspector vs GuardDuty.
- Com esta aula, os **4 domínios** foram cobertos.

### Quiz Q1 · [4 — Faturamento / várias contas] — motivo: li errado

> Uma empresa tem 12 contas AWS e quer receber uma única fatura, aproveitando os descontos por volume somados de todas as contas. O que usar?
>
> A) AWS Budgets · B) Faturamento consolidado do AWS Organizations · C) AWS Cost Explorer · D) AWS Control Tower

**Resposta:** B _(eu marquei D, sabendo a resposta)_

**Heads-up:** Os dois são de "várias contas", mas fazem coisas diferentes. O **Organizations** gerencia as contas, e o **faturamento consolidado** é recurso dele (uma fatura, descontos somados). O **Control Tower** **monta** um ambiente multi-conta já seguindo boas práticas.

| Serviço | Pista |
|---|---|
| **Organizations** | "várias contas", "uma fatura", "descontos somados", "SCP" |
| **Control Tower** | "configurar/montar ambiente multi-conta com boas práticas" (landing zone) |

**Palavras-chave:** "fatura única", "consolidado", "descontos por volume entre contas" → Organizations

---

### Quiz Q5 · [4 — Planos de suporte] — motivo: li errado

> Uma empresa precisa de suporte técnico 24/7 por telefone e de resposta em menos de 1 hora quando um sistema de produção cair. Qual o plano mais barato que atende?
>
> A) Developer · B) Business · C) Enterprise On-Ramp · D) Enterprise

**Resposta:** B _(eu marquei C, sabendo a resposta)_

**Heads-up:** O Enterprise On-Ramp também atende, mas a pergunta pedia o **mais barato**. Em questão de suporte, procurar sempre o **"mais barato que atende"**.

| Plano | Caso mais grave | Destaque |
|---|---|---|
| Developer | Sistema prejudicado: < 12h | E-mail, horário comercial |
| **Business** | **Produção fora do ar: < 1h** | **O mais barato com 24/7** |
| Enterprise On-Ramp | Crítico fora do ar: < 30 min | Grupo de TAMs |
| Enterprise | Crítico fora do ar: < 15 min | TAM dedicado |

**Palavras-chave:** "24/7" ou "produção em 1 hora" + "mais barato" → Business · "TAM dedicado" ou "15 minutos" → Enterprise

---

## Bloco misturado #1 — prática, sem aula (15 questões)

- **Data:** 16/11/2026 · **Nota:** 40% (6/15)
- Primeiro bloco no formato de prova: temas sorteados entre os 4 domínios, sem dica e sem consulta.
- **Diagnóstico:** cerca de metade dos erros veio da leitura (marquei por impulso, sem ler todas as opções), e a outra metade de serviços que eu ainda não tinha estudado a fundo (Shield, FSx, Route 53 routing, Cost Anomaly Detection).
- **Regra nova:** ler **todas** as opções antes de marcar, eliminar 2, e só então escolher pela palavra-chave.
- Q5, Q8, Q13, Q14 e Q15: falta anotar o que marquei e o motivo.

### Bloco Q2 · [1 — Well-Architected] — motivo: confundi serviços (pilares)

> Qual pilar do AWS Well-Architected Framework trata de reduzir o impacto ambiental das cargas de trabalho na nuvem?
>
> A) Excelência operacional · B) Eficiência de desempenho · C) Sustentabilidade · D) Otimização de custos

**Resposta:** C _(eu marquei A)_

**Heads-up:** Desde 2021 o Well-Architected tem **6 pilares**, e **Sustentabilidade** é um deles. Se a pergunta fala de **meio ambiente, energia ou pegada de carbono**, a resposta é Sustentabilidade. **Excelência operacional** trata de **rodar e melhorar a operação** (automação, runbooks, aprender com falhas), não do planeta.

| Pilar | Pista |
|---|---|
| Excelência operacional | operar, monitorar, automatizar mudanças, aprender com falhas |
| Segurança | proteger dados e sistemas, rastreabilidade, menor privilégio |
| Confiabilidade | recuperar de falhas, escalar para a demanda, multi-AZ |
| Eficiência de desempenho | escolher o recurso certo, experimentar, serverless |
| Otimização de custos | não pagar por ociosidade, medir gasto |
| **Sustentabilidade** | **impacto ambiental, energia, reduzir desperdício de recursos** |

**Palavras-chave:** "ambiental", "carbono", "energia", "minimizar recursos ociosos pelo planeta" → Sustentabilidade

---

### Bloco Q3 · [2 — Responsabilidade compartilhada] — motivo: li errado

> Quem é responsável por aplicar patches no sistema operacional que executa as funções AWS Lambda?
>
> A) O cliente · B) A AWS · C) Os dois, em partes iguais · D) O parceiro do AWS Marketplace

**Resposta:** B _(eu marquei A, sabendo a resposta)_

**Heads-up:** Quanto mais **gerenciado** o serviço, mais a AWS assume. No **EC2** o SO é **do cliente**. No **Lambda** (serverless) não existe servidor para o cliente: SO, runtime e patches são da **AWS**. O cliente cuida do **código, das permissões IAM e dos dados**.

| Serviço | SO / patches | Cliente cuida de |
|---|---|---|
| EC2 | **Cliente** | SO, apps, security groups, dados |
| RDS | AWS | dados, acessos, configuração do banco |
| **Lambda** | **AWS** | **código, IAM, dados** |

**Palavras-chave:** "serverless" / "Lambda" / "gerenciado" + "patch do SO" → AWS · "EC2" + "patch do SO" → cliente

---

### Bloco Q5 · [2 — Governança / conformidade] — motivo: _a confirmar_

> Uma empresa quer registrar o histórico de configuração dos recursos AWS e ser avisada quando um recurso sair das regras internas (por exemplo, um bucket S3 ficar público). Qual serviço usar?
>
> A) AWS CloudTrail · B) AWS Config · C) Amazon CloudWatch · D) AWS Trusted Advisor

**Resposta:** B _(marquei: a confirmar)_

**Heads-up:** Os três "de olho" da AWS se confundem fácil: o **Config** responde **"como este recurso está configurado e se está conforme"**. O **CloudTrail** responde **"quem fez qual chamada de API e quando"**. O **CloudWatch** responde **"como está o desempenho"** (métricas, logs, alarmes).

| Serviço | Pergunta que responde |
|---|---|
| **Config** | **como o recurso está configurado, histórico, está em conformidade?** |
| CloudTrail | **quem** fez **o quê**, quando (auditoria de API) |
| CloudWatch | métricas, logs, alarmes de desempenho |
| Trusted Advisor | recomendações de boas práticas (custo, segurança, limites) |

**Palavras-chave:** "histórico de configuração", "conformidade", "regras", "drift" → Config · "quem fez", "auditoria de API" → CloudTrail

---

### Bloco Q7 · [2 — Segurança / DDoS] — motivo: não sabia

> Uma empresa quer proteção contra ataques DDoS sofisticados, acesso 24/7 à equipe de resposta a DDoS da AWS e proteção dos custos gerados por picos durante um ataque. O que usar?
>
> A) AWS Shield Standard · B) AWS Shield Advanced · C) AWS WAF · D) Amazon GuardDuty

**Resposta:** B _(eu marquei outra opção; não conhecia os tipos de Shield)_

**Heads-up:** O Shield tem **dois níveis**. O **Standard** é **gratuito e automático** para todo cliente (camadas 3 e 4). O **Advanced** é **pago** e acrescenta a **equipe de resposta a DDoS (SRT) 24/7**, a **proteção de custo** e relatórios. O **WAF** filtra requisições na **camada 7** (SQL injection, XSS, bloqueio por IP), e não é "o serviço anti-DDoS".

| Serviço | Pista |
|---|---|
| Shield Standard | grátis, automático, DDoS comum (camadas 3/4) |
| **Shield Advanced** | **pago, equipe DDoS 24/7 (SRT), proteção de custo** |
| WAF | regras na camada 7: SQL injection, XSS, IP, país |
| GuardDuty | detecção de ameaças analisando logs |

**Palavras-chave:** "DDoS" + "sem custo / automático" → Shield Standard · "DDoS" + "equipe 24/7" ou "proteção de custo" → Shield Advanced · "SQL injection / XSS" → WAF

---

### Bloco Q8 · [2 — IAM / várias contas] — motivo: _a confirmar_

> Uma aplicação na conta A precisa ler objetos de um bucket S3 da conta B, sem criar usuários nem compartilhar chaves de acesso. Qual a forma recomendada?
>
> A) Criar um usuário IAM na conta B e copiar as chaves para a conta A · B) Criar uma role IAM na conta B que a conta A possa assumir · C) Tornar o bucket público · D) Usar o AWS Organizations

**Resposta:** B _(marquei: a confirmar)_

**Heads-up:** Acesso entre contas = **role**. A conta B cria uma role com permissão no bucket e uma **política de confiança** que deixa a conta A **assumir** essa role (via STS). A conta A recebe **credenciais temporárias**. Chave de acesso copiada é o anti-padrão que a prova quer que você descarte.

| Recurso IAM | Use quando |
|---|---|
| Usuário | uma pessoa ou app com identidade fixa (credencial de longo prazo) |
| Grupo | dar as mesmas permissões a vários usuários |
| **Role** | **acesso temporário: serviço AWS (EC2→S3), outra conta, federação** |
| Política | o documento JSON que define as permissões |

**Palavras-chave:** "outra conta", "sem compartilhar chaves", "temporário", "EC2 acessar S3" → Role

---

### Bloco Q10 · [3 — Armazenamento] — motivo: li errado (impulso, não li a última opção)

> Uma empresa vai migrar servidores de arquivos Windows que usam compartilhamento SMB e Active Directory, e quer um serviço totalmente gerenciado. O que usar?
>
> A) Amazon EFS · B) Amazon S3 · C) Amazon EBS · D) Amazon FSx for Windows File Server

**Resposta:** D _(marquei antes de ler a D)_

**Heads-up:** O **EFS** é sistema de arquivos compartilhado para **Linux** (NFS). Se a pergunta fala em **Windows, SMB ou Active Directory**, a resposta é **FSx for Windows**. E nunca marque antes de ler **todas** as opções: a certa estava na última.

| Serviço | Pista |
|---|---|
| EBS | disco (bloco) de **uma** instância EC2 |
| EFS | arquivos compartilhados, **Linux / NFS**, várias EC2 |
| **FSx for Windows** | **Windows, SMB, Active Directory** |
| FSx for Lustre | HPC, alto desempenho, machine learning |
| S3 | objetos, ilimitado, acesso via HTTP |

**Palavras-chave:** "Windows", "SMB", "Active Directory" → FSx for Windows · "Linux", "NFS", "compartilhado" → EFS

---

### Bloco Q13 · [3 — Rede / DNS] — motivo: _a confirmar_

> Uma aplicação roda em várias regiões AWS e a empresa quer que cada usuário seja direcionado para a região que der a **menor latência** para ele. Qual recurso usar?
>
> A) Route 53 com roteamento por latência · B) Route 53 com roteamento simples · C) Route 53 com roteamento por geolocalização · D) Elastic Load Balancer

**Resposta:** A _(marquei: a confirmar)_

**Heads-up:** O Route 53 tem várias **políticas de roteamento**. A palavra da pergunta já diz qual é: "menor latência" leva ao roteamento **por latência**. **Geolocalização** decide pelo **local do usuário** (país ou continente), por exemplo para conteúdo em outro idioma ou por exigência legal, e não pela velocidade. O **ELB** distribui tráfego **dentro de uma região**, não entre regiões.

| Política | Use quando |
|---|---|
| Simples | um único recurso |
| Ponderada (weighted) | dividir %: 90/10, teste A/B, canary |
| **Latência** | **mandar para a região mais rápida para o usuário** |
| Failover | primário/secundário com health check |
| Geolocalização | decidir pelo **país/continente** do usuário |

**Palavras-chave:** "menor latência", "mais rápida" → Latência · "país", "idioma", "lei local" → Geolocalização · "porcentagem" → Ponderada · "recuperação de desastre" → Failover

---

### Bloco Q14 · [4 — Gestão de custos] — motivo: _a confirmar_

> Uma empresa quer ser alertada automaticamente quando o gasto subir de forma **fora do padrão**, sem precisar definir um valor-limite fixo. Qual serviço usar?
>
> A) AWS Budgets · B) AWS Cost Explorer · C) AWS Cost Anomaly Detection · D) AWS Pricing Calculator

**Resposta:** C _(marquei: a confirmar)_

**Heads-up:** No **Budgets** **você** define o limite ("me avise quando passar de US$ 500"). O **Cost Anomaly Detection** usa **machine learning** para descobrir sozinho o que é fora do normal, **sem limite fixo**. O **Cost Explorer** serve para **analisar** o gasto passado e a previsão. A **Pricing Calculator** **estima** o custo **antes** de usar.

| Serviço | Pista |
|---|---|
| Budgets | **limite que eu defino**, alerta quando passa ou vai passar |
| **Cost Anomaly Detection** | **"fora do padrão", "anomalia", ML, sem limite fixo** |
| Cost Explorer | gráficos, analisar gasto passado, previsão |
| Pricing Calculator | estimar antes de migrar ou criar |

**Palavras-chave:** "anomalia", "fora do padrão", "inesperado", "sem definir valor" → Cost Anomaly Detection · "limite", "orçamento" → Budgets

---

### Bloco Q15 · [4 — Planos de suporte] — motivo: _a confirmar_

> Uma equipe está **testando** cargas de trabalho na AWS (não é produção) e quer orientação técnica por **e-mail em horário comercial**, pagando o mínimo possível. Qual plano?
>
> A) Basic · B) Developer · C) Business · D) Enterprise On-Ramp

**Resposta:** B _(marquei: a confirmar)_

**Heads-up:** O **Basic** é grátis, mas **não tem suporte técnico**: só faturamento, conta, documentação e Trusted Advisor básico. O **Developer** é o **mais barato com suporte técnico**, por e-mail e em horário comercial, feito para **testes e desenvolvimento**. Quando a pergunta fala em **produção** ou **24/7**, o plano já é **Business**.

| Plano | Pista |
|---|---|
| Basic | grátis, **sem** suporte técnico |
| **Developer** | **testes/dev, e-mail, horário comercial** |
| Business | **produção**, 24/7, telefone, < 1h |
| Enterprise On-Ramp / Enterprise | crítico, TAM (grupo / dedicado) |

**Palavras-chave:** "teste", "desenvolvimento", "horário comercial", "e-mail" → Developer · "produção", "24/7" → Business

---

## Bloco misturado #2 — mesmos temas do #1, perguntas novas (10 questões)

- **Data:** 19/11/2026 · **Nota:** 100% (10/10) 🎯
- Usei o protocolo: anotar a palavra que decide, ler todas as opções, eliminar 2 e só então marcar. **De 40% para 100% com o mesmo conteúdo**: o problema do bloco #1 era a leitura.

### Anotação: quando a resposta é role IAM?

| Quem precisa do acesso | Use |
|---|---|
| **Serviço AWS → outro serviço AWS** (EC2→S3, Lambda→DynamoDB, ECS→SQS) | **Role** (na EC2 via *perfil de instância*; na Lambda, *execution role*) |
| Conta A → recurso da conta B | **Role** que a conta A assume |
| Login de funcionários com o AD/SSO da empresa (federação) | **Role** via IAM Identity Center |
| Uma pessoa com acesso fixo | Usuário IAM (em grupo) ou Identity Center |
| App **fora** da AWS (servidor on-premises) | Access key de longo prazo (o jeito moderno é IAM Roles Anywhere) |

**Palavras-chave:** "instância/função/container acessar outro serviço", "MAIS seguro", "sem armazenar credenciais", "temporário", "outra conta" → Role · Opção com access key no código, em variável de ambiente ou com root → **eliminar primeiro**

---

## Official Practice Question Set — 2ª tentativa (20 questões)

- **Data:** 23/11/2026 · **Nota:** 75% (15/20) · 1ª tentativa (26/10): 60%
- **Tempo:** 30 min (média de 1 min 31 s por questão). Na 1ª foram 13 min (41 s por questão): o protocolo de leitura está sendo usado.
- **Padrão principal:** Q5 e Q14 com as respostas **trocadas** entre si. VPN × Direct Connect é a confusão nº 1 a atacar.
- Critério para agendar a prova: 3 resultados seguidos ≥ 80%. Ainda não.

### Simulado Q2 · [2 — Segurança] — motivo: _a confirmar_

> Which AWS service identifies security groups that allow unrestricted access to a user's AWS resources?

**Resposta:** AWS Trusted Advisor _(eu marquei IAM)_

**Heads-up:** O **IAM** **define** quem pode fazer o quê. Ele não **analisa** a conta procurando problemas. Quem faz verificações automáticas e recomenda melhorias é o **Trusted Advisor**: security group com porta aberta para o mundo (`0.0.0.0/0`), root sem MFA, bucket S3 público, limites de serviço e economia de custo.

| Serviço | Pista |
|---|---|
| IAM | **criar e controlar** usuários, grupos, roles e políticas |
| **Trusted Advisor** | **checagens automáticas** de boas práticas: security group aberto, MFA no root, S3 público, custo, limites |
| Inspector | vulnerabilidades em EC2, containers e Lambda |
| GuardDuty | ameaças detectadas em logs |

**Palavras-chave:** "identifica", "verifica", "recomenda", "security group sem restrição", "boas práticas" → Trusted Advisor

---

### Simulado Q5 · [3 — Rede] — motivo: confundi serviços

> A company wants to establish a consistent and private connection from the company's on-premises data center to the AWS Cloud.

**Resposta:** AWS Direct Connect _(eu marquei Site-to-Site VPN)_

### Simulado Q14 · [3 — Rede] — motivo: confundi serviços

> A company requires an encrypted connection between the company's on-premises servers and AWS. The connection must use the company's existing internet connection.

**Resposta:** AWS Site-to-Site VPN _(eu marquei Direct Connect, sem certeza)_

**Heads-up (as duas juntas):** troquei as respostas entre si, então a regra é uma só. **Direct Connect** é um **cabo dedicado**, físico e privado: **não passa pela internet**, por isso é **consistente** (latência e banda estáveis). Leva semanas para instalar. **Site-to-Site VPN** é um **túnel criptografado pela internet que você já tem**: sobe em minutos, é barato, mas oscila com a internet.

| | Direct Connect | Site-to-Site VPN |
|---|---|---|
| Caminho | **link dedicado, fora da internet** | **pela internet existente** |
| Desempenho | **consistente**, previsível | varia com a internet |
| Criptografia | não por padrão (é privado, não cifrado) | **criptografado (IPsec)** |
| Tempo para ativar | semanas | minutos |
| Custo | alto | baixo |

**Palavras-chave:** "consistente", "dedicada", "privada", "não pela internet", "alta banda" → **Direct Connect** · "criptografada", "internet existente", "rápido de configurar", "barato" → **Site-to-Site VPN**

---

### Simulado Q12 · [1/3 — Migração] — motivo: _a confirmar_

> A company has an on-premises Linux-based server with an Oracle database that runs on it. The company wants to migrate the database server to run on an Amazon EC2 instance in AWS.

**Resposta:** AWS Application Migration Service _(eu marquei AWS DMS)_

**Heads-up:** A palavra que decide é "**server** → **EC2**": é levar o **servidor inteiro** como ele é (lift-and-shift, rehost). Isso é o **Application Migration Service (MGN)**, que replica a máquina para uma EC2. O **DMS** migra **os dados** de um banco para **outro banco**, normalmente gerenciado (RDS, Aurora), inclusive entre motores diferentes, com o SCT.

| Serviço | Pista |
|---|---|
| **Application Migration Service (MGN)** | **servidor inteiro** para **EC2**, lift-and-shift, rehost |
| DMS | **dados** de um banco para outro banco (para RDS/Aurora), com o banco de origem funcionando |
| Schema Conversion Tool (SCT) | converter schema entre motores diferentes (Oracle → PostgreSQL) |
| Snowball | mover muitos TB fisicamente, offline |

**Palavras-chave:** "servidor" + "EC2" + "como está" → MGN · "banco" + "RDS/Aurora" + "mínimo downtime" ou "outro motor" → DMS

---

### Simulado Q13 · [3 — Rede/CDN] — motivo: _a confirmar (não anotei o que marquei)_

> A company is hosting a static website from a single Amazon S3 bucket. Which AWS service will achieve lower latency and high transfer speeds?

**Resposta:** Amazon CloudFront

**Heads-up:** Usuários no mundo inteiro + site estático + menor latência = **CDN**, ou seja, **CloudFront**: ele guarda cópias nas **edge locations**, perto do usuário. Você até acertou a Q10, que fala das edge locations do CloudFront.

| Serviço | Pista |
|---|---|
| **CloudFront** | **CDN**, cache nas edge locations, conteúdo estático, menor latência para o usuário |
| S3 Transfer Acceleration | acelera **uploads** para um bucket distante |
| Global Accelerator | acelera tráfego TCP/UDP de **aplicações** pela rede da AWS, IPs fixos |
| Route 53 | DNS, roteamento |

**Palavras-chave:** "site estático", "conteúdo", "menor latência", "usuários globais", "cache" → CloudFront

---

## Bloco misturado #3 — erros do simulado + temas novos (15 questões)

- **Data:** 25/11/2026 · **Nota:** 87% (13/15) ✅ 1º resultado ≥ 80% (critério: 3 seguidos)
- Acertei: VPN (Q1), Trusted Advisor, DMS × Application Migration Service, CloudFront, Snowball, Auto Scaling, NAT Gateway, Savings Plans, KMS, Beanstalk, Glacier Deep Archive, SCPs.
- Errei: Direct Connect (de novo) e Artifact.

### Bloco #3 Q2 · [3 — Rede] — motivo: confundi serviços (3ª vez com VPN × Direct Connect)

> Uma empresa transfere grandes volumes de dados todos os dias entre o data center e a AWS e precisa de banda consistente, sem passar pela internet pública. O que usar?

**Resposta:** AWS Direct Connect

**Heads-up:** Essa é a 3ª vez. A regra de bolso: **procure a palavra "internet" no enunciado.**
- "**sem** passar pela internet", "consistente", "dedicada", "grandes volumes todo dia" → **Direct Connect** (cabo próprio, do prédio até a AWS)
- "**usando** a internet existente", "criptografada", "hoje", "barato" → **Site-to-Site VPN** (túnel pela internet)

**Truque do nome:** **Direct** Connect = conexão **direta**, um cabo seu, sem internet no meio.

---

### Bloco #3 Q11 · [2 — Conformidade] — motivo: confundi serviços

> Um auditor pede os relatórios de conformidade da AWS (como SOC e PCI). Onde a empresa obtém esses documentos?

**Resposta:** AWS Artifact _(eu marquei AWS Config)_

**Heads-up:** O **Config** olha os **seus** recursos: como estão configurados e se estão conformes com as **suas** regras. O **Artifact** é o "cartório" da AWS: onde você **baixa** os relatórios de auditoria **da própria AWS** (SOC, PCI, ISO) e aceita acordos (como o BAA). Se a pergunta fala em **relatório, certificado, documento de conformidade da AWS**, é Artifact.

| Serviço | Pista |
|---|---|
| **Artifact** | **baixar relatórios de conformidade da AWS** (SOC, PCI, ISO), acordos |
| Config | configuração e conformidade dos **seus** recursos, histórico |
| Audit Manager | coletar evidências da **sua** conta para a **sua** auditoria |
| Trusted Advisor | recomendações de boas práticas |

**Palavras-chave:** "relatório", "SOC", "PCI", "ISO", "documento de conformidade da AWS", "auditor pede" → Artifact

---

<!-- Próximas: copie o bloco de uma questão acima e preencha -->
