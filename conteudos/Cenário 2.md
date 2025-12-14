## 🎬 Cenário 2 - Processamento de Modelos de ML com API Assíncrona, SQS e ECS

Uma empresa está desenvolvendo uma nova solução de modelo de aprendizado de máquina (ML) na AWS.  
Os modelos são desenvolvidos como microsserviços independentes que buscam aproximadamente 1 GB de dados de modelo do Amazon S3 no início e carregam os dados na memória.  
Os usuários acessam os modelos por meio de uma API assíncrona.  
Os usuários podem enviar uma solicitação ou um lote de solicitações e especificar para onde os resultados devem ser enviados.  

A empresa fornece modelos para centenas de usuários.  
Os padrões de uso dos modelos são irregulares.  
Alguns modelos podem ficar sem uso por dias ou semanas.  
Outros modelos podem receber lotes de milhares de solicitações de uma só vez.  

🤔 Qual design um arquiteto de soluções deve recomendar para atender a esses requisitos?  

➡️ Resposta: Direcionar as solicitações da API para uma fila do Amazon Simple Queue Service (Amazon SQS).  
Implantar os modelos como serviços do Amazon Elastic Container Service (Amazon ECS) que leem da fila.  
Habilitar o AWS Auto Scaling no Amazon ECS tanto para o cluster quanto para as cópias do serviço com base no tamanho da fila.  

## Explicando o porquê de cada decisão
Essa arquitetura foi escolhida porque resolve três problemas ao mesmo tempo:  
1. Processamento assíncrono
2. Carga irregular (picos e ociosidade)
3. Modelos grandes e pesados em memória

## 👇🏻 Agora vamos por partes 👇🏻

### 1️⃣ Direcionar as solicitações da API para uma fila do Amazon SQS

*Por que usar SQS?*  
A API é assíncrona → não precisa processar na hora  
O SQS:  
- desacopla quem envia de quem processa
- absorve picos de milhares de requisições
- garante durabilidade das mensagens

**O que isso resolve no cenário?**

| Problema                         | Como o Amazon SQS ajuda                           |
|----------------------------------|--------------------------------------------------|
| Pico de milhares de solicitações | A fila armazena e controla o volume de mensagens |
| Usuários simultâneos             | Evita sobrecarga direta nos serviços de backend  |
| Processamento em lote            | Permite consumo de mensagens em massa            |

👉 Resumo: SQS funciona como um amortecedor de carga.  

### 2️⃣ Implantar os modelos como serviços do Amazon ECS que leem da fila

*Por que ECS (containers)?*  
O Amazon ECS é o motor de processamento dos modelos de Machine Learning.
Ele é responsável por:
- executar os microsserviços de ML em containers  
- carregar ~1 GB de modelo do S3 na inicialização
- manter dados em memória
- ler mensagens da fila SQS
- processar uma ou várias soliciações
- enviar o resultado para o destino informado pelo usuário

**O que isso resolve no cenário?**

| Requisito              | Por que usar Amazon ECS                     |
|------------------------|---------------------------------------------|
| Modelos grandes        | AWS Lambda não suporta bem cargas pesadas   |
| Microsserviços         | ECS é nativo para arquiteturas de containers|
| Isolamento por modelo  | Cada modelo pode ser implantado como um serviço ECS independente |

👉 Resumo: ECS é ideal para workloads pesados e stateful em memória.  

### 3️⃣ Habilitar Auto Scaling no ECS com base no tamanho da fila

*Por que escalar pela fila?*  
O tamanho da fila mostra:
- quantas solicitações estão pendentes  
- a carga real do sistema

**O que isso resolve no cenário?**

| Situação                     | Com AWS Auto Scaling                     |
|------------------------------|------------------------------------------|
| Milhares de mensagens na fila| O ECS cria mais containers automaticamente |
| Fila vazia por dias          | O ECS reduz a quantidade de containers (ou escala para zero) |
| Uso imprevisível             | A capacidade escala automaticamente conforme a demanda |

👉 Resumo: A fila vira o “termômetro” da escala.  

### 🧩 Juntando tudo (o raciocínio da AWS)
API assíncrona  
→ SQS (buffer + durabilidade)  
→ ECS (processamento pesado)  
→ Auto Scaling (escala sob demanda)  

Isso garante:  
⚡ performance  
💸 baixo custo  
📈 escalabilidade  
🔄 tolerância a picos  

### 📝 Frase-chave para prova AWS  
Uma arquitetura baseada em SQS + ECS com Auto Scaling permite processar workloads assíncronos e irregulares de forma escalável e econômica.  

---

👉🏻 [Clique aqui para voltar ao Readme](https://github.com/DrikaDev/Estudando-AWS-Solutions-Architect-Associate/blob/main/README.md) 📒
