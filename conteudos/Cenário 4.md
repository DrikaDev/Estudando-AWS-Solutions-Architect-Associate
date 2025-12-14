## Cenário 4 - AWS Lake Formation (LF-TBAC)

Uma empresa armazena vários petabytes de dados em várias contas AWS.  
A empresa usa o AWS Lake Formation para gerenciar seu lago de dados.  
A **equipe de ciência de dados** da empresa pretende partilhar de forma segura dados seletivos das suas contas com a **equipe de engenharia** da empresa para fins analíticos.

### 🤔 Qual solução atenderá a esses requisitos com o MENOR custo operacional?

### ➡️ Resposta:
Usar o controle de acesso baseado em tags (LF-TBAC) do Lake Formation para autorizar e conceder permissões entre contas apenas para os dados necessários à equipe de engenharia.  

> Em outras palavras: compartilhar dados seletivos, entre contas, com segurança, sem copiar dados, usando tags.

## 🧩 Entendendo o cenário
A empresa:
- Armazena vários petabytes de dados
- Usa múltiplas contas AWS
- Gerencia o data lake com AWS Lake Formation

Até aqui, duas palavras já acendem o alerta de prova:
- Data lake
- Lake Formation

## 🎯 O que a equipe de ciência de dados quer?
- Compartilhar dados seletivos: não é tudo, apenas alguns datasets, tabelas ou colunas
- De forma segura
- Entre contas AWS 
- Para análises (analytics)

⚠️ Isso elimina soluções “na unha” como:  
- políticas do S3 policies manuais
- cópia de dados
- controle apenas com IAM

## 🔑 O que a AWS espera como solução
Usar o cross-account data sharing do AWS Lake Formation com:   
- compartilhamento de bancos, tabelas ou colunas
- controle via Lake Formation tags
- sem mover ou duplicar dados

## 🔑 O que é LF-TBAC (Lake Formation Tag-Based Access Control)?
O **LF-TBAC** é um modelo de controle de acesso onde:  
- Os dados do data lake são **marcados com tags (LF-tags)**  
- As permissões são concedidas **às tags**, e não diretamente aos dados

👉 Qualquer principal (usuário, role ou conta) que tenha permissão para uma tag pode acessar **todos os dados associados a ela**.

## 🏷️ Exemplo Prático

### 🔹 Conta A - Equipe de Ciência de Dados

Tabela: `vendas`

| Coluna | Tag aplicada |
|------|-------------|
| valor | sensibilidade=baixa |
| regiao | sensibilidade=baixa |
| cpf | sensibilidade=alta |

LF-tags criadas:
- `dominio = financeiro`
- `sensibilidade = baixa`
- `sensibilidade = alta`

### 🔹 Conta B - Equipe de Engenharia 

Permissão concedida:
- Acesso apenas à tag `sensibilidade=baixa`

📌 Resultado:
- ✅ Pode consultar `valor` e `regiao`
- ❌ Não pode acessar `cpf`
- ❌ Não pode fazer cópia ou movimentação de dados

## 🎯 Por que essa solução é ideal para o cenário?

### 1️⃣ Grande volume de dados

- ❌ Copiar petabytes entre contas é caro e ineficiente
- ✅ Lake Formation compartilha dados **sem duplicação**

### 2️⃣ Compartilhamento seletivo

O LF-TBAC permite controle granular:
- Por banco
- Por tabela
- Por coluna
- Por categoria de dados (tags)

👉 IAM ou políticas de bucket S3 **não oferecem esse nível de granularidade**.

### 3️⃣ Compartilhamento entre contas AWS (multi-conta)

O Lake Formation oferece **cross-account data sharing nativo**, integrado com:

- AWS Glue Data Catalog
- Amazon Athena
- Amazon Redshift Spectrum
- Amazon EMR

### 4️⃣ Segurança e Governança

- Governança centralizada
- Princípio do menor privilégio
- Auditoria via AWS CloudTrail

## ⚠️ Pegadinha Clássica de Prova AWS

Se aparecer no enunciado:

- Data lake
- Governança de dados
- Compartilhamento seletivo
- Múltiplas contas AWS
- AWS Lake Formation

👉 **A resposta provavelmente envolve LF-tags (TBAC)**.

## 📌 Resumo Final

- Use **AWS Lake Formation** para governança de data lakes
- Use **LF-TBAC** para controle de acesso escalável
- Compartilhe dados entre contas **sem copiar** informações!
- Ideal para ambientes com **grande volume de dados**, **baixo custo operacional** e **múltiplas equipes**

---

👉🏻 [Clique aqui para voltar ao Readme](https://github.com/DrikaDev/Estudando-AWS-Solutions-Architect-Associate/blob/main/README.md) 📒
