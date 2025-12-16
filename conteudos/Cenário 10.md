## 🎬 Cenário 10 - Acesso Cross-Account ao Amazon S3 com Princípio do Menor Privilégio

Um arquiteto de soluções precisa permitir que membros da equipe acessem os buckets do Amazon S3 em duas contas AWS diferentes:  
uma conta de desenvolvimento e uma conta de produção.  
A equipe atualmente tem acesso aos buckets do S3 na conta de desenvolvimento usando usuários IAM exclusivos que são atribuídos a um grupo IAM que possui permissões apropriadas na conta.  
O arquiteto de soluções criou uma função IAM na conta de produção.  
A função tem uma política que concede acesso a um bucket do S3 na conta de produção.  

### 🤔 Qual solução atenderá a esses requisitos enquanto estiver em conformidade com o princípio do menor privilégio?

### ➡️ Resposta
Adicionar a conta de desenvolvimento como um principal na política de confiança (trust policy) da função IAM na conta de produção.

## 🧠 Explicação

### 🔑 Acesso entre contas (Cross-Account Access)
Na AWS, o acesso entre contas é feito de forma segura usando:
- **IAM Roles**
- **Trust Policies (políticas de confiança)**

Nesse cenário:
- A função IAM está na **conta de produção**
- Os usuários estão na **conta de desenvolvimento**
- É necessário permitir que esses usuários **assumam a função** na conta de produção

Isso é feito adicionando a **conta de desenvolvimento** como um **principal confiável** na *trust policy* da função.

## 🛡️ Por que essa solução segue o Princípio do Menor Privilégio?

- Os usuários **não recebem permissões diretas** na conta de produção
- Eles só podem acessar o **bucket específico** definido na política da função
- O acesso ocorre **somente quando a função é assumida**
- Permissões ficam centralizadas e controladas

👉🏻 Nenhum acesso extra ou permanente é concedido.

## 🧩 Benefícios da Solução

- ✅ Controle granular de acesso ao S3
- ✅ Separação clara entre ambientes (dev e prod)
- ✅ Melhor governança e auditoria
- ✅ Fácil revogação de acesso
- ✅ Conformidade com boas práticas de segurança da AWS

## ⚠️ O que NÃO é recomendado nesse cenário?
- ❌ Criar usuários IAM duplicados na conta de produção
- ❌ Compartilhar credenciais entre contas
- ❌ Conceder permissões diretas ao S3 da conta de produção

Essas abordagens violam o princípio do menor privilégio e aumentam riscos de segurança.

## 📝 Dica para Provas AWS
Se o cenário mencionar:
- Múltiplas contas AWS
- Acesso ao S3 entre contas
- Princípio do menor privilégio

👉🏻 Pense imediatamente em **IAM Role + Trust Policy (cross-account access)**.

## 📌 Resumo Final
A forma correta e segura de permitir acesso ao Amazon S3 entre contas AWS é:
- Criar uma **função IAM** na conta de destino
- Conceder permissões mínimas ao recurso necessário
- Autorizar a **conta de origem** na política de confiança da função

Isso garante segurança, governança e conformidade com as melhores práticas da AWS.

---

👉🏻 [Clique aqui para voltar ao Readme](https://github.com/DrikaDev/Estudando-AWS-Solutions-Architect-Associate/blob/main/README.md) 📒
