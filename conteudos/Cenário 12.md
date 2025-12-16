## 🎬 Cenário 12 - Migração de Grandes Volumes de Dados para a AWS com DataSync

Uma empresa está a armazenar 700 terabytes de dados num grande sistema de armazenamento ligado à rede (*NAS) no seu centro de dados empresarial.  
A empresa tem um ambiente híbrido com uma ligação AWS Direct Connect de 10 Gbps.  
Após uma auditoria de um regulador, a empresa tem 90 dias para mover os dados para a nuvem.  
A empresa precisa de mover os dados de forma eficiente e sem interrupções. A empresa ainda precisa ser capaz de acessar e atualizar os dados durante a janela de transferência.  

- **NAS** significa Network Attached Storage — em português, Armazenamento Conectado à Rede.

### 🤔 Qual solução atenderá a esses requisitos?

### ➡️ Resposta
Criar um agente do AWS DataSync no centro de dados corporativo. 
Criar uma tarefa de transferência de dados Iniciar a transferência para um bucket do Amazon S3.

## 🧠 Explicação Geral

### 🔄 AWS DataSync
O **AWS DataSync** é um serviço totalmente gerenciado projetado para transferências de dados:
- Rápidas
- Seguras
- Confiáveis

Ele é ideal para migrações entre:
- Armazenamento on-premises (como NAS)
- Serviços de armazenamento da AWS (como Amazon S3)

### 🖥️ Agente do DataSync no Data Center
- O **agente do AWS DataSync** é instalado no ambiente local
- Ele se conecta diretamente ao sistema NAS
- Otimiza o uso da rede para transferências de grande volume

### 🌐 Uso do AWS Direct Connect
- A conexão **Direct Connect de 10 Gbps**:
  - Garante alta largura de banda
  - Oferece desempenho previsível
  - Reduz latência e custos de transferência
- Permite mover **700 TB** dentro do prazo regulatório de **90 dias**

### ♻️ Continuidade Operacional
- O DataSync:
  - Suporta **transferências incrementais**
  - Copia apenas dados alterados após a transferência inicial
- Isso permite que a empresa:
  - Continue acessando os dados
  - Atualize arquivos durante a migração
  - Evite downtime

## 🎯 Benefícios da Solução
- ✅ Migração rápida de grandes volumes de dados
- ✅ Transferência segura e criptografada
- ✅ Nenhuma interrupção das operações
- ✅ Compatível com ambientes híbridos
- ✅ Cumprimento de prazos regulatórios
- ✅ Integração nativa com Amazon S3

## ❌ Por que outras soluções não são ideais?
- ❌ **AWS Snowball / Snowmobile**: indicados quando não há conectividade adequada ou quando não é possível manter acesso aos dados durante a migração
- ❌ Scripts manuais: não escalam bem para centenas de terabytes e aumentam risco operacional
- ❌ Upload via internet pública: pode não atender ao prazo de 90 dias

## 📝 Dica para Provas AWS
Se o cenário mencionar:
- Grande volume de dados (centenas de TB)
- NAS on-premises
- Ambiente híbrido
- Acesso contínuo aos dados
- Direct Connect disponível

👉🏻 Pense imediatamente em **AWS DataSync**

## 📌 Resumo Final
O **AWS DataSync**, combinado com uma conexão **AWS Direct Connect**, é a solução ideal para migrar grandes volumes de dados de forma **rápida, segura e sem interrupções**, permitindo acesso contínuo aos dados durante toda a transferência.

---

👉🏻 [Clique aqui para voltar ao Readme](https://github.com/DrikaDev/Estudando-AWS-Solutions-Architect-Associate/blob/main/README.md) 📒
