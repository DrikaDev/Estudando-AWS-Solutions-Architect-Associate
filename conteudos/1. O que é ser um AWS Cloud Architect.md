## O que é ser um AWS Cloud Architect? 👷🏻☁️

Ser um **AWS Cloud Architect** (ou trabalhar com *Cloud Architecting* na AWS) significa ser o profissional responsável por **desenhar, planejar e 
implementar soluções na nuvem usando os serviços da Amazon Web Services**.

É como ser o **"arquiteto da nuvem"**: você entende o problema do negócio e cria uma arquitetura segura, escalável, eficiente e bem estruturada usando os 
serviços da AWS.

<img width="833" height="423" alt="image" src="https://github.com/user-attachments/assets/12579706-6890-4cf6-b44d-cafd9bf86a18" />

---

## 🧠 A Profundidade do Papel do Arquiteto de Nuvem

Os arquitetos de nuvem dedicam seu tempo para **se atualizar constantemente** sobre os mais recentes desenvolvimentos e tendências da computação em nuvem.  
Eles são responsáveis por **projetar arquiteturas de aplicações** e escolher as tecnologias adequadas para atender requisitos técnicos e empresariais.

Para isso, precisam conhecer em detalhes as diversas opções de serviços de nuvem. Quanto maior o entendimento dos serviços, suas limitações e vantagens, melhor será a tomada de decisão sobre **quais serviços adotar para cada cenário**.

Os arquitetos de nuvem também orientam as equipes por meio de:
- **Diagramas de arquitetura**
- **Documentação técnica**
- **Ferramentas e boas práticas**

Eles fornecem direcionamento, mas também criam espaço para que desenvolvedores inovem para atingir os objetivos do projeto.

### 📌 Desafios comuns na função de arquiteto
- Gerenciamento eficiente de recursos  
- Otimização de custos  
- Definição de práticas recomendadas de desempenho  
- Garantia de confiabilidade e segurança  

As responsabilidades do arquiteto de nuvem se alinham diretamente aos pilares do **AWS Well-Architected Framework**, que orienta decisões sólidas de arquitetura.  
Essa perspectiva é a base para evoluir no papel e para avançar neste curso e na certificação.

---

## 🧩 O que faz um AWS Cloud Architect?

### ✔️ 1. Desenha arquiteturas na nuvem
Cria soluções como:

### 📌 1. Sistemas distribuídos
Sistemas onde diferentes partes trabalham separadas, de forma escalável e independente.

**Exemplos:**
- Sistema de e-commerce:
  - API no **API Gateway + Lambda**
  - Catálogo no **DynamoDB**
  - Processamento de pagamentos com **SQS + Lambda**
- Plataforma de streaming usando **CloudFront + S3**
- Arquitetura de microserviços com **EKS** ou **ECS**

---

### 📌 2. Aplicações web escaláveis
Aplicações que aumentam sua capacidade automaticamente conforme o tráfego cresce.

**Exemplos:**
- Aplicação em EC2 com:
  - **Auto Scaling Group**
  - **Application Load Balancer (ALB)**
- Aplicações serverless usando:
  - **API Gateway**
  - **AWS Lambda**
  - **S3 / DynamoDB**
- Containers escalando automaticamente no **ECS Fargate**

---

### 📌 3. Pipelines de dados
Fluxos automáticos de ingestão, transformação e análise de dados.

**Exemplos:**
- Pipeline em tempo real:
  - **Kinesis Data Streams → Lambda → DynamoDB**
- ETL automatizado:
  - **AWS Glue → S3 → Athena**
- Data Lake completo:
  - Armazenamento em **S3**
  - Catálogo no **Glue Catalog**
  - Consultas com **Athena**
  - Dashboards no **QuickSight**

---

### 📌 4. Ambientes de alta disponibilidade (HA)
Arquiteturas que permanecem no ar mesmo se uma zona de disponibilidade falhar.

**Exemplos:**
- Aplicação em EC2 distribuída em várias AZs com:
  - **ALB**
  - **RDS Multi-AZ**
- API serverless com:
  - **Lambda** (multi-AZ de forma nativa)
  - **DynamoDB** (multi-AZ por padrão)
- Cluster **EKS** com nós espalhados em múltiplas zonas

---

### 📌 5. Infraestruturas resilientes e tolerantes a falhas
Arquiteturas preparadas para se recuperar automaticamente de erros.

**Exemplos:**
- Filas **SQS** para evitar perda de mensagens
- Processamento assíncrono com **SNS + SQS + Lambda**
- Backups automáticos com:
  - **AWS Backup**
  - **Snapshots automáticos**
- Cache e fallback global usando **CloudFront**
- Armazenamento durável no **S3** (11 9's de durabilidade)

---

### ✔️ 2. Traduz necessidades do negócio em soluções técnicas
Exemplo:  
A empresa quer reduzir custos → você analisa e desenha uma solução com instâncias otimizadas, serverless, autoscaling, etc.

---

### ✔️ 3. Garante segurança e boas práticas
Inclui:
- IAM corretamente configurado  
- Princípio do menor privilégio  
- Segmentação de redes (VPC, subnets, NACLs, SGs)  
- Criptografia e auditoria (KMS, CloudTrail)  

---

### ✔️ 4. Otimiza custos
Avalia:
- Saving Plans  
- Reservas  
- Ajuste de recursos  
- Observabilidade  
- Uso de serviços gerenciados  

---

### ✔️ 5. Trabalha com equipes multidisciplinares
Colabora com:
- Devs  
- SREs  
- Segurança  
- Produto  
- Dados  
- Infraestrutura  

> Você não está ali apenas para “clicar no console”, mas para **liderar decisões técnicas**.

---

### ✔️ 6. Documenta e revisa arquiteturas
Cria:
- Diagramas (Lucidchart, Draw.io, Whimsical)  
- Documentos de arquitetura (ADR, HLDs, LLDs)  
- Playbooks e boas práticas  

---

## 🎯 Habilidades essenciais de um AWS Cloud Architect

### 🔹 Habilidades técnicas
- Redes na AWS (VPC, subnets, rotas, NAT, VPN, Direct Connect)  
- Computação (EC2, Fargate, Lambda)  
- Armazenamento (S3, EFS, FSx)  
- Bancos (RDS, Aurora, DynamoDB)  
- Segurança (IAM, KMS, WAF, Shield)  
- Observabilidade (CloudWatch, X-Ray)  
- Infra como código (CloudFormation, Terraform)  
- Microserviços  
- Arquitetura Serverless  

---

### 🔹 Soft skills
- Pensamento arquitetural  
- Tomada de decisão técnica  
- Boa comunicação  
- Análise de trade-offs  
- Explicar soluções complexas de forma simples  
- Documentação clara  

---

## 🎓 E o AWS Solutions Architect – Associate?

É o **exame que comprova que você entende os fundamentos para atuar como arquiteto(a) na AWS**, incluindo:
- Alta disponibilidade  
- Tolerância a falhas  
- Custos  
- Segurança  
- Boas práticas de arquitetura  

> Ou seja: **é a porta de entrada para a carreira de Cloud Architect**.

---
