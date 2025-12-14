## 🎬 Cenário 3 - Escolhendo o banco de dados certo sem mudar o MySQL

Uma empresa possui uma aplicação web com padrões de uso esporádicos.  
Há um uso intenso no início de cada mês, uso moderado no início de cada semana e uso imprevisível durante a semana.  
A aplicação consiste em um servidor web e um servidor de banco de dados MySQL em execução dentro do data center.  

### 🤔 A empresa gostaria de mover a aplicação para a AWS Cloud e precisa selecionar uma plataforma de banco de dados com custo eficaz que não exigirá modificações no banco de dados.  

### ➡️ Resposta: 
Amazon Aurora Serverless compatível com MySQL

## 🧩 Entendendo o problema (tradução da prova)
A empresa tem:

🔹 Tipo de aplicação
- Aplicação web tradicional
- Com: Servidor web / Servidor de banco de dados MySQL
- Atualmente rodando on-premises

🔹 Padrão de uso
- Pico forte no início do mês
- Uso moderado no início da semana
- Uso imprevisível durante a semana
- Ou seja: workload irregular e esporádico

🔹 Requisitos IMPORTANTES
- Quer mover para a AWS
- Precisa de um banco de dados: custo eficaz / sem modificações no banco / compatível com MySQL

⚠️ Essas três frases são as chaves da questão.

## 🔑 O que a prova quer que você perceba  
1️⃣ “Não exigir modificações no banco de dados”  
👉 Favorece: MySQL compatível (RDS MySQL ou Aurora MySQL)  

2️⃣ “Padrões de uso esporádicos”  
👉 A empresa não quer pagar por capacidade ociosa.  
Então precisa de algo que escale automaticamente.  

3️⃣ “Custo eficaz”  
👉 Isso elimina soluções:  
- superdimensionadas
- com alta complexidade
- com custo fixo alto

## 🧠 Tradução do raciocínio da AWS

Quando uma aplicação possui padrões de uso imprevisíveis e requer compatibilidade com MySQL sem modificações, o Amazon Aurora Serverless oferece escalabilidade automática e 
custo otimizado.  

## 📝 Frase pronta de prova (essa cai MUITO)

Para workloads esporádicos e imprevisíveis, o Amazon Aurora Serverless é a opção mais econômica quando não se deseja gerenciar capacidade de banco de dados.  

---

👉🏻 [Clique aqui para voltar ao Readme](https://github.com/DrikaDev/Estudando-AWS-Solutions-Architect-Associate/blob/main/README.md) 📒
