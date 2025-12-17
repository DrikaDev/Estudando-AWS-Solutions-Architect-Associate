## 🎬 Cenário 16 - Migração de Armazenamento iSCSI On-Premises para a AWS com Baixa Latência

Uma empresa possui vários servidores de armazenamento de rede *ISCSI (Internet Small Computer Systems Interface) on-premises.  
A empresa deseja reduzir o número desses servidores migrando para a Nuvem AWS.  
Um arquiteto de soluções deve fornecer acesso de baixa latência a dados frequentemente utilizados e 
reduzir a dependência de servidores on-premises com um número mínimo de mudanças na infraestrutura.

### 🤔 Qual solução atenderá a esses requisitos?

### ➡️ Resposta
Implantar um **AWS Storage Gateway – Volume Gateway** configurado com **volumes em cache (cached volumes)**.

## 🏗️ Arquitetura da Solução

- O **AWS Storage Gateway** é implantado no ambiente on-premises (como VM ou hardware)
> É o serviço híbrido da AWS que conecta o ambiente on-premises com o armazenamento na nuvem AWS, permitindo que aplicações locais usem a nuvem como se fosse um storage local.  
> Permitindo usar a AWS como extensão do seu storage on-premises, com baixa latência e mínima mudança na infraestrutura.  
- O **Volume Gateway com volumes em cache**:
  - Mantém os **dados mais acessados localmente**
  - Armazena o **conjunto completo de dados na AWS (Amazon S3)**
- Os servidores existentes continuam acessando os volumes via **iSCSI**

> **iSCSI (Internet Small Computer Systems Interface)**
> É um **protocolo de rede** que permite acessar **armazenamento em bloco** através de uma rede **IP (TCP/IP)**,
> fazendo com que esse armazenamento remoto seja visto pelo sistema operacional **como se fosse um disco local**.
> O iSCSI possui dois componentes principais:  
> - **Initiator**: é o cliente (servidor ou sistema operacional), inicia a conexão e solicita acesso ao armazenamento
> - **Target**: é o servidor de armazenamento, disponibiliza os volumes (LUNs) para os initiators
> A comunicação ocorre via **TCP/IP**, normalmente utilizando a **porta 3260**.

## 🔍 Por que essa é a solução mais adequada?

### 1️⃣ Baixa latência para dados frequentemente acessados
Os **volumes em cache** mantêm uma cópia local dos dados mais utilizados, garantindo:
- Acesso rápido
- Baixa latência
- Melhor desempenho para workloads ativos

### 2️⃣ Redução da dependência de servidores on-premises
Com o armazenamento principal movido para a AWS:
- Menos necessidade de servidores físicos locais
- Menor custo com manutenção e escalabilidade
- Armazenamento durável e escalável na nuvem

### 3️⃣ Mudanças mínimas na infraestrutura
O **Volume Gateway com cached volumes** é:
- **Compatível com iSCSI**
- Totalmente integrado aos sistemas existentes

👉🏻 Não é necessário reescrever aplicações ou alterar profundamente a arquitetura atual.

### 4️⃣ Transição suave para a nuvem
Essa abordagem permite:
- Migração gradual para a AWS
- Manutenção do desempenho local
- Modernização da infraestrutura sem impacto significativo nas operações

## 🧠 Conceitos-chave para provas AWS

- **AWS Storage Gateway**
- **Volume Gateway**
- **Cached Volumes**
- **iSCSI**
- Migração híbrida
- Baixa latência
- Mudanças mínimas na infraestrutura

## 📎 Conclusão

O **AWS Storage Gateway – Volume Gateway com volumes em cache** é a solução ideal para empresas que desejam **migrar armazenamento iSCSI para a AWS**, 
reduzindo a infraestrutura on-premises, mantendo **alto desempenho local** e realizando a transição para a nuvem de forma **simples e eficiente**.

---

👉🏻 [Clique aqui para voltar ao Readme](https://github.com/DrikaDev/Estudando-AWS-Solutions-Architect-Associate/blob/main/README.md) 📒
