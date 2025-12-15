## Cenário 5 - S3, SQS e Lambda

Uma empresa implementou um aplicativo serverless que invoca uma função AWS Lambda sempre que novos documentos são carregados em um bucket Amazon S3.  
O aplicativo usa a função Lambda para processar os documentos.  
Após uma campanha de marketing recente, a empresa percebeu que o aplicativo não processou muitos dos documentos.  

### 🤔 O que um arquiteto de soluções deve fazer para melhorar a arquitetura deste aplicativo?

### ➡️ Reposta:
Criar uma fila Amazon Simple Queue Service (Amazon SQS). Enviar os requests para a fila. Configurar a fila como uma fonte de eventos para a função Lambda.  

## 🔍 Análise do Problema

A arquitetura atual é:
```
Amazon S3
↓ (evento)
AWS Lambda
```

- Cada upload no S3 **invoca diretamente** a Lambda  
- Após a campanha de marketing recente, houve **picos de uploads simultâneos**  
- Muitos documentos **não foram processados**

⚠️ Isso indica um problema clássico de **pico de carga e perda de eventos**.

## 🚨 Onde está o gargalo?

Apesar de o AWS Lambda escalar automaticamente, existem limitações importantes:

- Limite de **concorrência** da conta ou da função  
- Possibilidade de **throttling (limitação de velocidade)** durante picos  
- Se a invocação falhar, o evento do S3 **não é reenfileirado automaticamente**  
- O Amazon S3 **não garante *retries* infinitos**

👉🏻 Resultado: durante picos de tráfego, **eventos podem ser perdidos**.

### ✅ Solução Recomendada pela AWS: Desacoplar o S3 da Lambda usando uma fila **Amazon SQS**

## 🏗️ Arquitetura Melhorada
```
Amazon S3
↓
Amazon SQS (fila)
↓
AWS Lambda
```

## 🤔 O que muda?

1. O S3 envia eventos para o **Amazon SQS**, não diretamente para a Lambda  
2. A fila **absorve picos de milhares de uploads**  
3. A Lambda consome mensagens **no ritmo suportado**  
4. Em caso de falha:
   - A mensagem **permanece na fila durante o visibility timeout**
   - Há **retries automáticos**
   - Pode-se configurar uma **Dead-Letter Queue (DLQ)**
     > **Dead-Letter Queue (DLQ)** é uma fila de mensagens para erros.  
     > É onde vão parar mensagens que **não conseguiram ser processadas após várias tentativas**.  

## 🚀 Por que essa arquitetura funciona?

### ✔️ Tolerância a picos
A fila atua como um **buffer de carga**, evitando sobrecarga da Lambda.
> *Buffer é um amortecedor de carga.
> É um componente que recebe dados/eventos, guarda temporariamente e entrega no ritmo que o consumidor consegue processar.

### ✔️ Maior confiabilidade
Nenhum documento é perdido se a função falhar temporariamente.  

### ✔️ Escalabilidade controlada
A Lambda **escala** com base no **tamanho da fila**.  
> 1. A AWS **monitora continuamente a fila SQS**
> 2. Detecta **mensagens pendentes**
> 3. Cria **execuções concorrentes da função Lambda**
> 4. Cada execução processa **uma ou um lote de mensagens**
> 5. Quanto maior a fila, **mais execuções em paralelo** são criadas
> 👉🏻 Isso é o que significa **escalar com base na fila**.

### ✔️ Arquitetura resiliente
Menor acoplamento entre produtor (S3) e consumidor (Lambda).  

## ⚠️ Pegadinhas Clássicas de Prova

- ❌ Aumentar memória da Lambda → não evita perda de eventos  
- ❌ Aumentar timeout → não resolve picos de carga  
- ❌ \"Lambda já escala automaticamente\" → escala, mas **não garante durabilidade do evento**

## 📌 Resumo Final

- AWS Lambda escala, mas **não é um buffer**  
- Amazon SQS garante **durabilidade e absorção de picos**  
- Arquiteturas serverless **também precisam ser resilientes**

👉🏻 Sempre que houver **eventos + picos imprevisíveis**, pense em **SQS no meio**.

---

👉🏻 [Clique aqui para voltar ao Readme](https://github.com/DrikaDev/Estudando-AWS-Solutions-Architect-Associate/blob/main/README.md) 📒
