## Cenário 11 - Alta Disponibilidade e Resiliência na AWS

Uma empresa deseja utilizar a nuvem AWS para tornar um aplicativo existente altamente disponível e resiliente.  
A versão atual do aplicativo está no data center da empresa.  
Recentemente, o aplicativo sofreu perda de dados após a falha de um servidor de banco de dados devido a uma queda de energia inesperada.  
A empresa precisa de uma solução que evite pontos únicos de falha.  
A solução deve permitir que o aplicativo escale para atender à demanda do usuário.  

### 🤔Qual solução atenderá a esses requisitos? (de **alta disponibilidade**, **resiliência** e **escalabilidade**, evitando pontos únicos de falha?)

### ➡️ Resposta
Implantar os servidores do aplicativo usando instâncias Amazon EC2 em um grupo de Auto Scaling em várias Zonas de Disponibilidade.  
Usar uma instância Amazon RDS DB em uma configuração Multi-AZ.  

## 🧠 Explicação

### 🖥️ Camada de Aplicação — Amazon EC2 + Auto Scaling
- O **Auto Scaling Group (ASG)** distribui instâncias EC2 em **múltiplas Zonas de Disponibilidade**
- Se uma instância ou uma AZ falhar, o Auto Scaling:
  - Substitui automaticamente as instâncias afetadas
  - Mantém a aplicação disponível
- Elimina **pontos únicos de falha** na camada de aplicação

### 🗄️ Camada de Banco de Dados — Amazon RDS Multi-AZ
- O **Amazon RDS Multi-AZ** replica os dados de forma síncrona para uma instância de standby em outra AZ
- Em caso de falha:
  - O RDS realiza **failover automático**
  - A aplicação continua operando sem perda de dados
- Garante **alta disponibilidade e durabilidade dos dados**

## 📈 Escalabilidade
- O **Auto Scaling Group** ajusta automaticamente a quantidade de instâncias EC2:
  - Aumenta durante picos de acesso
  - Reduz quando a demanda diminui
- Permite escalar o aplicativo conforme o crescimento do uso

## 🎯 Benefícios da Solução
- ✅ Alta disponibilidade
- ✅ Resiliência a falhas de infraestrutura
- ✅ Eliminação de pontos únicos de falha
- ✅ Recuperação automática
- ✅ Escalabilidade automática
- ✅ Segurança e confiabilidade dos dados

## 📝 Dica para Provas AWS
Se o cenário mencionar:
- Falha de servidor
- Queda de energia
- Perda de dados
- Alta disponibilidade + escalabilidade

👉🏻 Pense em **EC2 + Auto Scaling em múltiplas AZs**  
👉🏻 Combine com **Amazon RDS Multi-AZ** para o banco de dados

## 📌 Resumo Final
A combinação de:
- **Amazon EC2 com Auto Scaling em múltiplas Zonas de Disponibilidade**
- **Amazon RDS em configuração Multi-AZ**

é a abordagem recomendada pela AWS para criar aplicações **altamente disponíveis, resilientes e escaláveis**, evitando pontos únicos de falha.

---

👉🏻 [Clique aqui para voltar ao Readme](https://github.com/DrikaDev/Estudando-AWS-Solutions-Architect-Associate/blob/main/README.md) 📒
