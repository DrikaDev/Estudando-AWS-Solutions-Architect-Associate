## 🎬 Cenário 14 - Governança de Tags de Uso de Custos com AWS Organizations

Uma empresa está executando suas cargas de trabalho de ambiente de produção e não produção em várias contas AWS.  
As contas estão em uma organização na AWS Organizations.  
A empresa precisa projetar uma solução que impeça a modificação das tags de uso de custos.  

### 🤔 Qual solução atenderá a esses requisitos?

### ➡️ Resposta
Criar uma política de controle de serviço (SCP) para evitar a modificação de tags, exceto por princípios autorizados.

## 🧠 Explicação Geral

O **AWS Organizations** fornece uma forma centralizada de gerenciar múltiplas contas AWS.  
Uma de suas funcionalidades mais importantes são as **Service Control Policies (SCPs)**.

As **SCPs** permitem:
- Definir **limites máximos de permissões**
- Aplicar regras de segurança e governança **em todas as contas da organização**
- Garantir conformidade com políticas corporativas

## 🔐 Como a SCP resolve o problema

- É possível criar uma SCP que **negue ações relacionadas à modificação de tags**
- A negação pode ser aplicada:
  - A todas as contas da organização
  - Com exceção de **usuários ou funções específicas autorizadas**
- Isso impede alterações:
  - Acidentais
  - Maliciosas
  - Fora do padrão corporativo

## 🎯 Benefícios da Solução

### 🏛️ Governança Centralizada
- Controle aplicado no nível da organização
- Não depende de configurações individuais por conta

### 🔒 Segurança e Conformidade
- Protege tags críticas usadas para:
  - **Cost Allocation**
  - Relatórios financeiros
  - Chargeback interno

### ⚙️ Eficiência Operacional
- Reduz retrabalho
- Evita erros humanos
- Facilita auditorias

## 📝 Dica para Provas AWS

Se o cenário mencionar:
- Múltiplas contas AWS  
- AWS Organizations  
- Governança centralizada  
- Controle de permissões globais  
- Proteção de tags ou padrões corporativos  

👉🏻 Pense imediatamente em **Service Control Policies (SCPs)**.

---

👉🏻 [Clique aqui para voltar ao Readme](https://github.com/DrikaDev/Estudando-AWS-Solutions-Architect-Associate/blob/main/README.md) 📒
