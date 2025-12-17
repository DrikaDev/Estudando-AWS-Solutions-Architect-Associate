## 🎬 Cenário 15 - Failover Regional com AWS Lambda, API Gateway e Amazon Route 53

Uma empresa possui uma aplicação web sem estado que é executada em funções do AWS Lambda que são invocadas pelo Amazon API Gateway.  
A empresa deseja implantar a aplicação em várias Regiões da AWS para fornecer capacidades de failover regional.

### 🤔 O que um arquiteto de soluções deve fazer para rotear o tráfego para várias Regiões?

### ➡️ Resposta
Criar verificações de integridade do Amazon Route 53 para cada Região.  
Utilizar uma configuração de failover *ativo-ativo.

## Arquitetura Ativo-Ativo (Active-Active)

**Ativo-ativo (active-active)** é uma estratégia de arquitetura em que **duas ou mais instâncias, ambientes ou Regiões estão ativas simultaneamente**, recebendo e processando 
tráfego **ao mesmo tempo**.  

Não existe um ambiente principal e outro de backup: 👉🏻 **todos os ambientes estão ativos e atendendo usuários**.

### 🔄 Exemplo prático (AWS)
Uma aplicação está implantada em:
- **sa-east-1 (São Paulo)**
- **us-east-1 (N. Virginia)**

Em uma arquitetura **ativo-ativo**:
- Usuários do Brasil são atendidos pela Região de São Paulo
- Usuários dos EUA são atendidos pela Região da Virgínia
- Ambas as Regiões recebem tráfego simultaneamente

Se uma Região ficar indisponível:
- O tráfego é automaticamente redirecionado para a Região saudável
- A aplicação continua funcionando sem interrupção perceptível

### 🆚 Comparação: Ativo-Ativo vs Ativo-Passivo

| Modelo | Como funciona |
|-----|-------------|
| **Ativo-Ativo** | Todas as Regiões/instâncias estão ativas e recebem tráfego ao mesmo tempo |
| **Ativo-Passivo** | Uma Região ativa e outra fica em standby (backup) |

### 🚀 Vantagens do Ativo-Ativo
- Alta disponibilidade
- Menor latência para os usuários
- Melhor utilização dos recursos
- Failover quase imediato

### ⚠️ Desafios do Ativo-Ativo
- Replicação de dados entre Regiões
- Garantia de consistência de dados
- Custo operacional mais elevado
- Maior complexidade arquitetural

> ⚠️ Por esse motivo, arquiteturas ativo-ativo são mais indicadas para **aplicações sem estado (stateless)**.

## 🏗️ Arquitetura da Solução

- Cada Região possui:
  - Amazon API Gateway
  - Funções AWS Lambda
- O **Amazon Route 53** é utilizado como serviço de DNS global
- O tráfego é distribuído entre **múltiplas Regiões saudáveis**
- Caso uma Região falhe, o Route 53 **para de rotear tráfego** para ela automaticamente

## 🔍 Por que essa é a solução mais adequada?

### 1️⃣ Amazon Route 53 é um DNS global altamente disponível
O **Amazon Route 53** é um serviço de DNS **altamente disponível, escalável e resiliente**, projetado para **rotear tráfego entre endpoints em diferentes Regiões da AWS**.

### 2️⃣ Verificações de integridade detectam falhas regionais
As **health checks do Route 53** monitoram continuamente a saúde de cada endpoint regional (API Gateway).

- Se uma Região ficar indisponível:
  - O endpoint é marcado como **não saudável**
  - O tráfego deixa de ser enviado para essa Região
- Isso permite **failover automático** entre Regiões

### 3️⃣ Failover ativo-ativo melhora desempenho e resiliência
Na configuração **active-active**:

- Todas as Regiões estão **ativas simultaneamente**
- O tráfego é distribuído entre elas
- Se uma Região falhar, as demais continuam atendendo usuários

Benefícios:
- Menor latência
- Alta disponibilidade
- Melhor experiência do usuário

### 4️⃣ Ideal para aplicações sem estado (stateless)
Como a aplicação é **sem estado**:

- Não há necessidade de sincronizar sessões ou dados entre Regiões
- Cada requisição pode ser atendida por qualquer Região
- Facilita arquiteturas **multi-região e altamente disponíveis**

## 🧠 Conceitos-chave cobrados em provas AWS

- Arquitetura **multi-região**
- **Failover regional**
- **Amazon Route 53**
- **Health checks**
- **Active-active**
- Aplicações **stateless**

## 📎 Conclusão

Utilizar o **Amazon Route 53 com verificações de integridade** e uma estratégia de **failover ativo-ativo** é a solução mais indicada para rotear tráfego entre várias Regiões 
em aplicações **serverless e sem estado**, garantindo **alta disponibilidade, resiliência e performance global**.

---

👉🏻 [Clique aqui para voltar ao Readme](https://github.com/DrikaDev/Estudando-AWS-Solutions-Architect-Associate/blob/main/README.md) 📒
