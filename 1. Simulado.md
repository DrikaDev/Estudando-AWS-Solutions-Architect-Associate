## Simulado

## Questão 1: DynamoDB – Hot Partitions e Alta Popularidade

Uma empresa está usando o **Amazon DynamoDB** para preparar seu catálogo de produtos, que possui **1 TB** de dados.  
Cada produto tem em média **100 KB**, e o tráfego médio é de **250 solicitações por segundo**.  
O administrador provisionou **3.000 RCUs**, mas alguns itens são muito populares, causando **throttling** e **timeouts**.

Espera-se que a popularidade continue aumentando, mas o número total de produtos permanecerá o mesmo.

## Pergunta
**O que um arquiteto de soluções deve fazer como solução de longo prazo para esse problema?**

### Alternativas
A. Aumentar a taxa de transferência provisionada para 6.000 RCUs.  
B. Usar o DynamoDB Accelerator (DAX) para manter os itens lidos com frequência.  
C. Aumentar o DynamoDB armazenando apenas os principais atributos do produto e usar o Amazon S3 para armazenar os detalhes.  
D. Alterar a chave de partição para que ela tenha um hash da chave do produto e o tipo de produto em vez de apenas a chave do produto.

## ✔️ Resposta Correta
**B — Usar o DynamoDB Accelerator (DAX) para manter os itens lidos com frequência.**

## 🧠 Explicação

### 🔥 Problema: Itens populares (hot keys)
Alguns itens são acessados com muito mais frequência do que os demais, gerando **hot partitions**.  
Mesmo com capacidade provisionada alta, a partição que contém um item popular pode sofrer throttling.

### 📌 Por que o DAX resolve isso?

O **DynamoDB Accelerator (DAX)**:
- armazena em cache itens muito acessados,
- reduz drasticamente a necessidade de leitura no DynamoDB,
- diminui latência,
- elimina throttling de long tail/hot keys,
- é a **solução ideal de longo prazo** para cenários de leitura intensa.

## ❌ Por que as outras opções não são ideais

### A. Aumentar para 6.000 RCUs  
Não resolve hot partitions — só aumenta o custo.  
A partição “quente” continuará sendo o gargalo.

### C. Armazenar detalhes no S3  
Reduz o tamanho do item, mas não resolve o problema de itens populares.  
Adiciona complexidade ao sistema.

### D. Alterar chave de partição  
Mesmo com hash, o item popular continua sendo acessado mais vezes.  
A migração de schema é cara e não resolve o padrão de acesso.

## 📝 Resumo
**Quando houver poucos itens muito populares causando throttling, a solução de longo prazo mais eficiente é usar o DAX para caching de leituras frequentes.**

---

# Questão 2: Projeto de rastreamento de pacotes — escolha de armazenamento com menor TCO

Uma empresa está criando um **aplicativo web em 3 camadas** (servidor web, servidor de aplicação e servidor de banco de dados) para **rastrear coordenadas GPS de pacotes** durante a entrega.  
O aplicativo **atualiza o banco de dados a cada 0,5 segundos**. Leituras de rastreamento precisam ser **muito rápidas** para que usuários verifiquem o status.  

Características do tráfego:
- Alguns dias há poucos pacotes sendo rastreados; outros dias podem ser **milhões de pacotes**.  
- O rastreamento precisa ser pesquisável por **ID de rastreamento**, **ID do cliente** e **ID do pedido**.  
- Pedidos com **mais de 1 mês** não precisam mais ser mantidos.

## Pergunta
**O que um solutions architect deve recomendar para resolver isso com o mínimo de custo total de propriedade (TCO)?**

### Alternativas
A. Usar o Amazon DynamoDB. Ativar o Auto Scaling para a tabela do DynamoDB. Agendar um script de exclusão automática para itens com mais de 1 mês.

B. Usar o Amazon DynamoDB com índices secundários globais. Ativar o Auto Scaling para a tabela do DynamoDB e os índices secundários globais. Ativar o TTL para a tabela do DynamoDB.

C. Usar uma instância sob demanda do Amazon RDS com IOPS provisionados (PIOPS). Configurar alarmes do CloudWatch para notificar quando as PIOPS forem excedidas. Aumentar/diminuir as PIOPS conforme necessário.

D. Usar uma instância reservada do Amazon RDS com IOPS provisionados (PIOPS). Configurar alarmes do CloudWatch para notificar quando as PIOPS forem excedidas. Aumentar/diminuir as PIOPS conforme necessário.

## ✔️ Resposta correta
**B — Usar o Amazon DynamoDB com índices secundários globais. Ativar o Auto Scaling para a tabela do DynamoDB e os índices secundários globais. Ativar o TTL para a tabela do DynamoDB.**

## 🧠 Justificativa

### Por que DynamoDB (escolha B)
- **Escalabilidade elástica**: DynamoDB dimensiona automaticamente para lidar de forma eficiente com cargas variáveis — desde poucos rastreamentos até milhões — especialmente quando combinado com Auto Scaling. Isso reduz TCO porque evita superprovisionamento constante.
- **Latência de leitura/baixa latência de escrita**: É otimizado para leituras e escritas rápidas, adequado para atualizações a cada 0,5 s e leituras rápidas pelo usuário.
- **Consultas por múltiplos atributos**: Crie a tabela com a **chave primária** adequada (por ex. trackingID) e use **Global Secondary Indexes (GSIs)** para consultas por `customerID` e `orderID`. GSIs permitem buscas eficientes sem scans caros.
- **Gerenciamento de dados antigos (economia e conformidade)**: Ative **TTL (Time To Live)** para expirar automaticamente itens com mais de 1 mês — evita scripts manuais, reduz armazenamento e custo operacional.
- **Auto Scaling para GSIs**: Permite que índices também dimensionem conforme o padrão de acesso, evitando throttling nos índices.
- **Menor operação/complexidade**: Menos administração do que gerenciar instâncias de banco (backup, IOPS, tuning), reduzindo TCO.

### Observações operacionais
- Modele a tabela considerando padrões de acesso: chave de partição eficiente (por ex. trackingID) e GSIs para `customerID` e `orderID`.  
- Se houver leituras intensivas e padrões “quentes”, considerar caching (DAX ou cache na camada de aplicação) como otimização adicional.  
- Monitorar com CloudWatch e ajustar Auto Scaling targets (RPS, latência) conforme necessário.

## ❌ Por que as outras opções não são ideais

### A. DynamoDB + script de exclusão manual  
- Funciona, mas **script de exclusão agendado** é operacionalmente mais custoso e sujeito a erros; o TTL nativo é automático, mais barato e mais simples.

### C / D. RDS com PIOPS (on-demand ou reservado)  
- RDS exige **tunar IOPS**, gerenciar instâncias e pode ficar muito caro para picos de milhões de eventos; **não escala tão facilmente** para padrões muito variáveis e introduz maior complexidade operacional. PIOPS também aumenta custo fixo (especialmente se provisionado para pico).

## ✅ Resumo curto
Use **DynamoDB + GSIs + Auto Scaling + TTL** (opção B) — fornece pesquisa eficiente por trackingID/customerID/orderID, escala automaticamente para picos, remove dados antigos automaticamente e resulta no menor TCO e complexidade operacional para esse caso de uso.

---

# Questão 3: Processamento escalável de atualizações de localização

Um novo serviço receberá atualizações de localização de **3.600 carros a cada hora**.  
Cada carro faz upload de sua localização em um **bucket do Amazon S3**.  
Cada atualização deve ser verificada em relação à **distância do local de locação original**.

A solução precisa **processar automaticamente** cada atualização e **escalar automaticamente**.

## Pergunta
**Quais serviços vão processar as atualizações e escalar automaticamente?**

### Alternativas
A. Amazon EC2 e Amazon Elastic Block Store (Amazon EBS)  
B. Amazon Kinesis Data Firehose e Amazon S3  
C. Amazon Elastic Container Service (Amazon ECS) e Amazon RDS  
D. Eventos do Amazon S3 e AWS Lambda  

## ✔️ Resposta Correta
**D — Eventos do Amazon S3 e AWS Lambda**

## 🧠 Explicação

### 🔹 Por que S3 + Lambda é a opção ideal?
- **Eventos do S3** disparam automaticamente uma função **AWS Lambda** sempre que um arquivo é enviado ao bucket.
- O Lambda processa cada atualização (por exemplo, calculando distância), sem necessidade de servidores.
- **Escala automaticamente** conforme o número de uploads aumenta.
- Não há manutenção de infraestrutura — menor custo operacional e econômico.
- Perfeito para cargas esporádicas e baseadas em eventos, como uploads de arquivos.

## ❌ Por que as outras opções não servem bem?

### A. EC2 + EBS  
Exigiria gerenciamento de servidores e não escala automaticamente baseado em uploads do S3.

### B. Kinesis Data Firehose + S3  
Firehose entrega dados ao S3, não processa arquivos nem lê automaticamente dados já no bucket.

### C. ECS + RDS  
Escala, mas é muito mais caro e complexo para um caso simples de disparar processamento por upload.

## 📝 Resumo
**A arquitetura mais simples, automática, serverless e de menor custo é usar eventos do S3 para acionar funções Lambda.**

---

## Questão 4: Active Directory rodando na mesma instância do site — Recomendação de design

Uma empresa identificou, em uma análise do **AWS Well-Architected Framework**, que um site público está sendo executado **na mesma instância EC2** que um **controlador de domínio do Microsoft Active Directory** recém-instalado para suportar outros serviços AWS.  
O objetivo é **melhorar a segurança** da arquitetura e **minimizar a carga administrativa** da equipe de TI.

## Pergunta
**O que o solutions architect deve recomendar?**

### Alternativas
A. Usar o AWS Managed Microsoft AD para criar um Active Directory gerenciado. Desinstalar o Active Directory na instância atual do EC2.  
B. Criar outra instância do EC2 na mesma sub-rede e reinstalar o Active Directory nela. Desinstalar o Active Directory na instância atual do EC2.  
C. Usar o AWS Directory Service para criar um conector do Active Directory. Substituir por proxy as solicitações do Active Directory para o controlador de domínio do Active Directory em execução na instância atual do EC2.  
D. Configurar o AWS Single Sign-On (AWS SSO) com a federação SAML 2.0 com o controlador do Active Directory atual. Modificar o grupo de segurança da instância do EC2 para negar acesso público ao Active Directory.

## ✔️ Resposta correta
**A — Usar o AWS Managed Microsoft AD para criar um Active Directory gerenciado. Desinstalar o Active Directory na instância atual do EC2.**

## 🧠 Justificativa

- **Segurança**: Manter um controlador de domínio em uma instância que também executa um site público aumenta o risco (superfície de ataque, exposição acidental). Remover o AD da instância pública reduz essa superfície imediatamente.
- **Menor sobrecarga operacional**: O **AWS Managed Microsoft AD** é um serviço gerenciado que lida com patches, alta disponibilidade, replicação e backups — reduzindo muito a carga administrativa da equipe de TI.
- **Alta disponibilidade e boas práticas**: Managed AD fornece controladores redundantes e integrações fáceis com serviços AWS (ex.: Amazon RDS for SQL Server, EC2 instances, AWS Single Sign-On) sem a necessidade de gerenciar domain controllers manualmente.
- **Conformidade com Well-Architected**: Move-se a responsabilidade operacional para um serviço gerenciado e separa-se a camada pública (site) da camada de identidade/segurança.

## ❌ Por que as outras opções não são ideais

- **B (nova instância EC2 na mesma sub-rede)**  
  - Ainda exige gerência completa do controlador de domínio (patching, HA, backups).  
  - Se a sub-rede for pública ou houver configuração incorreta, ainda existe risco de exposição. Não reduz significativamente o esforço operacional.

- **C (Directory Service Connector + proxy para o controller na EC2)**  
  - O conector é usado para integrar AD on-premises com serviços AWS; nesse cenário o controlador problemático ainda permanece na instância EC2 (a superfície de ataque não é removida).  
  - Introduz complexidade e não traz os benefícios de um serviço totalmente gerenciado.

- **D (AWS SSO federado + negar acesso público)**  
  - AWS SSO com SAML é útil para gerenciamento de autenticação/federação, mas **não substitui um domínio do Active Directory** para serviços que requerem um AD gerenciado.  
  - Apenas negar acesso público ao grupo de segurança é uma mitigação parcial — não resolve a carga operacional, nem oferece alta disponibilidade/replicação gerenciada.

## ✅ Resumo
A opção **A** (mover para **AWS Managed Microsoft AD** e remover o AD da instância EC2) melhora a segurança ao separar a infraestrutura pública da infraestrutura de identidade e reduz significativamente o trabalho operacional, atendendo aos objetivos do Well-Architected Review.

---

## Questão 5: Permissões seguras para publicar no Amazon SQS a partir de ECS (launch type: EC2)

**Pergunta:** Qual é o método MAIS seguro de conceder permissão à aplicação (rodando no Amazon ECS com *EC2 launch type*) para publicar mensagens no Amazon SQS?

### Alternativas
A. Usar o IAM para conceder permissões do SQS à **função usada pela configuração de lançamento** (instance profile) do Auto Scaling do cluster ECS.  
B. Criar um usuário do IAM com permissões do SQS e colocar as chaves (ID + secret) como variáveis de ambiente na definição da tarefa.  
C. Criar uma **função do IAM** com permissões do SQS e atualizar a definição da tarefa para usar essa função na configuração **task role**.  
D. Atualizar os grupos de segurança do cluster ECS para permitir acesso ao Amazon SQS.

## ✅ Resposta correta
**C — Criar uma função do IAM com permissões do SQS e usar essa função como *task role* na definição da tarefa.**

## 🔒 Justificativa
- A **task role** fornece credenciais temporárias e limitadas **apenas** para a tarefa específica, seguindo o princípio do menor privilégio.  
- Credenciais são rotacionadas automaticamente pelo serviço (via ECS agent), evitando armazenamento manual de chaves secretas.  
- Funciona tanto para *EC2* quanto para *Fargate* (task role é o método recomendado para aplicações em containers).

## ❌ Por que as outras alternativas não são seguras/adequadas
- **A (função do instance profile):** Dar permissões ao *instance profile* (função da instância EC2) concede permissões a **todas** as tarefas e processos na instância — é menos granular e aumenta a superfície de ataque.  
- **B (usuário IAM + chaves em variáveis):** Armazenar credenciais estáticas em variáveis de ambiente é inseguro (risco de vazamento) e exige rotação manual.  
- **D (grupos de segurança):** Grupos de segurança controlam tráfego de rede, não autorizam chamadas de API ao SQS; portanto **não** fornecem permissões IAM.

## ✅ Resumo curto
Use **IAM task role** (definição de tarefa → task role) com políticas mínimas necessárias para publicar no SQS — é a forma mais segura e recomendada.

---

## Questão 6: Coletar e processar cliques da página — 50.000 requisições/seg — processamento sequencial por usuário

Uma empresa recebe **50.000 solicitações por segundo** em seu site.  
Ela quer coletar cliques para **analisar padrões de navegação** e **personalizar a experiência**.  
O requisito principal é: **processar eventos sequencialmente por usuário**.

## Pergunta
**Qual serviço ou recurso da AWS deve ser usado para coletar cliques e processá-los em ordem por usuário?**

### Alternativas
A. Amazon Kinesis Data Streams  
B. Fila padrão do Amazon SQS  
C. Fila FIFO do Amazon SQS  
D. AWS CloudTrail  

## ✔️ Resposta correta
**A — Amazon Kinesis Data Streams**

## 🧠 Explicação

### Por que **Kinesis Data Streams** é a melhor escolha?

- Suporta **altas taxas de ingestão** (50k req/s não é problema).  
- Permite **particionar** eventos por usuário usando `partition key`, garantindo **ordem para cada usuário**.  
- Permite múltiplas aplicações/sistemas consumirem os dados simultaneamente (personalização, analytics, machine learning, dashboards, etc.).  
- Latência de milissegundos e integração com Lambda, Firehose, Analytics, EMR, etc.  
- Solução ideal para **event streaming** de alto volume.

## ❌ Por que as outras opções não são ideais?

### B — SQS padrão  
- Não garante ordem de mensagens.

### C — SQS FIFO  
- Garante ordenação, mas **não escala para 50.000 req/s** (limites muito menores).  
- Não foi feito para ingestão massiva e análises paralelas.

### D — CloudTrail  
- Registra chamadas de API da AWS, **não é** para rastrear eventos de cliques dos usuários.

## 📝 Resumo
Use **Amazon Kinesis Data Streams** para capturar cliques, garantir ordem por usuário via partition key e permitir alto throughput com múltiplos consumidores.

---

## Questão 7: Acesso ao DynamoDB sem que o tráfego saia da rede da AWS

Uma aplicação em uma **sub-rede privada** precisa acessar uma tabela do **Amazon DynamoDB**.  
Requisito: **os dados não podem sair da rede da AWS** (ou seja, sem usar Internet).

## Pergunta
**Como esse requisito deve ser atendido?**

### Alternativas
A. Configurar uma ACL da rede no DynamoDB para limitar o tráfego à sub-rede privada.  
B. Habilitar a criptografia do DynamoDB em repouso usando uma chave do AWS KMS.  
C. Adicionar um NAT Gateway e configurar a tabela de rotas da sub-rede privada.  
D. Criar um endpoint da VPC para o DynamoDB e configurar a política de endpoint.  

## ✔️ Resposta correta
**D — Criar um endpoint da VPC para o DynamoDB e configurar a política de endpoint.**

## 🧠 Explicação

### 🔒 Por que a opção D é a correta?
- O DynamoDB é um serviço **global** que normalmente é acessado pela Internet.  
- Para garantir que **nenhum tráfego saia da rede da AWS**, é necessário criar um **VPC Gateway Endpoint** para o DynamoDB.  
- Com isso, o tráfego da aplicação na sub-rede privada acessa o DynamoDB **internamente**, via rede da AWS, sem usar Internet e sem NAT Gateway.  
- A política do endpoint garante que apenas recursos autorizados possam acessar a tabela.

## ❌ Por que as outras opções não funcionam?

### A — ACL de rede no DynamoDB  
DynamoDB **não usa ACL de rede**, e você não consegue associar ACL diretamente a serviços gerenciados da AWS.

### B — Criptografia em repouso  
A criptografia protege dados armazenados, **não controla o caminho de rede**.  
Não impede que o tráfego vá para a Internet.

### C — NAT Gateway  
O NAT Gateway **usa a Internet** para acessar o DynamoDB.  
Isso vai contra o requisito de manter o tráfego *dentro da rede da AWS*.

## 📝 Resumo
Para garantir que tráfego privado acesse DynamoDB **sem sair da AWS**, crie um **VPC Endpoint para DynamoDB**.

---

## Questão 8: COTA DO EC2 EXCEDIDA — como garantir que todas as instâncias sejam iniciadas?

Um solutions architect encontrou a mensagem **"COTA DO EC2 EXCEDIDA"** ao tentar iniciar instâncias EC2 num Auto Scaling Group.

## ✅ Resposta correta
**B — Usar Service Quotas para solicitar um aumento no número de instâncias do EC2 que a empresa pode iniciar.**

## 🔍 Explicação
- A mensagem indica que o *quota* (limite) de instâncias/recursos EC2 na região foi atingido (por exemplo limite de vCPUs por família, número de instâncias, etc.).  
- **Service Quotas** (ou um chamado ao AWS Support) é a forma correta de **solicitar aumento de limite** para que a AWS permita criar mais instâncias na região.  
- Antes de solicitar o aumento, verifique os **quotas atuais** e o **uso** no console Service Quotas ou no console EC2 (vCPU usage) para identificar qual quota exatamente está sendo atingida.

## ❌ Por que as outras opções estão incorretas
- **A (aumentar número máximo no Auto Scaling Group):** isso apenas altera o limite do próprio ASG, mas **não** altera as cotas da conta/região — se a cota da AWS estiver esgotada, as instâncias ainda não serão iniciadas.  
- **C (recriar o ASG):** não resolve restrição de quota da conta; é uma ação irrelevante para esse erro específico.  
- **D (modificar métrica do CloudWatch):** métricas/alarms controlam quando o ASG escala, mas não alteram cotas da AWS. Alarmes não fazem a conta ter mais capacidade de instância.

## ✅ Passos práticos recomendados
1. Verificar em **Service Quotas** (ou EC2 → Limits / vCPU) qual quota está sendo atingida.  
2. Abrir uma solicitação de aumento de quota via **Service Quotas** console (ou via AWS Support se necessário).  
3. Monitorar o uso e, se apropriado, ajustar a estratégia (usar instâncias de diferentes famílias, regionais, ou otimizadas por custo) para reduzir pressão sobre uma quota específica.

---

# Questão 9 — Criptografia de Backups no Amazon RDS (MySQL)

Uma empresa usa um banco de dados Amazon RDS para MySQL com **backups automatizados não criptografados**.  
Agora, uma auditoria exige:

- Todos os **backups futuros devem ser criptografados**.
- Os **backups antigos não criptografados** devem ser destruídos.
- A empresa fará **pelo menos um backup criptografado antes** de remover os antigos.

## ✅ Resposta Correta  
**C — Criar um snapshot do banco de dados. Copiá-lo para um snapshot criptografado. Restaurar o banco de dados do snapshot criptografado.**

## 🧠 Explicação

No Amazon RDS:

- **A criptografia não pode ser habilitada em uma instância RDS existente.**  
  Essa configuração só pode ser definida **na criação da instância**.
- Para passar uma instância RDS não criptografada para criptografada, o processo recomendado é:

1. **Criar um snapshot manual da instância atual.**  
2. **Copiar o snapshot** e, no momento da cópia, marcar a opção **"Enable Encryption"**.  
3. **Restaurar** um novo DB Instance **a partir do snapshot criptografado**.  
4. Validar e promover o novo banco.  
5. Somente depois, **deletar os antigos backups e snapshots não criptografados**.

Esse é exatamente o fluxo descrito na alternativa **C**.

## ❌ Por que as outras opções estão incorretas?

### A — Ativar criptografia no bucket S3  
Backups automáticos do RDS **não são armazenados diretamente em um bucket S3 gerenciado pelo cliente**.  
Não altera a criptografia do RDS.

### B — Marcar “Habilitar criptografia” na configuração  
A criptografia **não pode ser ativada** em um RDS já existente.  
O botão fica **desativado** após a criação da instância.

### D — Criar uma réplica criptografada e promovê-la  
O MySQL **não suporta** criar uma réplica criptografada a partir de uma instância não criptografada.  
Isso só funciona para alguns mecanismos específicos, e **não** é suportado para MySQL dessa forma.

## 📌 Resumo Curto  
Para criptografar um RDS existente:  
➡️ **snapshot → copiar com criptografia → restaurar banco criptografado**

---

## Questão 10 — Criptografia de imagens no Amazon S3

A empresa deseja armazenar imagens enviadas pelos usuários em um bucket do Amazon S3.  
Requisitos:

- As imagens **devem ser criptografadas em repouso**.
- A empresa **não quer gerenciar ou alternar chaves** manualmente.
- A empresa **quer controlar quem pode acessar as chaves** (controle de permissões).

## ✅ Resposta Correta  
**D — Criptografia do lado do servidor com chaves gerenciadas pelo AWS KMS (SSE-KMS).**

## 🧠 Explicação

O **SSE-KMS** oferece:

- **Criptografia em repouso automática** no Amazon S3  
- **Chaves gerenciadas pelo KMS**, que cuidam de:
  - Rotação automática
  - Armazenamento seguro
  - Auditoria via CloudTrail
- **Controle refinado de acesso** via permissões do IAM e políticas do KMS

Isso atende **exatamente** ao requisito da empresa:  
➡️ **não gerenciar chaves**, mas **controlar quem pode usá-las**.

## ❌ Por que as outras opções estão incorretas?

### A — SSE com chaves armazenadas no S3 (SSE-S3)
- As chaves são totalmente gerenciadas pelo S3.
- **Não permite controlar quem pode acessar as chaves individualmente.**
- Não cumpre o requisito de controle sobre as chaves.

### B — SSE-C (chaves fornecidas pelo cliente)
- O cliente **precisa gerenciar a chave e enviá-la a cada requisição**.
- Exige grande esforço operacional.
- Vai contra o requisito: “não quer gastar tempo gerenciando chaves”.

### C — Chaves no Systems Manager Parameter Store
- Não é usado para criptografia do S3.
- Não oferece rotação automática de chaves para S3.
- Não é uma opção de criptografia suportada pelo S3.

## 📌 Resumo Curto

Para criptografia no S3 com controle de acesso e sem precisar gerenciar chaves:  
➡️ **Use SSE-KMS**.

---

# Questão 11 — Armazenamento para saídas de tarefas ECS com MAIOR taxa de transferência

Um cluster ECS (EC2) em múltiplas AZs precisa que todas as instâncias/leitores acessem os dados de saída/estado das tarefas.  
Cada tarefa gera ~**10 MB**, centenas podem rodar simultaneamente, armazenamento total ≤ **1 TB**.

## Alternativas
A. Tabela do Amazon DynamoDB acessível por todas as instâncias do cluster do ECS.  
B. Volume do Amazon EBS montado nas instâncias do cluster do ECS.  
C. Amazon EFS com **modo de taxa de transferência de intermitência (bursting)**.  
D. Amazon EFS com **modo de taxa de transferência provisionada**.

## ✔️ Resposta correta
**D — Um sistema de arquivos Amazon EFS com o modo de taxa de transferência provisionada.**

## 🧠 Justificativa
- **DynamoDB (A)**: item máximo é **400 KB**, logo não aceita objetos de ~10 MB; portanto não serve.  
- **EBS (B)**: volumes EBS são **ligados a uma única instância** (exceto casos específicos de multi-attach muito restritos) — não é um sistema de arquivos compartilhado entre todas as instâncias do cluster, logo não atende o requisito de acesso compartilhado.  
- **EFS (C vs D)**: EFS é um sistema de arquivos **compartilhado**, acessível por todas as instâncias em várias AZs — adequado funcionalmente.  
  - O **modo de burst (intermitente)** fornece throughput relativo ao tamanho do armazenamento e capacidade de burst, mas pode não garantir a maior taxa sustentável quando há muitas tarefas concorrentes.  
  - O **modo provisionado** permite **especificar throughput independente** do tamanho do sistema de arquivos, garantindo **maior e previsível taxa de transferência** quando necessário — portanto é a escolha para obter a **MAIOR taxa de transferência**.

## ✅ Resumo
Para obter o **maior throughput** com acesso compartilhado entre instâncias EC2 em várias AZs, escolha **Amazon EFS com throughput provisionado** (opção D).

---

## Questão 12 — Comunicação privada entre EC2 e serviços AWS

Um aplicativo em uma VPC precisa acessar outros serviços AWS **na mesma região**, e **nenhum tráfego pode passar pela internet pública**.  
A solução deve ter **a maior eficiência operacional** — ou seja, simples, nativa, sem necessidade de gerenciamento complexo.

## ✔️ Resposta correta
**C — Endpoint de VPC**

## 🧠 Explicação

### Por que **VPC Endpoint** é a melhor opção?
- Permite acesso **privado** de instâncias EC2 para serviços AWS (como S3, DynamoDB, SQS, SNS, etc.).  
- O tráfego **não passa pela internet pública**, permanecendo dentro da rede da AWS.  
- Não requer hardware, VPN, rotas complexas ou administração extra.  
- É totalmente **gerenciado** e de **baixa manutenção**, atendendo ao requisito de **alta eficiência operacional**.

## ❌ Por que as outras opções estão incorretas?

### A — Internet Gateway  
Usado para tráfego **público**.  
👉 Exatamente o oposto do requisito.

### B — NAT Gateway  
Permite que instâncias privadas **acessem a internet pública**.  
👉 Ainda passa pela internet, não atende ao requisito.

### D — AWS Direct Connect  
Cria conexão privada **entre data center local e AWS**.  
- Não é necessário quando tudo já está na AWS.  
- Alto custo e grande sobrecarga operacional.  
👉 Muito além do necessário.

## 📌 Resumo
Para comunicação segura, privada e simples entre EC2 e serviços AWS na mesma região, use **VPC Endpoints (Interface ou Gateway Endpoints)**.

---

## Questão 13 — Infraestrutura imutável + testes antes de enviar tráfego

A empresa quer:

- **Infraestrutura imutável**
- Testar antes de enviar tráfego
- Solução eficiente
- Minimizar impacto de bugs

Esse cenário descreve claramente o padrão **blue/green** ou **canary**, usando duas infraestruturas separadas e rotas controladas, com IaC (CloudFormation).

## ✔️ Respostas corretas
**B — Aplicar roteamento ponderado do Route 53 para testar e aumentar gradualmente o tráfego.**  
**D — Usar CloudFormation com parâmetros para um ambiente separado (staging) fora da produção.**

## 🧠 Explicação

### ✔️ Por que a alternativa **D**?
Infraestrutura imutável significa que você **cria uma nova versão da infraestrutura do zero**, em um **ambiente separado** (por exemplo, *staging*) em vez de atualizar recursos existentes.

O CloudFormation permite:

- Criar ambientes totalmente novos (imutabilidade)
- Testar antes de promover para produção
- Garantir que staging ≈ production

Assim evita-se falhas e impactos.

### ✔️ Por que a alternativa **B**?
O **roteamento ponderado do Route 53** é ideal para:

- Testar a nova versão do ambiente com uma pequena porcentagem do tráfego  
- Aumentar gradualmente o tráfego conforme os testes passam  
- Reduzir impacto de bugs  
- Fazer rollback simplesmente mudando o peso do tráfego

Esse padrão é conhecido como **canary deployment**, amplamente usado com infra imutável.

## ❌ Por que as outras opções estão erradas?

### A — Reverter pilha CloudFormation
Ainda atualiza a infraestrutura atual → **não é infraestrutura imutável**.

### C — Roteamento de failover
Failover envia tráfego *somente quando há falha*, não para testes graduais.  
Não atende ao requisito de testar e aumentar o tráfego gradualmente.

### E — Reutilizar recursos entre ambientes
Vai contra **infraestrutura imutável**.  
Além disso, “reutilizar recursos de produção” = risco alto.

## 📌 Resumo
Para testes seguros com infraestrutura imutável:

- Use CloudFormation para criar **ambientes separados** (staging e prod).  
- Use **Route 53 Weighted Routing** para testar e aumentar o tráfego aos poucos.

➡️ Respostas: **B e D**

---

## Questão 14 — Autenticação segura com MFA em aplicativo móvel

Uma empresa está desenvolvendo um aplicativo móvel onde os clientes farão upload de fotos.  
Requisitos principais:

- Login seguro
- Autenticação multifator (MFA)
- Baixo tempo de implementação
- Baixa manutenção
- Solução gerenciada e ideal para apps móveis

## ✔️ Resposta Correta  
**A — Usar o Amazon Cognito Identity com MFA baseada em SMS.**

## 🧠 Explicação Detalhada

O **Amazon Cognito** é o serviço ideal para autenticação de usuários em aplicativos móveis e web.  
Ele fornece:

- Autenticação e gerenciamento de usuários
- Suporte nativo a **MFA via SMS** e **TOTP**
- Gestão automatizada de tokens
- Recuperação de senha, verificação de e-mail e telefone
- Baixa manutenção, pois é um serviço totalmente gerenciado
- Fácil integração com apps móveis via SDKs da AWS

Isso atende **diretamente** ao requisito de minimizar o esforço de desenvolvimento e manutenção.

## ❌ Por que as outras alternativas estão incorretas?

### **B — Exigir MFA nas políticas do IAM**
- IAM não deve ser usado para login de usuários finais.
- Destinado exclusivamente a **usuários administrativos** da AWS.
- Não atende ao uso em aplicativos.

### **C — Federar IAM com Active Directory corporativo**
- Muito mais complexo de implementar.
- A empresa quer reduzir esforço, não aumentar.
- Não é adequado para clientes externos de um app móvel.

### **D — API Gateway com SSE**
- SSE (Server-Side Encryption) criptografa arquivos no S3.
- Não adiciona login, autenticação ou MFA.
- Não atende ao requisito de autenticação segura.

## 📌 Resumo  
Para login seguro em app móvel + MFA + baixa manutenção:  
➡️ **Use Amazon Cognito com MFA baseada em SMS.**

---

## Questão 15 — Tratamento de falhas entre microserviços com SQS e Dead-Letter Queue

A aplicação foi dividida em dois microsserviços:

- **Microsserviço A** → envia mensagens  
- **Microsserviço B** → consome mensagens

Requisitos:

- Mensagem deve ser processada pelo Microsserviço B.  
- Se falhar **4 vezes**, precisa ser **removida da fila principal** automaticamente.  
- Deve ser armazenada para investigação → **Dead-Letter Queue (DLQ)**.

## ✔️ Resposta correta  
**B — Criar uma fila de mensagens mortas (DLQ) e configurar a fila principal para encaminhar mensagens após 4 recebimentos.**

## 🧠 Explicação Detalhada

O Amazon SQS oferece suporte nativo a **Dead-Letter Queues (DLQs)**.  
Você só precisa:

1. Criar uma fila DLQ.
2. Configurar a **fila principal** com um *redrive policy*:
   - `maxReceiveCount = 4`
   - quando excedido, a mensagem é **automaticamente movida** para a DLQ.

Nenhum microserviço precisa implementar lógica própria.  
Isso garante:

- Menor acoplamento
- Mais confiabilidade
- Comportamento totalmente gerenciado
- Melhor observabilidade para mensagens problemáticas

## ❌ Por que as outras opções estão incorretas?

### **A — O Microsserviço B adiciona mensagens na DLQ**
- A lógica de tratamento de erro não deve ficar no microserviço.
- A DLQ deve ser tratada nativamente pelo SQS.

### **C — O Microsserviço A adiciona mensagens na fila de falha**
- O Microsserviço A não sabe se B falhou.
- Arquiteturalmente incorreto e acoplado.

### **D — Configurar uma fila de falha para extrair mensagens da fila principal**
- Não existe "extração" automática entre filas no SQS.
- Só a fila principal pode enviar mensagens para a DLQ via *redrive policy*.

## 📌 Resumo
Para mover mensagens automaticamente após 4 falhas:  
➡️ **Configure uma DLQ na fila principal com `maxReceiveCount = 4`.**  
Essa é a opção **B**.

---

## Questão 16 – Auto Scaling e Política de Terminação

Um ambiente tem um grupo do Auto Scaling em duas zonas de disponibilidade chamadas **AZ-A** e **AZ-B**.  
- **AZ-A** possui **4 instâncias EC2**  
- **AZ-B** possui **3 instâncias EC2**  

O Auto Scaling usa a **política de terminação padrão**.  
Nenhuma instância está protegida contra scale-in.

**Pergunta:**  
Como o Auto Scaling procederá se houver um evento de redução da escala horizontal?

### ✅ Alternativas

A. O Auto Scaling escolherá uma instância para que ela seja terminada aleatoriamente.  

B. O Auto Scaling terminará a instância com a configuração de lançamento mais antiga entre todas.  

C. O Auto Scaling selecionará a zona de disponibilidade com quatro instâncias do EC2 e, depois, dará prosseguimento à avaliação.  

D. O Auto Scaling terminará a instância com a nova hora de faturamento mais próxima entre todas.

### 🎯 **Resposta Correta: C**

### 📝 **Comentário Explicativo**

A **política de terminação padrão** do Auto Scaling segue esta ordem:

1. **Selecionar a AZ com maior número de instâncias** para manter o balanceamento entre zonas.  
   - AZ-A tem 4 instâncias  
   - AZ-B tem 3 instâncias  
   ➝ **AZ-A será escolhida**

2. Dentro da zona selecionada, avaliar:
   - proteger instâncias? (não tem)
   - configuração de lançamento mais antiga? (apenas se houver múltiplas versões)
   - hora de faturamento mais próxima? (somente relevante para instâncias clássicas on-demand antigas, pouco aplicado hoje)

Mas o primeiro passo é **escolher a AZ com mais instâncias**, o que corresponde exatamente à alternativa **C**.

---

## Questão 17 – Resiliência em conexões híbridas com AWS Direct Connect

Um arquiteto de soluções está projetando uma aplicação híbrida usando a Nuvem AWS.  
A rede entre o datacenter on-premises e a AWS usará uma conexão do **AWS Direct Connect (DX)**.  
A conectividade entre AWS e on-premises deve ser **altamente resiliente**.

**Pergunta:**  
Qual configuração do DX deve ser implementada para atender a esses requisitos?

### ✅ Alternativas

A. Configurar uma conexão do DX com uma VPN sobre ela.  
B. Configurar uma conexão do DX usando o parceiro de DX mais confiável.  
C. Configurar várias interfaces virtuais sobre uma conexão do DX.  
D. Configurar conexões do DX em vários locais do DX.

### 🎯 **Resposta Correta: D**

### 📝 **Comentário Explicativo**

Para obter **alta resiliência**, a AWS recomenda:

- **Ter conexões Direct Connect redundantes**
- **Em locais físicos diferentes (múltiplos DX locations)**  
- Isso elimina pontos únicos de falha e garante que, caso um local tenha problema, o outro mantenha a conectividade.

Vamos analisar as alternativas:

#### ❌ A — Usar VPN sobre DX  
A VPN sobre DX fornece **resiliência lógica**, mas ainda existe **um único ponto de falha físico** no DX. Não atende ao requisito de alta resiliência física.

#### ❌ B — Usar o parceiro mais confiável  
Não garante redundância. Continua sendo **um único DX**.

#### ❌ C — Várias interfaces virtuais (VIFs)  
As VIFs não ajudam se a **conexão física DX** cair.  
Ou seja, **continua não resiliente**.

#### ✅ D — Conexões Direct Connect em múltiplos locais  
Esta é a melhor prática documentada pela AWS.  
Garante **redundância física + redundância lógica**.

---

## Questão 18 — Estratégia de Cache

Uma empresa tem uma solução de arquitetura de três camadas na qual um aplicativo grava em um banco de dados relacional. Devido a solicitações frequentes, a empresa deseja armazenar dados em cache sempre que o aplicativo grava dados no banco de dados. A prioridade da empresa é minimizar a latência para recuperação de dados e garantir que os dados no cache nunca fiquem obsoletos.

## ❓ Qual estratégia de armazenamento em cache a empresa deve usar?

**A**
Amazon ElastiCache com write-through

**B**
Amazon DynamoDB Accelerator (DAX)

**C**
Amazon ElastiCache com carregamento lento

**D**
Amazon Simple Queue Service (Amazon SQS)

## ✅ **Resposta correta: A — Amazon ElastiCache com write-through**

### ✔ Por que essa é a resposta correta?
- O padrão **write-through** grava os dados **no cache e no banco ao mesmo tempo**.
- Isso garante que o dado **nunca fique desatualizado**.
- Proporciona **baixa latência** nas leituras, pois o dado já está no cache no momento da escrita.

## ❌ Por que as outras opções estão erradas?

### **B — DynamoDB Accelerator (DAX)**
- Funciona apenas com **DynamoDB**, mas a questão fala em **banco relacional**.
- Não atende o cenário descrito.

### **C — ElastiCache com lazy loading**
- O cache só atualiza na leitura.
- Pode conter dados **desatualizados (stale)**.
- Não atende ao requisito de "nunca ficar obsoleto".

### **D — Amazon SQS**
- Não é solução de caching; é um serviço de mensageria.
- Não atende à necessidade de redução de latência ou consistência de dados.

---

## 19. Conexão de VPC Peering com provedor externo

Uma empresa está pretendendo usar um serviço de terceiros para análise de aplicações.  
Um arquiteto de soluções configura uma conexão de emparelhamento da VPC entre a VPC da empresa na AWS e a VPC do provedor externo de análise na AWS.

**Pergunta:**  
Que etapa adicional o arquiteto de soluções deve realizar para que o tráfego de rede possa fluir entre as duas VPCs?

### Alternativas

A. Resolver todos os intervalos CIDR sobrepostos.  
B. Configurar as tabelas de rotas para ambas as VPCs.  
C. Verificar se nenhuma das VPC tem outras conexões de emparelhamento.  
D. Verificar se os Internet Gateways estão anexados a cada VPC.

### ✅ **Resposta correta: B — Configurar as tabelas de rotas para ambas as VPCs.**

### ✔ Explicação
Após criar uma conexão de *VPC Peering*, o tráfego **não flui automaticamente** — é necessário adicionar rotas nas tabelas de rotas de **ambas** as VPCs apontando para a conexão de peering.

Sem isso, nenhuma instância consegue alcançar a outra VPC.

### ❌ Por que as outras opções estão incorretas?

**A — Resolver todos os intervalos CIDR sobrepostos**  
- CIDR sobreposto realmente impede a criação do peering, mas não é “etapa adicional”.  
- Só é necessário caso exista sobreposição, o que não foi informado.

**C — Verificar se nenhuma das VPC tem outras conexões de emparelhamento**  
- VPCs podem ter múltiplos peerings, isso não bloqueia o tráfego.  
- O que não é permitido é *roteamento transitivo*, mas isso não se aplica aqui.

**D — Verificar se os Internet Gateways estão anexados**  
- Internet Gateways não são usados em comunicação via VPC Peering.  
- O tráfego entre VPCs é privado e não passa pela internet.

---

## 20. Protegendo acesso direto a ativos em um bucket S3

Uma empresa está projetando um site que será hospedado no Amazon S3.

**Pergunta:**  
Como os usuários devem ser impedidos de acessar diretamente os ativos no bucket do S3?

### Alternativas

A. Criar um site estático e atualizar a política de bucket para exigir que os usuários acessem os recursos com o URL estático do site.  

B. Criar uma distribuição do Amazon CloudFront com um controle do acesso de origem (OAC) e atualizar a política de bucket para conceder permissão somente ao OAC.  

C. Criar um site estático e configurar um conjunto de registros do Amazon Route 53 com um alias apontando para o site estático. Fornecer esse URL aos usuários.  

D. Criar uma distribuição do Amazon CloudFront com uma ACL da Web do AWS WAF que permita acesso ao servidor de origem somente por meio da distribuição.  

### ✅ **Resposta correta: B — Criar uma distribuição CloudFront com OAC e permitir apenas o OAC no bucket.**

### ✔ Explicação

Para impedir que usuários acessem os arquivos **diretamente pelo S3**, é necessário:

- **Criar uma distribuição CloudFront**,  
- **Ativar o Origin Access Control (OAC)**,  
- **Atualizar a política do bucket para permitir acesso apenas ao OAC**.

Assim, os objetos só podem ser acessados **via CloudFront**, nunca diretamente pela URL do S3.

### ❌ Por que as outras opções estão incorretas?

**A — Exigir acesso via URL do site estático**  
- Políticas do S3 **não podem forçar** que o usuário acesse via URL do site.  
- Ainda seria possível acessar diretamente o objeto.

**C — Alias do Route 53**  
- Route 53 apenas resolve DNS; não impede acesso direto ao bucket.

**D — WAF controlando acesso ao servidor de origem**  
- O WAF atua no CloudFront, mas **não controla acessos diretos ao S3**.  
- Só o OAC ou OAI resolvem esse problema.

---

## 21. Como dissociar a arquitetura e torná-la escalável

Uma empresa criou um aplicativo de pedidos de alimentos que captura dados do usuário para análise futura.  
Atualmente:

- Front-end estático → hospedado em EC2  
- Back-end → outra instância EC2  
- Banco → Amazon RDS  
- Front-end envia requisições diretamente ao back-end

A empresa deseja **desacoplar** a arquitetura e permitir **escalabilidade**.

### Alternativas

A. Usar S3 para servir o front-end e enviar solicitações ao EC2 para o back-end, que grava no RDS.  
B. Usar S3 para o front-end ⇒ SNS ⇒ back-end inscrito ⇒ grava no RDS.  
C. Usar EC2 para front-end ⇒ fila SQS ⇒ Auto Scaling baseado na fila ⇒ back-end grava no RDS.  
D. Usar S3 para front-end ⇒ API Gateway ⇒ fila SQS ⇒ Auto Scaling baseado na fila ⇒ back-end grava no RDS.

### ✅ **Resposta correta: D — S3 + API Gateway + SQS + Auto Scaling no back-end**

### ✔ Explicação

A solução mais escalável e desacoplada envolve:

1. **Amazon S3** para hospedar o front-end estático (remoção do EC2 desnecessário).  
2. **Amazon API Gateway** como camada de API totalmente gerenciada.  
3. **Fila Amazon SQS** para desacoplar o front-end e o back-end, absorvendo picos de tráfego.  
4. **Back-end em Auto Scaling**, escalando automaticamente de acordo com o tamanho da fila.  
5. Back-end grava dados no **Amazon RDS**.

Essa arquitetura:

- elimina dependências diretas entre componentes,  
- melhora a escalabilidade,  
- absorve variação de carga,  
- reduz acoplamento entre front-end e back-end.

### ❌ Por que as outras opções estão erradas?

**A — Apenas troca o front-end para S3**  
- Continua acoplado diretamente ao back-end.  
- Não melhora escalabilidade.

**B — SNS não é ideal para processamento contínuo de requisições**  
- SNS não garante reprocessamento ou controle de carga como o SQS.  
- O back-end não escala baseado em tópicos SNS.

**C — Front-end ainda em EC2**  
- Não usa API Gateway.  
- Menos eficiente e menos gerenciado que a opção D.

---

## Questão 22 — Melhorar desempenho de streaming UDP para usuários norte-americanos (solução mais econômica)

Uma empresa transmite áudio por **UDP** hospedado em **eu-central-1** e quer melhorar o desempenho **para usuários na América do Norte**. A solução deve ser a **mais econômica**.

## Alternativas

A. Criar um **AWS Global Accelerator (standard)** com um grupo de endpoints em **eu-central-1**.  
B. Usar CloudFormation para implantar infraestrutura adicional nas regiões **us-east-1** e **us-west-1**.  
C. Criar uma distribuição do **Amazon CloudFront** e usar classes de preço para América do Norte e Europa/Israel.  
D. Configurar o aplicativo para usar uma **policy de roteamento baseada em latência do Route 53**.

## ✅ **Resposta correta: A — Criar um AWS Global Accelerator (standard) com endpoints em eu-central-1**

### Por que A é a melhor (mais econômica e apropriada)
- **Global Accelerator** otimiza tráfego de rede (incluindo **UDP**) usando a rede global da AWS — reduz rota por internet pública e melhora latência/estabilidade para clientes remotos sem precisar duplicar infraestrutura.  
- Não exige implantação de servidores adicionais em regiões dos EUA (economia de custo operacional e de infraestrutura).  
- Fornece IPs estáticos de borda e roteamento pelo backbone AWS até o endpoint em eu-central-1, beneficiando usuários NA com menor perda de pacotes e menor jitter para aplicações UDP.

## ❌ Por que as outras opções não são adequadas

- **B — Implantar infra adicional nas regiões dos EUA**  
  - Melhora latência (por estar fisicamente mais próximo), mas **é muito mais caro** (infra duplicada, gerenciamento, dados entre regiões, failover). Não é a opção *mais econômica*.

- **C — CloudFront**  
  - **CloudFront não suporta tráfego UDP** para streaming de aplicação em tempo real (é orientado a HTTP/S e WebSocket). Logo não atende o requisito de UDP.

- **D — Route 53 latency routing**  
  - Só melhora o roteamento se existirem **endpoints em múltiplas regiões**; não melhora por si só sem infraestrutura adicional nos EUA — então também implica custo extra (sem ser econômico).

## Resumo
Para **streaming UDP** servido a partir de **eu-central-1** com objetivo de **melhorar desempenho para usuários na América do Norte** e **minimizar custo**, use **AWS Global Accelerator (standard)** apontando para o endpoint existente em eu-central-1.  

---

## Questão 23 — Armazenamento seguro de chave de API para AWS Lambda

Uma aplicação em execução no **AWS Lambda** precisa de uma **chave de API** para acessar um serviço de terceiros.  
A chave deve ser armazenada **com segurança** e com **acesso auditado** apenas pela função Lambda.

## Alternativas

A. Como um objeto no Amazon S3  
B. Como uma **string segura no AWS Systems Manager Parameter Store**  
C. Dentro de um arquivo em um volume do Amazon EBS anexado à função Lambda  
D. Dentro de um arquivo de segredos armazenado no Amazon EFS  

## ✅ Resposta correta: **B — Como uma string segura no AWS Systems Manager Parameter Store**

### ✔ Por que essa é a opção mais segura?

- **Parameter Store (SecureString)** permite armazenar segredos de forma criptografada com **KMS**.  
- Fornece **controle de acesso granular via IAM**.  
- Oferece **auditoria completa** com logs no CloudTrail.  
- Integração simples com Lambda, usando SDK.  
- Não requer gerenciar servidores ou storage adicional.

## ❌ Por que as outras opções não são adequadas?

### **A — S3**
- Embora seja possível criptografar objetos, **não é apropriado para segredos**.  
- Controles de acesso são mais complexos e não oferecem auditabilidade tão granular para esse uso.

### **C — EBS**
- Lambda **não pode anexar volumes EBS**.  
- Além disso, seria inseguro e difícil de auditar.

### **D — EFS**
- Pode ser criptografado, mas **não é um repositório de segredos**.  
- Maior superfície de ataque e sem auditabilidade nativa de acesso a segredos.

## 📌 Resumo
A forma **mais segura e recomendada** para armazenar chaves sensíveis usadas por funções Lambda é:

👉 **AWS Systems Manager Parameter Store — SecureString com criptografia KMS**  

---

## Questão 24  
A equipe de operações na nuvem de uma empresa quer padronizar a remediação de recursos.  
A empresa quer fornecer um conjunto padrão de avaliações e remediações de governança para **todas as contas membros** em sua organização no **AWS Organizations**.

Qual serviço **autogerenciado** da AWS pode ser usado para atender a esses requisitos com a quantidade **mínima** de esforço operacional?

### 🔢 Alternativas
A. Padrões de conformidade do **AWS Security Hub**  
B. **Pacotes de conformidade do AWS Config**  
C. **AWS CloudTrail**  
D. **AWS Trusted Advisor**

## ✅ **Resposta Correta: B — Pacotes de conformidade do AWS Config**

### 💡 **Comentário Explicativo**
- Os **AWS Config Conformance Packs** (Pacotes de conformidade) permitem criar **pacotes padronizados** de regras e remediações automáticas utilizando:
  - Regras do AWS Config  
  - Ações de remediação baseadas em Systems Manager  
- É um serviço **autogerenciado** e integra nativamente com **AWS Organizations**, permitindo aplicar o mesmo conjunto de governança e correções para **todas as contas** da organização com **mínimo esforço operacional**.

### ❌ Por que as outras opções estão incorretas?

- **A — AWS Security Hub**  
  Focado em segurança e postura; oferece padrões, mas **não faz remediação padronizada** de recursos nem entrega governança completa como o Config.

- **C — AWS CloudTrail**  
  Serviço de auditoria; não avalia nem remedia configuração de recursos.

- **D — AWS Trusted Advisor**  
  Recomendações operacionais e de custos, mas **não permite criar pacotes padronizados de remediação** nem aplicar a todas as contas via Organizations de forma autogerenciada.

---

## Questão 25  
Atualmente, o aplicativo legado de uma empresa depende de um banco de dados **MySQL do Amazon RDS de instância única sem criptografia**.  
Devido aos novos requisitos de conformidade, **todos os dados existentes e novos** nesse banco de dados devem ser **criptografados**.

Como atender a esse requisito?

### 🔢 Alternativas
A. Criar um bucket do Amazon S3 com criptografia do lado do servidor ativada. Mover todos os dados para o S3. Excluir a instância do RDS.  
B. Configurar o modo Multi-AZ do RDS com criptografia em repouso. Executar o failover para excluir a instância original.  
C. Fazer um snapshot da instância do RDS. Criar uma **cópia criptografada** do snapshot. Restaurar a instância do RDS a partir do snapshot criptografado.  
D. Criar uma réplica de leitura com criptografia. Promover a réplica e mudar o aplicativo. Excluir a instância antiga.

## ✅ Resposta Correta: **C — Criar snapshot → copiar com criptografia → restaurar instância criptografada**

### 💡 Comentário Explicativo
- A criptografia **não pode ser ativada diretamente** em uma instância RDS já existente e sem criptografia.  
- A forma correta é:  
  1. Criar um **snapshot não criptografado** da instância atual.  
  2. Criar **uma cópia criptografada** desse snapshot (com KMS).  
  3. Restaurar uma **nova instância criptografada** a partir do snapshot copiado.  
- Isso garante que **todos os dados existentes e novos** fiquem criptografados, atendendo aos requisitos de conformidade.

### ❌ Por que as outras opções estão incorretas?

- **A — Mover para S3**  
  Totalmente fora do contexto. Não resolve o uso do RDS nem mantém o aplicativo funcionando.

- **B — Multi-AZ com criptografia não funciona**  
  Não é possível **ativar criptografia** em uma instância RDS existente apenas habilitando Multi-AZ.  
  A criptografia precisa ser definida **na criação** da instância.

- **D — Réplica com criptografia**  
  Réplicas **não podem ter criptografia ativada se a instância primária não for criptografada**.  
  Portanto, não é possível criar uma réplica criptografada de uma instância sem criptografia.

---

## Questão 26

Uma empresa quer medir a eficácia das campanhas de marketing recentes. A empresa executa o processamento em batch em arquivos `.csv` de dados de vendas e armazena os resultados em um bucket do Amazon S3 uma vez a cada hora. O bucket do S3 contém petabytes de objetos. A empresa executa consultas únicas no Amazon Athena para determinar quais produtos são mais populares em uma data específica para uma região específica. Às vezes, as consultas falham ou demoram mais do que o esperado para concluir a execução.

**Quais ações um solutions architect deve tomar para melhorar o desempenho e a confiabilidade da consulta?**  
*(Selecione DUAS.)*

- **A.** Reduzir os tamanhos dos objetos do S3 para menos de 128 MB.  
- **B.** Particionar os dados por data e região no Amazon S3.  
- **C.** Armazenar os arquivos como objetos grandes e únicos no Amazon S3.  
- **D.** Usar o Amazon Kinesis Data Analytics para executar as consultas como parte da operação de processamento em batch.  
- **E.** Usar um processo de extração, transformação e carregamento (ETL) do AWS Glue para converter os arquivos `.csv` no formato Apache Parquet.

## ✅ Respostas Corretas: **B** e **E**

### 📌 **B. Particionar os dados por data e região no Amazon S3.**  
**Por quê?**  
O Amazon Athena é um serviço baseado em SQL que lê dados diretamente do S3. Quando os dados são particionados (ex: `s3://bucket/data/year=2025/month=11/day=14/region=SP/`), o Athena **ignora automaticamente partições irrelevantes** durante a consulta. Como as consultas sempre filtram por **data e região**, a partição reduz drasticamente a quantidade de dados escaneados — como se você buscasse um livro em uma biblioteca só nos prateleiros da seção correta, em vez de vasculhar todo o acervo. Isso acelera consultas, reduz custos e melhora a confiabilidade ao evitar timeouts.

> 💡 *Analogia:* É como procurar suas chaves na cozinha, em vez de revirar toda a casa. Partições são os cômodos da sua casa de dados.

### 📌 **E. Usar um processo de extração, transformação e carregamento (ETL) do AWS Glue para converter os arquivos `.csv` no formato Apache Parquet.**  
**Por quê?**  
CSV é um formato textual, não otimizado para análise. O **Apache Parquet** é um formato columnar, comprimido e otimizado para consultas analíticas. Ele:  
- Armazena apenas as colunas necessárias (não lê tudo),  
- Aplica compressão eficiente (reduz I/O),  
- Suporta esquema complexo e tipos de dados fortes.  

Com Parquet, o Athena escaneia **10x a 100x menos dados** do que com CSV — o que resolve tanto lentidão quanto falhas por timeout em grandes volumes. O AWS Glue é a ferramenta ideal para fazer esse ETL de forma serverless e escalável.

> 💡 *Brincadeira mental:* CSV é como ler um livro página por página, mesmo quando você só quer a tabela do final. Parquet é como pular direto para a página da tabela — e ainda está em PDF compactado.

## ❌ Por que as outras estão erradas?

- **A. Reduzir os tamanhos dos objetos para menos de 128 MB**  
  → *Falso!* O Athena funciona melhor com arquivos **maiores** (128 MB a 1 GB). Arquivos muito pequenos geram muitas requisições ao S3 e aumentam a sobrecarga. O ideal é evitar "small file problem".

- **C. Armazenar os arquivos como objetos grandes e únicos**  
  → *Pior ainda!* Um único arquivo de petabytes é inviável: não é paralelizável, impossibilita particionamento e torna qualquer falha catastrófica. O Athena precisa de múltiplos arquivos para escalar.

- **D. Usar o Amazon Kinesis Data Analytics**  
  → *Errado para o cenário.* Kinesis é para **streaming em tempo real**, não para consultas batch em dados históricos armazenados. Aqui, o processamento já é em batch — não há necessidade de mudar a arquitetura para streaming.

## ✅ Conclusão:  
Para consultas rápidas, confiáveis e econômicas no Athena:  
🔹 **Particione os dados** (B) — para reduzir escaneamento.  
🔹 **Use Parquet** (E) — para reduzir I/O e aumentar compressão.  

Essas duas práticas são **best practices fundamentais** em arquiteturas de dados no AWS e transformam consultas lentas em respostas em segundos.  

---

## Questão 27 — Acesso seguro de aplicações EC2 a serviços da AWS na mesma região

Uma aplicação executada em instâncias do **Amazon EC2** precisa publicar informações de identificação pessoal (PII) sobre clientes usando o **Amazon SNS**.  
A aplicação está em **sub-redes privadas** dentro de uma **VPC**.

**Pergunta:**  
Qual é a maneira **mais segura** de permitir que a aplicação acesse endpoints de serviço na **mesma região da AWS**?

### Alternativas

A. Usar um **Internet Gateway**  
B. Usar o **AWS PrivateLink**  
C. Usar um **NAT Gateway**  
D. Usar uma **instância de proxy**

## ✅ **Resposta correta: B — Usar o AWS PrivateLink**

### ✔ Por que essa é a resposta certa?

- **AWS PrivateLink** permite que serviços da AWS (como SNS, SQS, API Gateway, etc.) sejam acessados **privadamente** a partir de sub-redes privadas, sem passar pela internet pública.  
- O tráfego permanece dentro da **rede da AWS**, aumentando a segurança e reduzindo o risco de exposição de PII.  
- Elimina a necessidade de usar NAT ou instâncias de proxy, simplificando a arquitetura e melhorando segurança.

### ❌ Por que as outras opções estão incorretas?

- **A — Internet Gateway**  
  - Permite acesso à Internet pública, o que expõe dados sensíveis.  
  - Não atende ao requisito de acesso **privado**.

- **C — NAT Gateway**  
  - Permite que sub-redes privadas acessem a Internet, mas o tráfego ainda sai da rede privada da AWS, adicionando custo e risco desnecessário.

- **D — Instância de proxy**  
  - Solução gerenciada manualmente, aumenta complexidade operacional e não é necessária com PrivateLink.

---

## Questão 28 — Proteção de acesso a servidores Web e banco de dados RDS

Uma empresa hospeda uma aplicação web conhecida.  
Requisitos:

- Os **servidores Web** devem ser acessíveis **somente via SSL** pelos clientes.  
- O **banco de dados RDS MySQL** deve ser acessível **somente pelos servidores Web**.  
- A solução não deve afetar as aplicações em execução.

### Alternativas

A. Criar ACLs de rede na sub-rede do servidor Web permitindo entrada HTTPS e saída MySQL. Colocar banco e Web na mesma sub-rede.  

B. Abrir porta HTTPS no grupo de segurança dos servidores Web com origem 0.0.0.0/0. Abrir porta MySQL no grupo de segurança do banco e definir a origem como **grupo de segurança dos servidores Web**.  

C. Criar ACLs de rede na sub-rede do servidor Web permitindo entrada HTTPS (origem 0.0.0.0/0). Criar ACLs na sub-rede do banco permitindo entrada porta MySQL apenas dos servidores Web e negar todo tráfego de saída.  

D. Abrir porta MySQL no grupo de segurança para servidores Web com origem 0.0.0.0/0. Abrir porta HTTPS no grupo de segurança do banco com origem grupo de segurança do servidor Web.  

## ✅ **Resposta correta: B — Usar grupos de segurança adequados**

### ✔ Explicação

- **Servidores Web**:
  - Grupo de segurança com **entrada HTTPS (porta 443)** da origem **0.0.0.0/0** (acesso público).  

- **Banco de dados RDS MySQL**:
  - Grupo de segurança com **entrada na porta MySQL (3306)** apenas do grupo de segurança dos **servidores Web**.  
  - Isso garante que somente as instâncias Web podem acessar o banco.  

- **Vantagens**:
  - Segue as melhores práticas da AWS (grupos de segurança stateful).  
  - Sem necessidade de alterar sub-redes ou ACLs de rede.  
  - Não interrompe aplicações existentes.

### ❌ Por que as outras opções estão incorretas?

- **A — ACL de rede e mesma sub-rede**  
  - Colocar banco e Web na mesma sub-rede **não é necessário** e aumenta risco.  
  - ACLs complicam a manutenção e são menos flexíveis que grupos de segurança.

- **C — ACLs de rede com bloqueio de saída**  
  - Negar todo tráfego de saída no banco pode quebrar funcionalidades que requerem conexão de saída.  
  - Mais complexo e difícil de manter.

- **D — MySQL exposto publicamente / HTTPS no banco**  
  - Banco de dados não deve ser acessível publicamente.  
  - HTTPS no banco é incorreto, MySQL não usa SSL para clientes externos nesse cenário.

---

## Questão 29 — Site de backup com failover usando Route 53

Uma empresa executa um site em **instâncias EC2** atrás de um **Application Load Balancer (ALB)**.  
O **Route 53** é usado para DNS.  

Requisito: Configurar um **site de backup** com mensagem e contatos caso o site principal esteja **fora do ar**.

### Alternativas

A. Usar **hospedagem de sites do Amazon S3** para o site de backup e uma **política de roteamento de failover** do Route 53.  

B. Usar S3 para site de backup e **roteamento baseado em latência** do Route 53.  

C. Implantar o aplicativo em outra região da AWS e usar **verificações de saúde do ELB** para failover.  

D. Implantar o aplicativo em outra região e usar **redirecionamento do lado do servidor** no site principal.  

## ✅ **Resposta correta: A — S3 + Route 53 Failover**

### ✔ Explicação

- **Amazon S3** é ideal para **site estático de backup** (mensagem simples, número de telefone e e-mail).  
- **Route 53 Failover Routing** permite que, se o site principal falhar (checado via health check), o tráfego seja **automaticamente redirecionado** para o site de backup no S3.  
- É simples, econômico e confiável.

### ❌ Por que as outras opções estão incorretas?

- **B — Roteamento de latência**  
  - Escolhe região com menor latência, **não detecta falhas** do site principal.  
  - Não atende o requisito de failover.

- **C — Outra região com ELB health checks**  
  - Funciona para alta disponibilidade de aplicações completas, mas **é mais caro** e complexo para um simples site de backup estático.

- **D — Redirecionamento do lado do servidor**  
  - Requer que o site principal esteja funcional para executar o redirecionamento.  
  - **Não funciona se o site principal estiver fora do ar**, que é justamente o caso de falha.

---

## Questão 30 — Otimização de custos em Auto Scaling para site com demanda variável

Uma empresa hospeda seu site na AWS e usa **Auto Scaling** para o front-end de um aplicativo de três camadas.  
Problema: **provisionamento excessivo** causando aumento de custos.

Pergunta: Como otimizar custos **sem afetar o desempenho**?

### Alternativas

A. Usar Auto Scaling com **instâncias reservadas**  
B. Usar Auto Scaling com **política de escalabilidade agendada**  
C. Usar Auto Scaling com **recurso suspender-retomar**  
D. Usar Auto Scaling com **política de scaling com monitoramento de alvo (Target Tracking)**  

## ✅ **Resposta correta: D — Auto Scaling com Target Tracking**

### ✔ Explicação

- **Target Tracking Scaling Policy** ajusta automaticamente a quantidade de instâncias EC2 com base em métricas como **utilização de CPU, requisições por ALB ou outra métrica personalizada**.  
- Permite **otimizar custos**, evitando superprovisionamento.  
- Mantém **desempenho adequado**, pois escala para cima ou para baixo conforme a carga real.

### ❌ Por que as outras opções estão incorretas?

- **A — Instâncias reservadas**  
  - Reduz custo de instâncias de longo prazo, mas não resolve **superprovisionamento automático** ou variabilidade de carga.

- **B — Escalabilidade agendada**  
  - Útil para cargas **previsíveis**, mas não funciona bem para picos imprevisíveis ou demanda altamente variável.

- **C — Suspender-retomar**  
  - Apenas pausa temporariamente políticas de Auto Scaling; **não ajusta dinamicamente** a quantidade de instâncias conforme a demanda.

---

## Questão 31 — Minimizar impacto de endpoints de longa duração em API

Uma empresa tem uma aplicação web que faz chamadas a um **serviço de API backend** executado em instâncias **EC2** atrás de um **ELB**.  

Problema:

- A maioria dos endpoints responde rapidamente.  
- Um endpoint específico **leva muito tempo** para criar objetos em um serviço externo.  
- Isso causa **tempos limite do cliente** e aumenta a **latência geral** do sistema.

### Alternativas

A. Alterar o tamanho da instância EC2 para aumentar memória e CPU.  
B. Usar **Amazon SQS** para descarregar as solicitações de longa duração para **processamento assíncrono** por operadores separados.  
C. Aumentar o **timeout do ELB** para permitir que solicitações longas sejam concluídas.  
D. Usar **ElastiCache for Redis** para armazenar em cache as respostas do serviço externo.

## ✅ **Resposta correta: B — Usar Amazon SQS para processamento assíncrono**

### ✔ Explicação

- **Desacoplar a chamada longa** do fluxo de requisição síncrona reduz **latência aparente** para o cliente.  
- **Amazon SQS** permite colocar a solicitação em uma fila e processá-la **assíncronamente** com workers separados.  
- Benefícios:  
  - Evita **tempos limite do cliente**.  
  - Mantém alta **taxa de transferência** para endpoints rápidos.  
  - Escalabilidade simples e confiável.

### ❌ Por que as outras opções estão incorretas?

- **A — Aumentar tamanho da instância EC2**  
  - Não resolve o problema fundamental: a operação é lenta devido ao **serviço externo**, não à capacidade da instância.

- **C — Aumentar timeout do ELB**  
  - Apenas adia o tempo limite; não melhora a **taxa de transferência geral**.  
  - Pode causar bloqueio de threads e degradação do sistema.

- **D — Usar ElastiCache**  
  - Útil para respostas **frequentemente reutilizadas**, mas neste caso cada solicitação **gera um novo objeto**, então o cache não ajuda.

---

## Questão 32 — Desacoplamento e análise quase em tempo real de cliques de usuários

Uma empresa de mídia tem uma aplicação que rastreia **cliques dos usuários** em seus sites e fornece **recomendações quase em tempo real**.  

Arquitetura atual:

- Frota de **EC2** recebe dados dos sites e envia para **RDS** para retenção de longo prazo.  
- Outra frota de EC2 verifica alterações no banco e executa consultas SQL para recomendações.  

Requisitos:

- **Desacoplar a infraestrutura**.  
- Analistas devem executar SQL apenas nos **novos dados**.  
- Nenhum dado pode ser perdido durante a implantação.  
- Fornecer **acesso mais rápido** à atividade do usuário.

### Alternativas

A. Usar **Kinesis Data Streams** para capturar os dados, **Kinesis Data Firehose** para persistir no S3, e **Amazon Athena** para consultas.  

B. Usar **Kinesis Data Streams** para capturar dados, **Kinesis Data Analytics** para consultas em tempo real, e **Kinesis Data Firehose** para persistir no S3.  

C. Usar **SQS** para capturar dados, manter frota EC2 e aumentar tamanho das instâncias.  

D. Usar **SNS** para receber dados e Lambda para consultas e persistência, migrando RDS para **Aurora Serverless**.  

## ✅ **Resposta correta: B — Kinesis Data Streams + Kinesis Data Analytics + Kinesis Data Firehose**

### ✔ Explicação

- **Kinesis Data Streams** captura dados continuamente, garantindo **nenhuma perda de dados**.  
- **Kinesis Data Analytics** permite **consultas SQL em tempo real** nos dados que chegam, atendendo ao requisito de analisar apenas novos dados.  
- **Kinesis Data Firehose** persiste os dados no S3 para **armazenamento de longo prazo**.  
- Arquitetura **desacoplada**: produtores de dados e consumidores (analytics) são independentes.  
- Alta **taxa de ingestão e baixa latência** para recomendações quase em tempo real.

### ❌ Por que as outras opções estão incorretas?

- **A — Athena**  
  - Athena consulta dados no S3 de forma batch; não fornece **análise quase em tempo real**.  

- **C — SQS + EC2**  
  - Mantém a **infraestrutura acoplada** e não melhora o acesso em tempo real.  
  - Aumento de instâncias não resolve a necessidade de processamento contínuo.

- **D — SNS + Lambda + Aurora Serverless**  
  - Lambda não é ideal para **processamento de grande volume contínuo** de eventos quase em tempo real.  
  - SNS + Lambda pode ter **limitações de taxa e simultaneidade**, aumentando complexidade.

---

## Questão 33 — Lidar com pico de tráfego em site de mídia viral

Uma empresa hospeda um **site de mídia online on-premises**.  
Um usuário publicou uma avaliação com **vídeos e fotos** que viralizou, causando **pico de tráfego**.

Pergunta: Qual ação fornece uma **solução imediata**?

### Alternativas

A. Redesenhar o site para usar **API Gateway + Lambda**  
B. Adicionar instâncias EC2 e usar Route 53 com **failover**  
C. Entregar **imagens e vídeos via Amazon CloudFront** com o site como origem  
D. Usar **ElastiCache for Redis** para cachear e reduzir carga na origem  

## ✅ **Resposta correta: C — Amazon CloudFront**

### ✔ Explicação

- **CloudFront** é uma **CDN global** que entrega conteúdo estático e dinâmico com baixa latência.  
- **Imagens e vídeos** podem ser armazenados em cache em **edge locations**, reduzindo imediatamente a carga no servidor original.  
- Solução **rápida, econômica e escalável** para lidar com tráfego viral sem redesenhar a aplicação.

### ❌ Por que as outras opções estão incorretas?

- **A — API Gateway + Lambda**  
  - Requer **redesign completo**; não é uma solução imediata para picos.  

- **B — Adicionar instâncias EC2**  
  - Demora para provisionar, não resolve a **entrega rápida de conteúdo pesado**.  

- **D — ElastiCache**  
  - Cache melhora performance para dados **frequentemente consultados**, mas não entrega **conteúdo estático como vídeos e imagens** para usuários finais globalmente.

---

## Questão 34 — Redução de custos do Amazon EBS sem downtime

O **Cost Explorer** indica cobranças altas para **volumes EBS io2** conectados a servidores de aplicativos.  
Prioridade: **reduzir custos sem causar downtime**.

### Alternativas

A. Usar **ModifyInstanceAttribute** para habilitar otimização EBS nas instâncias do servidor.  
B. Usar **GetMetricData** do CloudWatch para avaliar operações e bytes de leitura/gravação de cada volume.  
C. Usar **ModifyVolume** para reduzir o tamanho dos volumes io2 subutilizados.  
D. Usar **ModifyVolume** para alterar o tipo de volume dos io2 subutilizados para **gp3**.  
E. Usar **PutBucketPolicy** no S3 para migrar snapshots para **Glacier Flexible Retrieval**.  

## ✅ **Respostas corretas: B e D**

### ✔ Explicação

- **B — Avaliar métricas do CloudWatch**  
  - Permite identificar **volumes subutilizados** com pouca IOPS ou throughput.  
  - Ajuda a tomar decisões baseadas em uso real, evitando alterações desnecessárias.

- **D — Alterar tipo de volume io2 para gp3**  
  - **gp3** oferece desempenho suficiente para a maioria das cargas com **custo menor** que io2.  
  - Pode ser feito **online**, sem downtime.

### ❌ Por que as outras opções estão incorretas

- **A — Habilitar EBS optimization**  
  - Melhora performance, **não reduz custo**.

- **C — Reduzir tamanho do volume**  
  - Nem sempre é possível sem downtime; alteração de tamanho pode causar problemas se houver dados ocupando o espaço reduzido.

- **E — Mover snapshots para Glacier**  
  - Apenas afeta custo de snapshots, não do volume em si.  
  - Não ajuda nos custos contínuos de EBS provisionado para produção.

---

## Questão 35 — Armazenamento compartilhado para múltiplas instâncias EC2

Uma aplicação elástica terá entre 10 e 50 **instâncias EC2**.  
Requisito: cada instância deve **montar e ler/gravar** no mesmo armazenamento de **50 GB**.

### Alternativas

A. Amazon S3  
B. Amazon Elastic File System (Amazon EFS)  
C. Volumes do Amazon Elastic Block Store (Amazon EBS)  
D. Armazenamento de instância do Amazon EC2  

## ✅ **Resposta correta: B — Amazon Elastic File System (EFS)**

### ✔ Explicação

- **Amazon EFS** fornece **sistema de arquivos NFS totalmente gerenciado**.  
- Permite **acesso simultâneo** de múltiplas instâncias EC2.  
- Escala automaticamente com a carga e tamanho dos dados.  
- Ideal para **cenários de armazenamento compartilhado entre instâncias**.

### ❌ Por que as outras opções estão incorretas

- **A — S3**  
  - Armazenamento de objetos, não sistema de arquivos tradicional.  
  - Não pode ser montado como diretório de leitura/gravação simultâneo.

- **C — EBS**  
  - Volumes EBS **não podem ser montados por múltiplas instâncias EC2 simultaneamente** (exceto usando EBS Multi-Attach, que tem restrições e complexidade).

- **D — Armazenamento de instância EC2**  
  - Volátil (perde dados se a instância for encerrada)  
  - Não compartilhado entre instâncias.

---

## Questão 36 — Aumentar taxa de transferência de VPN Site-to-Site

Uma empresa usa **AWS Site-to-Site VPN** para conectividade segura com recursos AWS.  
Problema:

- Aumento de tráfego entre VPNs e EC2.  
- Usuários percebem **conectividade lenta**.  
- Conexão de internet possui **largura de banda disponível não utilizada**.

### Alternativas

A. Implementar múltiplos gateways do cliente na mesma rede.  
B. Configurar **VGW** com roteamento de múltiplos caminhos de custo igual e múltiplos canais.  
C. Usar **Transit Gateway** com roteamento de múltiplos caminhos de custo igual e adicionar outros túneis VPN.  
D. Aumentar o número de túneis na configuração da VPN.

## ✅ **Resposta correta: C — Transit Gateway com múltiplos caminhos de custo igual + túneis adicionais**

### ✔ Explicação

- **AWS Transit Gateway (TGW)** permite agregar várias conexões VPN.  
- **Múltiplos túneis e Equal-Cost Multi-Path (ECMP)** distribuem o tráfego uniformemente.  
- Escala a **largura de banda total** usando a largura de banda disponível da internet.  
- Solução **gerenciável e eficiente** para aumento de throughput sem depender de hardware on-premises adicional.

### ❌ Por que as outras opções estão incorretas

- **A — Vários gateways do cliente**  
  - Não resolve distribuição de tráfego automaticamente; aumenta complexidade.  

- **B — VGW com múltiplos caminhos ECMP**  
  - VGW suporta ECMP, mas tem **limites de throughput menores** que o Transit Gateway para múltiplas conexões.  

- **D — Aumentar número de túneis**  
  - Por si só, sem TGW/ECMP, o tráfego pode não se distribuir eficientemente; **não garante melhor throughput**.

---

## Questão 37

Uma equipe de aplicações começou a usar o Amazon EMR para executar trabalhos em lote, usando conjuntos de dados localizados no Amazon S3. Durante o teste inicial do workload, um arquiteto de soluções percebe que a conta está começando a acumular custos de processamento de dados do NAT gateway.

**Como a equipe pode otimizar os custos do workload?**

- **A.** Desvincular o NAT gateway da sub-rede em que os clusters do Amazon EMR estão sendo executados.  
- **B.** Substituir o NAT gateway por um gateway do cliente.  
- **C.** Substituir o NAT gateway por um endpoint da VPC do S3.  
- **D.** Configurar uma ACL da rede nas sub-redes em que os clusters do Amazon EMR estão sendo executados para abrir o acesso ao Amazon S3.

## ✅ Resposta Correta: **C**

### 📌 **C. Substituir o NAT gateway por um endpoint da VPC do S3.**

**Por quê?**  
Quando instâncias do Amazon EMR (em uma sub-rede privada) acessam o Amazon S3 usando um NAT gateway, todo o tráfego é roteado pela internet (ou pelo NAT), o que **gera custos de processamento de dados** do NAT gateway — e também aumenta latência.

A solução ideal é usar um **VPC endpoint para o S3**, especificamente um **gateway endpoint** (não confundir com interface endpoint). Esse endpoint permite que o tráfego entre as instâncias da VPC e o S3 **nunca saia da rede da AWS**, eliminando:
- Custos de NAT gateway,
- Latência de roteamento externo,
- Dependência de internet.

Além disso, o uso de VPC endpoints para serviços da AWS **não gera cobrança adicional**.

> 💡 **Analogia:** É como construir um túnel direto da sua fábrica (VPC) até o depósito da Amazon (S3), em vez de mandar caminhões (dados) passarem por pedágios (NAT). Sem trânsito, sem pedágio, mais rápido e de graça.

## ❌ Por que as outras estão erradas?

- **A. Desvincular o NAT gateway da sub-rede...**  
  → Se você simplesmente desvincular o NAT gateway **sem fornecer outra rota ao S3**, as instâncias do EMR **perderão conectividade** com o S3 e os trabalhos falharão. Isso não resolve o problema — quebra a aplicação.

- **B. Substituir o NAT gateway por um gateway do cliente.**  
  → “Gateway do cliente” não é um componente padrão da AWS. Pode ser uma confusão com **Customer Gateway**, usado em conexões VPN com data centers locais — **não se aplica aqui**.

- **D. Configurar uma ACL da rede...**  
  → ACLs de rede (Network ACLs) controlam tráfego com base em regras de porta/IP, mas **não eliminam a necessidade de um NAT gateway**. Se o tráfego ainda precisar sair da VPC para acessar o S3 via internet, os custos do NAT permanecem.

## ✅ Conclusão:

Para evitar custos desnecessários com NAT gateway ao acessar serviços da AWS como o S3 a partir de recursos em sub-redes privadas (como clusters do EMR), **sempre use VPC endpoints (gateway endpoints para S3/DynamoDB)**. É uma prática recomendada, segura, gratuita e elimina custos de saída indireta.

> 🚀 **Dica de arquiteto:** Sempre que seu workload dentro da VPC precisar acessar S3, DynamoDB, ou outros serviços compatíveis, pergunte: “Será que já criei o endpoint da VPC?”

---

## Questão 38

Uma empresa tem um aplicativo web de três camadas na AWS para armazenamento e recuperação de documentos. O aplicativo armazena documentos em um volume NFS compartilhado e faz referência a documentos usando uma implantação multi-AZ de uma instância de banco de dados do Amazon RDS para MySQL. Os metadados do documento são consultados regularmente. Os documentos não são acessados mais de uma vez por ano, mas devem permanecer **disponíveis imediatamente**.

Um *solutions architect* precisa otimizar o *workload* e implementar modificações de aplicativos.

**Qual solução atenderá a esses requisitos de maneira MAIS econômica?**

- **A.** Usar um volume compartilhado do Amazon FSx for Lustre para armazenamento de documentos. Usar uma implantação multi-AZ de uma instância de banco de dados do RDS para MySQL para manter os metadados do documento.  
- **B.** Usar um bucket do Amazon S3 com a classe de armazenamento S3 Glacier Deep Archive para armazenamento de documentos. Usar uma tabela do Amazon DynamoDB para manter os metadados do documento.  
- **C.** Usar um bucket do Amazon S3 com a classe de armazenamento S3 Standard-Infrequent Access (S3 Standard-IA) para armazenamento de documentos. Usar uma tabela do Amazon DynamoDB para manter os metadados do documento.  
- **D.** Usar um sistema de arquivos do Amazon Elastic File System (Amazon EFS) com a classe de armazenamento EFS Standard-Infrequent Access (EFS Standard-IA) para armazenamento de documentos. Usar uma implantação multi-AZ de uma instância de banco de dados do RDS para MySQL para manter os metadados do documento.

## ✅ Resposta Correta: **C**

### 📌 **C. Usar um bucket do Amazon S3 com a classe de armazenamento S3 Standard-IA + DynamoDB para metadados**

**Por quê?**  
Vamos analisar os **requisitos-chave**:

| Requisito | Implicação |
|---------|-----------|
| Documentos raramente acessados (<1x/ano) | Baixa frequência de acesso |
| Devem estar **imediatamente disponíveis** | Não pode haver tempo de restauração |
| Necessidade de **otimização de custo** | Solução deve ser a mais econômica possível |

Com isso em mente:

#### 🔹 S3 Standard-IA:
- Ideal para dados **acessados raramente**, mas que precisam estar **prontamente acessíveis**.
- Custo de armazenamento menor que S3 Standard.
- Sem tempo de espera para recuperação (ao contrário do Glacier).
- Perfeito para documentos arquivados que ainda precisam ser abertos "na hora".

> 💡 *Analogia:* É como guardar documentos importantes no sótão da casa, mas com uma escada sempre encostada — você sobe quando precisar, sem esperar ninguém trazer a escada.

#### 🔹 Amazon DynamoDB para metadados:
- Alta performance para consultas frequentes.
- Escalável, serverless e muito mais econômico que RDS para uso intensivo de leitura.
- Ideal para armazenar e consultar metadados rapidamente (ex: nome, tipo, data, localização do documento).

> 💡 *Brincadeira mental:* DynamoDB é como um índice de livro digital — rápido, leve e feito para pesquisas constantes. RDS seria como ler todo o livro toda vez que você quer achar uma palavra.

## ❌ Por que as outras estão erradas?

### **A. Amazon FSx for Lustre**
→ Excelente para alto desempenho (HPC, machine learning), mas **muito cara** para documentos inativos. Não é econômica nem necessária aqui. Além disso, não há benefício em usar FSx para um cenário de baixo acesso.

### **B. S3 Glacier Deep Archive**
→ É a opção mais barata de armazenamento... **mas exige horas para restauração** (até 12h).  
→ **Viola o requisito de disponibilidade imediata.**  
→ Mesmo sendo econômica, **não atende ao SLA funcional.**

> ⚠️ *Glacier Deep Archive é como colocar seus documentos num cofre subterrâneo selado — barato, mas você precisa ligar para a equipe de resgate antes de abrir.*

### **D. Amazon EFS Standard-IA**
→ Parece boa à primeira vista: suporta NFS, tem IA, parece substituir diretamente o volume NFS atual.
→ Mas: **EFS é significativamente mais caro que S3**, mesmo com IA.
→ Além disso, **S3 é mais escalável, durável e econômico** para armazenar objetos estáticos como documentos.
→ Manter RDS para metadados também é desnecessariamente caro comparado ao DynamoDB.

## ✅ Conclusão:

A combinação **S3 Standard-IA + DynamoDB** oferece:
- Disponibilidade imediata dos documentos,
- Baixo custo de armazenamento (ideal para acesso raro),
- Alto desempenho e baixo custo para consultas de metadados.

Essa solução elimina dependência de sistemas de arquivos caros (NFS/EFS) e bancos SQL pesados (RDS), substituindo-os por serviços nativos da nuvem, escaláveis e econômicos.

> 🏆 **Melhor escolha:** Quando o documento é raro, mas importante na hora certa, S3 Standard-IA é o equilíbrio perfeito entre custo e prontidão.

---

## Questão 39

Uma empresa precisa implementar um banco de dados relacional com um **objetivo de ponto de recuperação (RPO) multirregional de 1 segundo** e um **objetivo de tempo de recuperação (RTO) de 1 minuto**.

**Qual solução da AWS pode conseguir isso?**

- **A.** Amazon Aurora Global Database  
- **B.** Tabelas globais do Amazon DynamoDB  
- **C.** Amazon RDS para MySQL com Multi-AZ ativado  
- **D.** Amazon RDS para MySQL com uma cópia de snapshot entre regiões

## ✅ Resposta Correta: **A**

### 📌 **A. Amazon Aurora Global Database**

**Por quê?**  
O **Amazon Aurora Global Database** foi projetado exatamente para cenários de **recuperação de desastres multirregional** com requisitos rigorosos de RPO e RTO:

- **RPO ≤ 1 segundo**: A replicação síncrona dentro da região principal + replicação **quase em tempo real (até 1 segundo de atraso)** para réplicas em outras regiões.
- **RTO < 1 minuto**: Em caso de falha na região primária, é possível **promover uma réplica secundária em outra região em menos de um minuto**, com apenas um comando ou automação via AWS.

Além disso:
- Funciona com **bancos de dados relacionais** (MySQL e PostgreSQL compatíveis).
- Escala globalmente com baixa latência de replicação.
- Totalmente gerenciado pela AWS.

> 💡 **Analogia:** É como ter um gêmeo idêntico morando em outra cidade, que recebe todas as suas atualizações por um walkie-talkie com atraso de meio segundo — e que pode assumir sua vida imediatamente se algo acontecer com você.

## ❌ Por que as outras estão erradas?

### **B. Tabelas globais do Amazon DynamoDB**
- DynamoDB é **não relacional (NoSQL)**.
- A questão especifica claramente que a empresa precisa de um **banco de dados relacional**.
- Apesar de oferecer RPO ≈ 0 e RTO rápido, **não atende ao modelo de dados exigido**.

### **C. Amazon RDS para MySQL com Multi-AZ ativado**
- Multi-AZ replica **dentro da mesma região** (não é multirregional).
- Ótimo para alta disponibilidade local, **mas não protege contra falhas de região inteira** (ex: terremoto, apagão regional).
- **Não atende ao requisito multirregional**.

### **D. Amazon RDS para MySQL com cópia de snapshot entre regiões**
- Snapshots são **pontos no tempo**, geralmente feitos a cada poucas horas.
- RPO seria de **horas**, não de **1 segundo**.
- RTO envolveria restaurar uma instância a partir de um snapshot — processo que leva **minutos a dezenas de minutos**, **não garantindo 1 minuto**.
- Totalmente inadequado para RPO/RTO tão rigorosos.

## ✅ Conclusão:

Para um **banco de dados relacional** com **RPO de 1 segundo** e **RTO de 1 minuto em cenário multirregional**, a **única solução da AWS que atende plenamente** é o **Amazon Aurora Global Database**.

> 🔐 **Dica de arquiteto:** Sempre que vir “recuperação de desastres multirregional + RPO < 1s + RTO < 1 min”, pense primeiro em **Aurora Global Database** — foi feito para isso.

---

## Questão 40

Uma equipe tem uma aplicação que detecta quando é feito upload de novos objetos em um bucket do Amazon S3. Os uploads invocam uma função do AWS Lambda para gravar metadados de objeto em uma tabela do Amazon DynamoDB e em um banco de dados do Amazon RDS para PostgreSQL.

**Que ação a equipe deve realizar para garantir alta disponibilidade?**

- **A.** Habilitar a replicação entre regiões no bucket do S3.  
- **B.** Criar uma função do Lambda para cada zona de disponibilidade em que a aplicação for implantada.  
- **C.** Habilitar o recurso multi-AZ no banco de dados do RDS for PostgreSQL.  
- **D.** Criar um stream do DynamoDB para a tabela do DynamoDB.

## ✅ Resposta Correta: **C**

### 📌 **C. Habilitar o recurso multi-AZ no banco de dados do RDS for PostgreSQL**

**Por quê?**  
O objetivo é **garantir alta disponibilidade (HA)** da aplicação como um todo, especialmente nos componentes críticos.

Analisando os pontos de falha:

- O **Amazon S3** já é altamente durável e disponível por padrão (99,99% de disponibilidade).
- O **AWS Lambda** é serverless e automaticamente distribuído por zonas de disponibilidade — não precisa ser "replicado manualmente".
- O **DynamoDB** é nativamente redundante e altamente disponível por design.
- Já o **Amazon RDS**, por padrão, roda em uma única instância dentro de uma única AZ — se essa instância ou AZ falhar, o banco fica indisponível.

**Habilitar o Multi-AZ no RDS** resolve isso:
- Cria uma réplica síncrona em outra zona de disponibilidade.
- Em caso de falha (instância, AZ, manutenção), o failover automático ocorre em minutos.
- Garante continuidade do serviço ao escrever metadados no PostgreSQL.

> 💡 *Analogia:* É como ter um gerente de plantão em outro andar do prédio: se o elevador travar no seu andar, ele assume imediatamente. Sem ele, você fica parado.

Portanto, **habilitar Multi-AZ no RDS é a ação mais eficaz para aumentar a alta disponibilidade** do workload descrito.

## ❌ Por que as outras estão erradas?

### **A. Habilitar a replicação entre regiões no bucket do S3**
→ O S3 já é globalmente resiliente por padrão (dados replicados entre AZs na mesma região).  
→ A replicação entre regiões (CRR) é útil para **recuperação de desastres (DR)**, não para alta disponibilidade contínua.  
→ Além disso, **não melhora diretamente a disponibilidade do fluxo de processamento** (Lambda → RDS).

### **B. Criar uma função do Lambda para cada zona de disponibilidade**
→ **Incorreto conceitualmente.** O AWS Lambda é um serviço serverless totalmente gerenciado — as funções são **automaticamente distribuídas e executadas em múltiplas zonas de disponibilidade** pela AWS.  
→ Você **não controla nem precisa provisionar** funções por AZ.  
→ Essa opção mostra um mal-entendido sobre como o Lambda opera.

### **D. Criar um stream do DynamoDB para a tabela do DynamoDB**
→ O DynamoDB Streams captura alterações na tabela — útil para disparar Lambda, auditar ou sincronizar dados.  
→ Mas **não contribui para alta disponibilidade do sistema como um todo**, especialmente porque o ponto fraco aqui é o RDS, não o DynamoDB.  
→ É uma funcionalidade adicional, não uma medida de HA.

## ✅ Conclusão:

Embora todos os serviços sejam projetados para resiliência, o **bottleneck de alta disponibilidade neste cenário é o Amazon RDS** — o único componente que, por padrão, **não é automaticamente altamente disponível**.

✅ **Habilitar Multi-AZ no RDS for PostgreSQL** é a **única ação prática e eficaz** listada que **realmente aumenta a disponibilidade do sistema crítico de gravação de metadados**.

> 🛡️ **Dica de arquiteto:** Em qualquer arquitetura com RDS, pergunte sempre: "Está em Multi-AZ?" Se a resposta for não, a alta disponibilidade está comprometida.

---

## Questão 41

Uma aplicação é executada em instâncias do Amazon EC2 em várias zonas de disponibilidade por trás de um Application Load Balancer. O balanceador de carga encontra-se em sub-redes públicas. As instâncias do EC2 estão em sub-redes privadas e **não devem ser acessíveis pela Internet**. As instâncias do EC2 **devem chamar serviços externos na Internet**. **Toda zona de disponibilidade deve poder chamar os serviços externos, independentemente do status das outras zonas de disponibilidade**.

**Como esses requisitos devem ser atendidos?**

- **A.** Criar um NAT gateway anexado à VPC. Adicionar uma rota ao gateway que se conecta a cada tabela de rotas de sub-rede privada.  
- **B.** Configurar um Internet gateway. Adicionar uma rota ao gateway que se conecta a cada tabela de rotas de sub-rede privada.  
- **C.** Criar uma instância NAT na sub-rede privada de cada zona de disponibilidade. Atualizar as tabelas de rotas para cada sub-rede privada a fim de direcionar o tráfego vinculado à Internet à instância NAT.  
- **D.** Criar um NAT gateway em cada zona de disponibilidade. Atualizar as tabelas de rotas para cada sub-rede privada a fim de direcionar o tráfego vinculado à Internet ao NAT gateway.

## ✅ Resposta Correta: **D**

### 📌 **D. Criar um NAT gateway em cada zona de disponibilidade...**

**Por quê?**  
O cenário exige:
1. **Acesso à Internet apenas de saída** (instâncias em sub-redes privadas → serviços externos).
2. **Isolamento de entrada** (instâncias não devem ser acessíveis da Internet).
3. **Alta disponibilidade por zona de disponibilidade** — cada AZ deve funcionar **independentemente** das outras.

O **NAT gateway** é o serviço gerenciado da AWS ideal para acesso de saída de sub-redes privadas. Porém:

> ❗ Um NAT gateway é **implantado em uma única sub-rede (e, portanto, em uma única AZ)**.

Se você tiver instâncias em **múltiplas AZs** e usar **apenas um NAT gateway**, e a AZ desse NAT falhar, **todas as instâncias em outras AZs perdem o acesso à Internet**, violando o requisito de independência por AZ.

✅ A solução é:  
- **Criar um NAT gateway em cada AZ** (em sua respectiva sub-rede pública).  
- **Atualizar a tabela de rotas de cada sub-rede privada** para rotear tráfego `0.0.0.0/0` para o NAT gateway **da mesma AZ**.

Isso garante:
- Alta disponibilidade por zona,
- Tráfego de saída confiável,
- Sem exposição à Internet (sem IPs públicos nas instâncias).

> 💡 **Analogia:** É como ter um tradutor exclusivo em cada terminal de um aeroporto multinacional — se um terminal tiver problema, os outros continuam funcionando normalmente.

## ❌ Por que as outras estão erradas?

### **A. Um único NAT gateway para toda a VPC**
→ Viola o requisito de **independência por AZ**.  
→ Se a AZ do NAT gateway falhar, **todas as instâncias perdem saída para a Internet**, mesmo estando em outras AZs.  
→ **Não é tolerante a falhas por zona.**

### **B. Internet gateway nas sub-redes privadas**
→ Um Internet gateway permite **acesso bidirecional** (entrada e saída).  
→ Para usá-lo, as instâncias precisariam de IP público — o que **viola o requisito de não serem acessíveis da Internet**.  
→ Sub-redes privadas **não devem ter rota direta para Internet gateway**.

### **C. Instância NAT em cada AZ**
→ Embora funcional, **instâncias NAT**:
   - Não são gerenciadas (você cuida de patch, escalonamento, disponibilidade),
   - São mais propensas a falhas,
   - Podem se tornar gargalo de desempenho,
   - **Não são a recomendação atual da AWS** para novas arquiteturas.

> ✅ A AWS recomenda **NAT gateways** (gerenciados e altamente disponíveis dentro da AZ) em vez de instâncias NAT, sempre que possível.

## ✅ Conclusão:

Para atender **alta disponibilidade por zona**, **acesso seguro à Internet** e **isolamento de instâncias**, a melhor prática é:

> **Implantar um NAT gateway em cada zona de disponibilidade** e configurar as rotas das sub-redes privadas correspondentes.

Essa abordagem é **tolerante a falhas, segura, escalável e totalmente gerenciada** — alinhada com os princípios da Well-Architected Framework.

> 🌐 **Dica de arquiteto:** Sempre que usar múltiplas AZs com instâncias privadas que precisam de saída para a Internet, pense: **um NAT gateway por AZ**.

---

## Questão 42

Uma empresa quer implantar um cluster adicional de banco de dados MySQL do Amazon Aurora para desenvolvimento. Esse cluster será usado **várias vezes por semana durante alguns minutos**, mediante solicitação, para depurar problemas de consulta de produção. A empresa quer **manter a sobrecarga baixa** para esse recurso.

**Qual solução atende aos requisitos da empresa de forma MAIS econômica?**

- **A.** Comprar uma instância reservada para as instâncias de banco de dados.  
- **B.** Executar as instâncias de banco de dados no Aurora Serverless.  
- **C.** Criar uma programação de interrupção/início para as instâncias de banco de dados.  
- **D.** Criar uma função do AWS Lambda para interromper as instâncias de banco de dados se não houver conexões ativas.

## ✅ Resposta Correta: **B**

### 📌 **B. Executar as instâncias de banco de dados no Aurora Serverless**

**Por quê?**  
O perfil de uso é crítico aqui:  
- Uso **esporádico** (algumas vezes por semana),  
- Duração **muito curta** (alguns minutos),  
- Necessidade de **baixa sobrecarga operacional** (sem gerenciamento manual de ligar/desligar).

O **Aurora Serverless** (especificamente **Aurora Serverless v2** ou **v1 para MySQL/PostgreSQL compatíveis**) é ideal porque:

- **Escala automaticamente para zero** quando inativo (no caso do Aurora Serverless v1), ou **dimensiona para a capacidade mínima** (v2),
- Você **paga apenas pelo tempo em que o banco está processando consultas** (v1) ou pela capacidade provisionada mínima (v2),
- **Não requer gerenciamento de instâncias**: sem necessidade de ligar/desligar manualmente,
- **Disponível sob demanda**, com inicialização rápida (em segundos),
- **Compatível com MySQL** — atende ao requisito do cluster Aurora MySQL.

> 💡 **Analogia:** Aurora Serverless é como contratar um consultor que aparece **só quando você liga**, resolve o problema em minutos e some — sem pagar por ele ficar sentado na sala de espera.

## ❌ Por que as outras estão erradas?

### **A. Instância reservada**
→ Reservadas são econômicas **somente para uso contínuo** (24/7).  
→ Aqui, o uso é de **minutos por semana** — uma instância reservada geraria **custo fixo alto** por algo quase nunca usado.  
→ **Totalmente inadequada** para carga esporádica.

### **C. Programação de interrupção/início**
→ O **Amazon Aurora (provisionado) não suporta "parar e iniciar"** como o RDS para MySQL/PostgreSQL (até 2025, Aurora não tem estado "stopped").  
→ Esse recurso **não existe para clusters do Aurora** — você não pode simplesmente "desligar" um cluster Aurora e ligá-lo depois.  
→ Mesmo que fosse possível, **programações fixas** não atendem a uso **sob demanda** ("mediante solicitação").

> ⚠️ Aurora ≠ RDS tradicional. Aurora **não para** — ele sempre consome capacidade se provisionado.

### **D. Função Lambda para interromper se não houver conexões**
→ Novamente, **você não pode "interromper" um cluster do Aurora** como faz com instâncias RDS.  
→ Mesmo com scripts inteligentes, **não há estado "stopped" no Aurora** — o cluster continua rodando e gerando custo.  
→ Além disso, adiciona **complexidade operacional**, contrariando o requisito de **baixa sobrecarga**.

## ✅ Conclusão:

Para um banco de dados **usado esporadicamente por poucos minutos**, o **Aurora Serverless** é a **única opção nativa, econômica e de baixa manutenção** na AWS.

- ✅ Paga por uso real (não por tempo ocioso),  
- ✅ Sem necessidade de automação externa,  
- ✅ Escala automaticamente,  
- ✅ Totalmente compatível com MySQL.

> 🚀 **Melhor escolha:** Quando o uso é "quando precisar, por alguns minutos", **Serverless é o caminho**.

---

## Questão 43

Um *solutions architect* é responsável por uma nova arquitetura de três camadas altamente disponível na AWS. Um Application Load Balancer distribui o tráfego para duas zonas de disponibilidade diferentes com um grupo do Auto Scaling que consiste em instâncias do Amazon EC2 e uma instância de banco de dados Multi-AZ do Amazon RDS.  

O *solutions architect* deve recomendar um **plano de recuperação multirregional** com um **objetivo de tempo de recuperação (RTO) de 30 minutos**.  

Devido às **restrições orçamentárias**, o *solutions architect* **não pode recomendar um plano que replique toda a arquitetura**. O plano de recuperação **não deve usar a Região secundária, a menos que seja necessário**.

**Qual estratégia de recuperação de desastres atenderá a esses requisitos?**

- **A.** Backup e restauração  
- **B.** Multissite ativo/ativo  
- **C.** Luz piloto  
- **D.** Standby passivo

## ✅ Resposta Correta: **C**

### 📌 **C. Luz piloto** (*Pilot Light*)

**Por quê?**  
A estratégia **“Luz Piloto”** é um modelo híbrido de recuperação de desastres que:
- Mantém **apenas os componentes essenciais** em funcionamento na região secundária (ex: banco de dados replicado ou snapshots atualizados),
- **Não replica a camada de aplicação completa** (como Auto Scaling groups, ALBs, etc.),
- Permite **recuperar rapidamente** o restante da aplicação **sob demanda**, em caso de falha,
- **Minimiza custos**, pois a infraestrutura secundária permanece quase inativa até ser necessária.

No contexto da questão:
- A aplicação já tem alta disponibilidade em **uma região** (2 AZs, ALB, Auto Scaling, RDS Multi-AZ).
- O RTO requerido é **30 minutos** → factível com uma abordagem "luz piloto", pois:
  - O banco de dados pode ser mantido em sincronia (ex: réplica RDS cross-region ou snapshots frequentes),
  - As instâncias EC2 e o ALB podem ser **implantados rapidamente** a partir de AMIs, templates do CloudFormation ou Terraform, e dados mais recentes do banco.
- A restrição orçamentária **descarta ativo/ativo ou standby completo**.
- A região secundária **só é usada em caso de desastre**, como exigido.

> 💡 **Analogia:** É como ter um carro de corrida desmontado na garagem, mas com o motor ligado e aquecido. Em 30 minutos, você monta o resto e sai acelerando.

## ❌ Por que as outras estão erradas?

### **A. Backup e restauração**
- Usa snapshots ou backups periódicos.
- **RTO normalmente > 1 hora**, às vezes horas ou dias.
- **Não atende ao RTO de 30 minutos**, pois requer restauração completa e reimplantação da infraestrutura.
- Aceitável para cargas de trabalho não críticas, **mas não para RTO rigoroso de meia hora**.

### **B. Multissite ativo/ativo**
- Replica **toda a arquitetura** em duas regiões simultaneamente.
- Oferece RTO ≈ 0, mas **viola a restrição orçamentária** (custo dobrado).
- **Não atende ao requisito de “não usar a região secundária, a menos que necessário”**, pois ela está ativa o tempo todo.

### **D. Standby passivo** (*Warm standby*)
- Mantém uma **cópia em escala reduzida ou completa** da aplicação em outra região, pronta para escalar.
- RTO bom (pode ser < 30 min), **mas mais caro que “luz piloto”**.
- Geralmente envolve manter instâncias EC2 em execução (mesmo em menor número), ALBs, etc.
- **Requer mais infraestrutura na região secundária**, o que **vai contra a restrição de não replicar toda a arquitetura**.

> ⚖️ Comparação:
> - **Luz piloto**: apenas o "coração" (banco de dados) está quente.
> - **Standby passivo**: coração + pulmões (aplicação mínima) estão funcionando.
> - Para RTO de 30 min + baixo custo → **luz piloto é o equilíbrio ideal**.

## ✅ Conclusão:

A estratégia **“Luz Piloto”** atende perfeitamente aos requisitos:
- ✅ RTO de 30 minutos (factível com automação e réplica de banco),
- ✅ Baixo custo (apenas infraestrutura crítica ativa na região secundária),
- ✅ Região secundária é usada **apenas em caso de desastre**,
- ✅ Não replica toda a arquitetura.

> 🔥 **Dica de arquiteto:** Quando o orçamento é apertado, mas o RTO é moderado (15–60 min), **pense em “luz piloto”** — a chama que mantém a esperança acessa sem queimar seu bolso.

---

## Questão 44

**Pergunta:**  
Uma empresa interrompe um cluster de instâncias do Amazon EC2 em um fim de semana. Os custos diminuem, mas eles não caem para zero.  

**Quais recursos ainda poderiam estar gerando custos? (Selecione DUAS respostas.)**

**Alternativas:**

- **A. Endereços de IP elásticos** ✅  
  *Comentário:* Mesmo que a instância esteja parada, o Elastic IP associado à conta ainda gera cobrança se não estiver em uso em uma instância em execução.

- **B. Transferência de dados de saída**  
  *Comentário:* Custos de transferência de dados normalmente ocorrem durante o tráfego de rede. Se a instância está parada, não há tráfego, então não gera custo.

- **C. Transferências de dados regionais**  
  *Comentário:* Sem instâncias ativas, transferências regionais não ocorrem, portanto não geram custos.

- **D. Volumes do Amazon Elastic Block Store (Amazon EBS)** ✅  
  *Comentário:* Volumes EBS permanecem provisionados mesmo quando a instância está parada, e continuam gerando cobrança.

- **E. AWS Auto Scaling**  
  *Comentário:* O Auto Scaling em si não gera custos; ele apenas gerencia instâncias EC2 que são cobradas individualmente.

**Respostas corretas:** A e D

---

## Questão 45

**Pergunta:**  
Um solutions architect descobre que um cluster do Amazon Aurora com definição de preço de instância sob demanda está sendo subutilizado para um aplicativo de blog. O aplicativo é usado apenas por alguns minutos várias vezes ao dia para leituras.  

**O que um solutions architect deve fazer para otimizar a utilização de forma MAIS econômica?**

**Alternativas:**

- **A. Ativar o Auto Scaling no banco de dados original do Aurora**  
  *Comentário:* O Auto Scaling ajusta recursos, mas ainda mantém instâncias provisionadas, o que pode não ser a forma mais econômica para uso esporádico.

- **B. Refatorar o aplicativo de blog para usar a consulta paralela do Aurora**  
  *Comentário:* Isso pode melhorar desempenho, mas não reduz custos para um banco subutilizado.

- **C. Converter o banco de dados Aurora original em um banco de dados global do Aurora**  
  *Comentário:* Bancos globais são úteis para replicação entre regiões, mas não otimizam custos para uso esporádico.

- **D. Converter o banco de dados original do Aurora em Amazon Aurora Serverless** ✅  
  *Comentário:* Aurora Serverless ajusta automaticamente a capacidade de acordo com a demanda e cobra apenas pelo tempo em que o banco está ativo, sendo ideal para cargas esporádicas como este blog.

**Resposta correta:** D

---

## Questão 46

**Pergunta:**  
Um arquiteto de soluções está projetando uma solução de banco de dados que deve comportar uma alta taxa de leitura e gravação aleatórias de disco. Ela deve fornecer performance consistente e requer persistência de longo prazo.  

**Qual solução de armazenamento atende a esses requisitos?**

**Alternativas:**

- **A. Um volume de IOPS provisionadas do Amazon Elastic Block Store (Amazon EBS)** ✅  
  *Comentário:* Volumes de IOPS provisionadas (io1/io2) oferecem alta performance consistente para cargas de trabalho com muitas leituras e gravações aleatórias e garantem persistência de longo prazo.

- **B. Um volume de uso geral do Amazon Elastic Block Store (Amazon EBS)**  
  *Comentário:* Volumes gp2/gp3 são adequados para cargas de trabalho gerais, mas podem não oferecer performance consistente suficiente para taxas muito altas de I/O aleatório.

- **C. Um volume magnético do Amazon Elastic Block Store (Amazon EBS)**  
  *Comentário:* Volumes magnéticos (standard) são mais lentos e não oferecem performance consistente para aplicações de alta demanda.

- **D. Um armazenamento de instância do Amazon EC2**  
  *Comentário:* Armazenamento de instância é rápido, mas não é persistente — os dados são perdidos quando a instância é interrompida ou terminada.

**Resposta correta:** A

---

## Questão 47

**Pergunta:**  
Uma empresa quer criar uma versão em áudio do manual do produto. O manual do produto contém nomes e abreviações de produtos personalizados. O manual do produto está dividido em seções.  

**Qual solução atenderá a esses requisitos com a MENOR quantidade de custos operacionais indiretos?**

**Alternativas:**

- **A. Usar o Amazon Polly. Criar léxicos personalizados para os nomes e abreviações dos produtos. Usar a operação da API StartSpeechSynthesisTask para cada seção do manual do produto.** ✅  
  *Comentário:* Amazon Polly é ideal para converter texto em fala. A criação de léxicos personalizados permite pronunciar corretamente nomes e abreviações. StartSpeechSynthesisTask permite processar cada seção em lote, minimizando esforço operacional.

- **B. Usar o Amazon Polly. Criar uma linguagem de marcação de síntese de fala (SSML) personalizada para os nomes e abreviações dos produtos. Usar a operação da API StartDocumentTextDetection para cada seção do manual do produto.**  
  *Comentário:* StartDocumentTextDetection pertence ao Amazon Textract (OCR), não é usado para síntese de fala. SSML sozinha não resolve o processamento em lote.

- **C. Usar o Amazon Textract. Criar uma linguagem de marcação de síntese de fala (SSML) personalizada para os nomes e abreviações dos produtos. Usar a operação da API StartDocumentTextDetection para cada seção do manual do produto.**  
  *Comentário:* Textract é usado para extrair texto de documentos, não para gerar áudio. Não atende ao requisito.

- **D. Usar o Amazon Textract. Criar léxicos personalizados para os nomes e abreviações dos produtos. Usar a operação da API StartTranscriptionJob para cada seção do manual do produto.**  
  *Comentário:* TranscriptionJob é usado para converter áudio em texto, não o contrário. Não atende ao requisito.

**Resposta correta:** A

---

## Questão 48

**Pergunta:**  
Uma empresa quer criar uma aplicação que transmitirá informações de saúde protegidas (PHI) para milhares de consumidores de serviços em diferentes contas da AWS. Os servidores da aplicação ficarão em sub-redes privadas da VPC. O roteamento da aplicação deve ser tolerante a falhas.  

**O que deve ser feito para atender a esses requisitos?**

**Alternativas:**

- **A. Criar um serviço de endpoint da VPC e conceder permissões aos consumidores de serviços específicos para criar uma conexão.** ✅  
  *Comentário:* Criar um **AWS PrivateLink** (serviço de endpoint da VPC) permite que os consumidores acessem os serviços de forma segura através de interfaces de rede privadas, sem expor tráfego à Internet. Escalável e tolerante a falhas.

- **B. Criar uma conexão de gateway privado virtual entre cada par de VPCs do provedor de serviços e VPCs do consumidor de serviços.**  
  *Comentário:* VPNs site-to-site ou gateways privados não são práticos para milhares de consumidores; gerenciar todas essas conexões seria complexo e sujeito a erros.

- **C. Criar um Application Load Balancer interno na VPC do provedor de serviços e colocar os servidores da aplicação por trás dele.**  
  *Comentário:* Um ALB interno distribui tráfego dentro da VPC, mas não permite acesso seguro direto de múltiplas contas externas.

- **D. Criar um servidor de proxy na VPC do provedor de serviços para encaminhar solicitações de consumidores de serviços para os servidores da aplicação.**  
  *Comentário:* Um servidor de proxy central se torna um ponto único de falha e não escala bem para milhares de consumidores.

**Resposta correta:** A

---

## Questão 49

**Pergunta:**  
Um arquiteto de soluções está projetando um novo workload no qual uma função do AWS Lambda acessará uma tabela do Amazon DynamoDB.  

**Qual é o meio MAIS seguro para conceder à função do Lambda acesso à tabela do DynamoDB?**

**Alternativas:**

- **A. Criar uma função do IAM com as permissões necessárias para acessar a tabela do DynamoDB. Atribuir a função à função Lambda.** ✅  
  *Comentário:* Esta é a prática recomendada. O Lambda assume a função do IAM, evitando o uso de credenciais estáticas, garantindo segurança e gerenciamento simplificado de permissões.

- **B. Criar um nome de usuário e uma senha do DynamoDB e fornecê-los ao desenvolvedor para que sejam usados na função do Lambda.**  
  *Comentário:* DynamoDB não usa usuários e senhas; essa abordagem não é suportada e é insegura.

- **C. Criar um usuário do IAM e chaves secretas e de acesso para o usuário. Conceder ao usuário as permissões necessárias para acessar a tabela do DynamoDB. Pedir para o desenvolvedor usar essas chaves ao acessar os recursos.**  
  *Comentário:* Usar chaves de acesso estáticas é menos seguro que atribuir uma função do IAM diretamente ao Lambda, pois aumenta o risco de exposição de credenciais.

- **D. Criar uma função do IAM que conceda acesso pelo AWS Lambda. Atribuir a função à tabela do DynamoDB.**  
  *Comentário:* Funções do IAM não são atribuídas a tabelas; elas são assumidas por serviços (como Lambda) que precisam acessar recursos.

**Resposta correta:** A

---

## Questão 50

**Pergunta:**  
Uma aplicação de reserva de mesa em restaurante precisa acessar uma lista de espera. Quando um cliente tenta reservar uma mesa e não há nenhuma disponível, a aplicação do cliente colocará o usuário na lista de espera e enviará uma notificação quando uma mesa estiver livre. A lista de espera deve manter a ordem em que os clientes foram adicionados.  

**O arquiteto de soluções deve recomendar qual serviço para armazenar essa lista de espera?**

**Alternativas:**

- **A. Amazon Simple Notification Service (Amazon SNS)**  
  *Comentário:* SNS é um serviço de publicação/assinatura para enviar notificações, mas não armazena a lista de espera nem garante ordem.

- **B. AWS Step Functions invocando funções do AWS Lambda**  
  *Comentário:* Step Functions orquestram workflows, mas não são adequadas para armazenar e manter a ordem de uma lista de espera.

- **C. Uma fila FIFO no Amazon Simple Queue Service (Amazon SQS)** ✅  
  *Comentário:* Fila FIFO (First-In-First-Out) mantém a ordem de processamento das mensagens e é ideal para listas de espera que precisam de ordem garantida.

- **D. Uma fila padrão no Amazon Simple Queue Service (Amazon SQS)**  
  *Comentário:* Filas padrão do SQS não garantem ordem de entrega; podem processar mensagens fora de ordem.

**Resposta correta:** C

---

## Questão 51

**Pergunta:**  
Uma aplicação fornece um recurso que permite aos usuários baixar arquivos privados e pessoais com segurança. Atualmente, o servidor Web está sobrecarregado com o fornecimento de arquivos para download. Um arquiteto de soluções deve encontrar uma solução mais eficaz para reduzir a carga e o custo do servidor Web e permitir que os usuários baixem apenas seus próprios arquivos.  

**Qual solução atende a todos os requisitos?**

**Alternativas:**

- **A. Armazenar os arquivos com segurança no Amazon S3 e possibilitar que a aplicação gere um URL pré-assinado do Amazon S3 para o usuário baixar.** ✅  
  *Comentário:* URLs pré-assinados permitem que os usuários acessem apenas seus próprios arquivos sem sobrecarregar o servidor Web. S3 fornece armazenamento seguro, escalável e econômico.

- **B. Armazenar os arquivos em um volume criptografado do Amazon Elastic Block Store (Amazon EBS) e usar um conjunto separado de servidores para fornecer os downloads.**  
  *Comentário:* Embora seguro, exige gerenciamento de servidores adicionais e não é tão escalável ou econômico quanto S3 com URLs pré-assinados.

- **C. Possibilitar que a aplicação criptografe os arquivos e os armazene no armazenamento de instância local do Amazon EC2 antes de enviá-los para download.**  
  *Comentário:* Armazenamento de instância é volátil e não persistente, o que torna essa abordagem insegura e não escalável.

- **D. Criar uma distribuição do Amazon CloudFront para distribuir e armazenar os arquivos em cache.**  
  *Comentário:* CloudFront melhora desempenho e cache, mas sozinho não garante que apenas usuários autorizados possam acessar seus próprios arquivos. Normalmente usado em conjunto com S3 e URLs pré-assinados.

**Resposta correta:** A

---

## Questão 52

**Pergunta:**  
Um aplicativo web é executado em instâncias do Amazon EC2 atrás de um Application Load Balancer (ALB). O aplicativo permite que os usuários criem relatórios personalizados de dados meteorológicos históricos. A geração de um relatório pode levar até 5 minutos. Essas solicitações de longa duração usam muitas das conexões de entrada disponíveis, fazendo com que o sistema não responda a outros usuários.  

**Como um solutions architect pode tornar o sistema mais responsivo?**

**Alternativas:**

- **A. Usar o Amazon SQS com o AWS Lambda para gerar relatórios.** ✅  
  *Comentário:* Colocar a geração de relatórios em uma fila SQS permite que as solicitações sejam processadas de forma assíncrona, liberando o ALB e as instâncias EC2 para atender outros usuários, tornando o sistema mais responsivo.

- **B. Aumentar o tempo limite de inatividade no ALB para 5 minutos.**  
  *Comentário:* Isso apenas mantém as conexões abertas por mais tempo, mas não resolve o problema de bloqueio de recursos durante o processamento.

- **C. Atualizar o código do aplicativo do lado do cliente para aumentar o tempo limite da solicitação para 5 minutos.**  
  *Comentário:* Isso não melhora a capacidade de resposta do servidor nem resolve a sobrecarga das conexões.

- **D. Publicar os relatórios no Amazon S3 e usar o Amazon CloudFront para fazer download para o usuário.**  
  *Comentário:* Pode ajudar na distribuição dos relatórios, mas não resolve o problema de geração de relatórios de longa duração bloqueando o servidor.

**Resposta correta:** A

---

## Questão 53

**Pergunta:**  
Uma empresa está desenvolvendo uma solução de data lake no Amazon S3 para analisar conjuntos de dados em grande escala. A solução faz apenas consultas SQL infrequentes. Além disso, a empresa quer minimizar os custos de infraestrutura.  

**Qual produto da AWS deve ser usado para atender a esses requisitos?**

**Alternativas:**

- **A. Amazon Athena** ✅  
  *Comentário:* Athena permite consultar dados diretamente no S3 usando SQL sem necessidade de provisionar servidores ou clusters. É cobrado por consulta, o que é ideal para consultas infrequentes e ajuda a minimizar custos.

- **B. Amazon Redshift Spectrum**  
  *Comentário:* Redshift Spectrum permite consultar dados no S3, mas exige um cluster Redshift ativo, o que gera custos fixos mesmo para consultas infrequentes.

- **C. Amazon RDS for PostgreSQL**  
  *Comentário:* RDS é um banco de dados relacional gerenciado, mas não é otimizado para consultar grandes volumes de dados armazenados em S3. Também gera custos contínuos de instância.

- **D. Amazon Aurora**  
  *Comentário:* Aurora é um banco de dados relacional de alta performance, mas não é ideal para análise direta de grandes datasets em S3 e gera custos contínuos de infraestrutura.

**Resposta correta:** A

---

## Questão 54

**Pergunta:**  
Um arquiteto de banco de dados está projetando um aplicativo de jogos on-line que usa um formato de dados simples e não estruturado. O banco de dados deve ter a capacidade de armazenar informações do usuário e rastrear o progresso de cada usuário. O banco de dados deve ter a capacidade de dimensionar para milhões de usuários ao longo da semana.  

**Qual serviço de banco de dados atenderá a esses requisitos com a MENOR quantidade de suporte operacional?**

**Alternativas:**

- **A. Multi-AZ do Amazon RDS**  
  *Comentário:* Multi-AZ RDS oferece alta disponibilidade para bancos de dados relacionais, mas não é ideal para dados não estruturados e para escalabilidade massiva com mínimo suporte operacional.

- **B. Amazon Neptune**  
  *Comentário:* Neptune é um banco de dados de grafos; não é necessário para armazenamento simples de dados de usuários e progresso de jogo.

- **C. Amazon DynamoDB** ✅  
  *Comentário:* DynamoDB é um banco de dados NoSQL totalmente gerenciado, altamente escalável e de baixa manutenção. É ideal para dados simples e não estruturados e pode escalar automaticamente para milhões de usuários.

- **D. Amazon Aurora**  
  *Comentário:* Aurora é um banco de dados relacional de alta performance, mas requer gerenciamento de clusters e não é tão eficiente quanto DynamoDB para dados não estruturados e escalabilidade massiva.

**Resposta correta:** C

---

## Questão 55

**Pergunta:**  
Uma empresa implementou um microsserviço no AWS Lambda que acessa uma tabela do Amazon DynamoDB chamada “Books”. Um solutions architect está projetando uma política do IAM para ser anexada à função do perfil do IAM do Lambda, dando-lhe acesso para colocar, atualizar e excluir itens na tabela “Books”. A política do IAM deve impedir que a função execute outras ações na tabela “Books” e em qualquer outra tabela.  

**Qual política do IAM atenderia a essas necessidades e forneceria o acesso com MENOS privilégio?**

**Alternativas:**

- **A.**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PutUpdateDeleteOnBooks",
            "Effect": "Allow",
            "Action": [
                "dynamodb:PutItem",
                "dynamodb:UpdateItem",
                "dynamodb:DeleteItem"
            ],
            "Resource": "arn:aws:dynamodb:us-west-2:123456789012:table/Books"
        }
    ]
}
```
✅ Comentário: Esta política segue o princípio do menor privilégio: permite apenas PutItem, UpdateItem e DeleteItem na tabela Books e não concede acesso a outras ações ou tabelas.

B. 
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PutUpdateDeleteOnBooks",
            "Effect": "Allow",
            "Action": [
                "dynamodb:PutItem",
                "dynamodb:UpdateItem",
                "dynamodb:DeleteItem"
            ],
            "Resource": "arn:aws:dynamodb:us-west-2:123456789012:table/*"
        }
    ]
}
```
Comentário: Permite acesso a todas as tabelas na conta, não atende ao requisito de menor privilégio.

C.
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PutUpdateDeleteOnBooks",
            "Effect": "Allow",
            "Action": "dynamodb:*",
            "Resource": "arn:aws:dynamodb:us-west-2:123456789012:table/Books"
        }
    ]
}
```
Comentário: Permite todas as ações na tabela Books, o que concede privilégios desnecessários além do solicitado.

D.
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PutUpdateDeleteOnBooks",
            "Effect": "Allow",
            "Action": "dynamodb:*",
            "Resource": "arn:aws:dynamodb:us-west-2:123456789012:table/Books"
        },
        {
            "Sid": "PutUpdateDeleteOnBooks",
            "Effect": "Deny",
            "Action": "dynamodb:*:*",
            "Resource": "arn:aws:dynamodb:us-west-2:123456789012:table/Books"
        }
    ]
}
```
Comentário: Contém regras redundantes e confusas; o mesmo efeito é obtido com a opção A, de forma mais clara.

Resposta correta: A

---

## Questão 56

**Pergunta:**  
Um arquiteto de soluções precisa permitir que os desenvolvedores tenham conectividade SSH com servidores Web. Os requisitos são os seguintes:

- Limite de acesso aos usuários originados da rede corporativa.  
- Os servidores Web não podem ter acesso ao SSH diretamente pela Internet.  
- Os servidores Web residem em uma sub-rede privada.  

**Qual combinação de etapas o arquiteto deve realizar para atender a esses requisitos? (Selecione DUAS respostas.)**

**Alternativas:**

- **A. Criar um bastion host que autentique os usuários no diretório corporativo.** ✅  
  *Comentário:* Um bastion host permite acesso seguro à sub-rede privada e pode autenticar usuários usando o diretório corporativo, garantindo que apenas usuários autorizados consigam se conectar.

- **B. Criar um bastion host com regras de grupo de segurança que só permitam o tráfego da rede corporativa.** ✅  
  *Comentário:* Limitar o acesso SSH ao bastion host apenas à rede corporativa atende ao requisito de segurança.

- **C. Anexar uma função do IAM ao bastion host com permissões relevantes.**  
  *Comentário:* Funções do IAM controlam permissões de AWS, mas não limitam o acesso SSH; não atende diretamente ao requisito.

- **D. Configurar o grupo de segurança dos servidores Web para permitir o tráfego de SSH de um bastion host.**  
  *Comentário:* Embora seja uma boa prática, sozinho não atende ao requisito principal de limitar acesso à rede corporativa, mas normalmente é usado em conjunto com A e B.

- **E. Negar todo tráfego de SSH da rede corporativa na ACL da rede de entrada.**  
  *Comentário:* Negar tráfego seria contrário ao requisito de permitir acesso via bastion host.

**Respostas corretas:** A e B

---

## Questão 57

Uma empresa analisa uma recente migração de uma aplicação de três camadas para uma VPC. A equipe de segurança descobre que o **princípio de menor privilégio** não está sendo aplicado às regras de entrada e saída de grupos de segurança do Amazon EC2 entre as camadas da aplicação.

**O que um arquiteto de soluções deve fazer para corrigir esse problema?**

- **A.** Criar regras do grupo de segurança usando o ID da instância como origem ou destino.  
- **B.** Criar regras do grupo de segurança usando o ID do grupo de segurança como origem ou destino.  
- **C.** Criar regras do grupo de segurança usando os blocos CIDR da VPC como origem ou destino.  
- **D.** Criar regras do grupo de segurança usando os blocos CIDR da sub-rede como origem ou destino.

## ✅ Resposta Correta: **B**

### 📌 **B. Criar regras do grupo de segurança usando o ID do grupo de segurança como origem ou destino.**

**Por quê?**  
O **princípio do menor privilégio** exige que cada componente tenha apenas as permissões mínimas necessárias para funcionar.

Na AWS, ao configurar comunicação entre camadas (ex: web → aplicação → banco de dados), a melhor prática é:

> 🔐 Usar o **ID do grupo de segurança** (não IP, não CIDR) como origem em regras de entrada/saída.

#### Exemplo:
- Camada web pode acessar a camada de aplicação na porta 80.
- Em vez de permitir tráfego de qualquer IP (`0.0.0.0/0`) ou mesmo de toda a VPC,
- Você permite tráfego **apenas de instâncias que pertencem ao grupo de segurança da camada web**.

Isso garante:
- ✅ Comunicação segura entre camadas,
- ✅ Isolamento automático (instâncias fora do grupo não podem se comunicar),
- ✅ Escalabilidade: novas instâncias herdam automaticamente as regras de segurança,
- ✅ Adesão ao princípio do menor privilégio.

> 💡 **Analogia:** É como dar acesso a um cofre só para pessoas com um tipo específico de crachá — não importa quem seja ou onde esteja, se não tiver o crachá certo, não entra.

## ❌ Por que as outras estão erradas?

### **A. Usar o ID da instância como origem ou destino**
→ O ID da instância (ex: `i-1234567890abcdef0`) **não pode ser usado diretamente em regras de grupo de segurança** como origem/destino.
→ Grupos de segurança não aceitam IDs de instância como parâmetro válido.
→ **Tecnicamente inválido** → não funciona.

### **C. Usar blocos CIDR da VPC**
→ Permite tráfego de **toda a faixa de IPs da VPC**.
→ Muito amplo: inclui instâncias que talvez não façam parte da camada autorizada.
→ Viola o menor privilégio → "todo mundo da empresa pode entrar no servidor de produção".

### **D. Usar blocos CIDR da sub-rede**
→ Um pouco mais restrito que a VPC inteira, mas ainda assim **muito amplo**.
→ Qualquer instância (mesmo não autorizada) que esteja naquela sub-rede terá acesso.
→ Não é baseado em função ou papel — é baseado em localização de rede.
→ Pode permitir acesso por instâncias de outros sistemas ou ambientes.

## ✅ Conclusão:

Para aplicar o **princípio do menor privilégio** entre camadas de uma aplicação na AWS, a abordagem mais segura e nativa é:

> **Usar o ID do grupo de segurança como origem/destino nas regras do EC2 Security Group.**

Essa técnica é conhecida como **"security group referencing"** e é amplamente recomendada pela AWS.

> 🛡️ **Dica de arquiteto:** Sempre que possível, conecte camadas por **grupo de segurança**, nunca por IP ou CIDR. É mais seguro, mais flexível e escala naturalmente com Auto Scaling.

---

## Questão 58

Uma empresa está usando o **Amazon DynamoDB com taxa de transferência provisionada** para a camada de banco de dados de inventário de seu site de comércio eletrônico. Durante as **ofertas relâmpago**, os clientes enfrentam períodos em que o banco de dados não consegue atender ao alto número de transações, resultando em **perda de transações**. Durante períodos normais, o banco de dados funciona adequadamente.

**Qual solução resolve o problema de performance que a empresa enfrenta?**

- **A.** Alternar o DynamoDB para o modo sob demanda durante as ofertas relâmpago.  
- **B.** Implementar o DynamoDB Accelerator (DAX).  
- **C.** Usar o Amazon Kinesis a fim de enfileirar as transações para processamento no DynamoDB.  
- **D.** Usar o Amazon Simple Queue Service (Amazon SQS) para enfileirar as transações no DynamoDB.

## ✅ Resposta Correta: **A**

### 📌 **A. Alternar o DynamoDB para o modo sob demanda durante as ofertas relâmpago.**

**Por quê?**  
O problema é claro: **picos extremos e imprevisíveis de tráfego** durante eventos curtos (ofertas relâmpago), onde o **throughput provisionado do DynamoDB se esgota**, causando throttling e perda de transações.

#### O modo **sob demanda (on-demand)** do DynamoDB foi feito exatamente para esse cenário:
- Escala automaticamente para suportar **milhões de solicitações por segundo**,
- Você paga por **cada operação executada**, sem precisar provisionar capacidade,
- **Elimina throttling** causado por picos inesperados,
- Pode ser alternado dinamicamente entre os modos **provisionado** e **sob demanda** (sem downtime).

> 💡 **Analogia:** É como trocar um plano de internet fixo (com limite de banda) por um plano ilimitado só nos dias de pico — você só paga mais quando realmente precisa.

Como as ofertas relâmpago são **eventos programados ou previsíveis**, a empresa pode:
1. Mudar para **modo sob demanda antes do evento**,
2. Voltar ao modo provisionado após o pico (para controlar custos),
3. Evitar completamente perda de transações.

✅ Solução simples, nativa, eficaz e alinhada com o padrão AWS.

## ❌ Por que as outras estão erradas?

### **B. Implementar o DynamoDB Accelerator (DAX)**
→ DAX é um cache de leitura em memória.
→ **Acelera apenas operações de LEITURA**, não gravação.
→ O problema aqui é **alta carga de transações (gravações)** durante o pico — DAX **não ajuda nesse caso**.
→ Além disso, não elimina o throttling na escrita.

> ⚠️ Útil para escalar leituras, mas **não resolve gargalos de escrita**.

### **C. Usar o Amazon Kinesis para enfileirar transações**
→ Kinesis é ótimo para ingestão em tempo real, mas **não garante entrega única nem ordem sequencial para bancos de dados**.
→ Requer um processo de consumo complexo (ex: Lambda) para escrever no DynamoDB.
→ Pode introduzir **latência e complexidade desnecessária**.
→ Não é a solução ideal para transações de e-commerce que exigem **resposta imediata ao usuário**.

### **D. Usar o Amazon SQS para enfileirar transações**
→ Parece uma boa ideia à primeira vista: enfileirar e processar depois.
→ Mas: **transações de inventário precisam ser processadas em tempo real**.
→ Se você adia a atualização do estoque, corre o risco de **vender produtos fora de estoque**.
→ Além disso, **SQS não é idempotente por padrão** — pode gerar duplicatas.
→ Viola a expectativa de consistência imediata em sistemas de comércio eletrônico.

> 🛑 Enfileirar transações críticas de inventário = abrir mão da integridade do sistema.

## ✅ Conclusão:

Para cargas de trabalho com **picos intensos e breves**, como ofertas relâmpago, o **modo sob demanda do DynamoDB** é a **solução mais simples, segura e eficaz**.

> ✅ Use **provisionado** para cargas estáveis.  
> ✅ Use **sob demanda** para cargas imprevisíveis ou com picos.  

A AWS permite alternar entre os modos conforme necessário — aproveite essa flexibilidade!

> 🎯 **Dica de arquiteto:** Automatize a mudança para "on-demand" antes de eventos promocionais (usando EventBridge + Lambda) e volte ao provisionado depois. Assim, você tem desempenho garantido **e** controle de custos.

---

## Questão 59

Um aplicativo de fotos online permite que os usuários façam o upload de fotos e realizem operações de edição de imagens. O aplicativo é executado em instâncias do Amazon EC2 e oferece duas classes de serviço: **gratuito** e **pago**. As fotos enviadas por usuários **pagos são processadas antes** das enviadas por usuários gratuitos. As fotos são enviadas para o Amazon S3 e as informações do trabalho são enviadas para o **Amazon SQS**.

**Qual configuração um solutions architect deve recomendar?**

- **A.** Usar uma fila do SQS FIFO. Atribuir uma prioridade mais alta às fotos de usuários pagos para que elas sejam processadas primeiro.  
- **B.** Usar duas filas do SQS FIFO: uma para usuários pagos e outra para usuários gratuitos. Definir a fila gratuita para usar a sondagem curta e a fila paga para usar a sondagem longa.  
- **C.** Usar duas filas padrão do SQS: uma para usuários pagos e outra para usuários gratuitos. Configurar o aplicativo nas instâncias do Amazon EC2 para priorizar a sondagem para a fila paga em relação à fila gratuita.  
- **D.** Usar uma fila padrão do SQS. Definir o tempo limite de visibilidade das fotos de usuários pagos para zero. Configurar o aplicativo nas instâncias do Amazon EC2 para priorizar as configurações de visibilidade para que as fotos de usuários pagos sejam processadas primeiro.

## ✅ Resposta Correta: **C**

### 📌 **C. Usar duas filas padrão do SQS: uma para usuários pagos e outra para usuários gratuitos. Configurar o aplicativo nas instâncias do Amazon EC2 para priorizar a sondagem para a fila paga em relação à fila gratuita.**

**Por quê?**  
O requisito é claro: **prioridade de processamento para usuários pagos** — ou seja, **não é sobre ordem dentro da fila, mas sobre qual fila é processada primeiro**.

#### Por que isso funciona:
- **Duas filas separadas** (uma para pagos, outra para gratuitos) permitem isolamento lógico e controle de prioridade.
- **SQS padrão (standard)** é suficiente — não há necessidade de garantia de ordem (FIFO), pois cada foto é um trabalho independente.
- O aplicativo nas instâncias EC2 pode ser programado para:
  - **Sondar a fila de usuários pagos primeiro** (ex: 3x mais frequentemente),
  - Só quando a fila paga estiver vazia, sondar a fila gratuita.
- Isso garante que **trabalhos pagos sejam processados com prioridade**, sem precisar de recursos complexos como FIFO ou prioridade nativa (que SQS padrão não suporta).

> 💡 **Analogia:** É como ter duas filas no banco: uma para clientes VIP e outra para clientes comuns. O atendente sempre atende o VIP primeiro — mesmo que o comum tenha chegado antes.

#### Vantagens:
- ✅ Simples e eficaz,
- ✅ Baixo custo (SQS padrão é mais barato que FIFO),
- ✅ Escalável e confiável,
- ✅ Não depende de funcionalidades que o SQS não oferece (como prioridade por mensagem).

## ❌ Por que as outras estão erradas?

### **A. Usar uma fila FIFO e atribuir prioridade**
→ **SQS FIFO não suporta prioridade por mensagem.**  
→ FIFO garante **ordem estrita** e **entrega exatamente uma vez**, mas **não permite priorizar mensagens** dentro da mesma fila.  
→ Mesmo que você tente codificar prioridade no corpo da mensagem, o SQS FIFO **não a respeita** — ele processa na ordem de envio.  
→ **Não resolve o problema.**

### **B. Duas filas FIFO + sondagem curta/longa**
→ Usar **FIFO é desnecessário** — não há necessidade de ordem entre trabalhos de edição de fotos.
→ **Sondagem curta/longa** afeta a frequência de polling, mas **não garante prioridade**.  
→ Se a fila paga tiver sondagem longa e a gratuita tiver sondagem curta, o resultado será **o oposto do desejado**!
→ Além disso, FIFO é mais caro e complexo — **não justificado**.

### **D. Uma fila padrão + tempo de visibilidade zero**
→ **Tempo de visibilidade zero** não é uma configuração válida — o mínimo é 0 segundos, mas isso **não prioriza** mensagens.
→ SQS padrão **não tem mecanismo de prioridade** — todas as mensagens são tratadas de forma igual, independentemente de quando ou como foram enviadas.
→ O conceito de “priorizar configurações de visibilidade” **não existe** no SQS.
→ Essa opção confunde conceitos técnicos e é **incorreta**.

## ✅ Conclusão:

A melhor solução para **priorizar processamento de trabalhos** com base em classe de serviço (pago vs. gratuito) é:

> **Usar duas filas SQS padrão** — uma para cada grupo — e **implementar a lógica de prioridade no consumidor (EC2)**, sondando a fila paga com maior frequência.

É simples, barato, escalável e **alinhado com as melhores práticas da AWS**.

> 🎯 **Dica de arquiteto:** Quando precisa de prioridade, **separe as filas**. Quando precisa de ordem, use FIFO. Não force recursos que não existem — o SQS padrão + lógica de aplicação é a combinação mais elegante para esse cenário.

---

## Questão 60

Uma equipe de desenvolvimento está implantando um novo produto na AWS e usando o **AWS Lambda** como parte da implantação. A equipe aloca **512 MB de memória** para uma das funções do Lambda. Com essa alocação de memória, a função é concluída em **dois minutos (120 segundos)**. A função é executada **milhões de vezes mensalmente**, e a equipe de desenvolvimento está preocupada com o **custo**. A equipe realiza testes para ver como diferentes alocações de memória do Lambda afetam o custo da função.

**Quais etapas reduzirão os custos do Lambda para o produto?**  
*(Selecione DUAS respostas.)*

- **A.** Aumentar a alocação de memória dessa função do Lambda para 1.024 MB se essa alteração reduzir o tempo de execução de cada função para menos de um minuto.  
- **B.** Aumentar a alocação de memória dessa função do Lambda para 1.024 MB se essa alteração reduzir o tempo de execução de cada função para menos de noventa segundos.  
- **C.** Diminuir a alocação de memória dessa função do Lambda para 256 MB se essa alteração reduzir o tempo de execução de cada função para menos de quatro minutos.  
- **D.** Aumentar a alocação de memória dessa função do Lambda para 2.048 MB se essa alteração reduzir o tempo de execução de cada função para menos de um minuto.  
- **E.** Diminuir a alocação de memória dessa função do Lambda para 256 MB se essa alteração reduzir o tempo de execução de cada função para menos de cinco minutos.

## ✅ Respostas Corretas: **A** e **D**

### 📌 **A. Aumentar a alocação de memória para 1.024 MB se reduzir o tempo de execução para menos de um minuto.**  
### 📌 **D. Aumentar a alocação de memória para 2.048 MB se reduzir o tempo de execução para menos de um minuto.**

**Por quê?**  
O custo do Lambda é calculado com base em:

Custo = (Número de requisições + Unidades de Confiabilidade) * Preço por requisição
Custo de computação = (Memória alocada * Tempo de execução) * Preço por GB-segundo

Ou seja: **mais memória = mais custo por segundo, mas tempo de execução pode ser reduzido**, impactando o custo total.

#### Vamos calcular os custos relativos para a configuração atual:
- **Atual:** 512 MB por 120 segundos = `512 * 120 = 61.440 MB-segundos = 60 GB-segundos`.

#### Avaliando as opções:

#### ✅ **A: 1.024 MB por <60 segundos**
- Pior caso: 1.024 MB * 60 segundos = `61.440 MB-segundos = 60 GB-segundos`.
- **Igual ao custo atual**, mas se for **menos de 60 segundos**, o custo **cai**.
- **Pode reduzir custo**.

#### ✅ **D: 2.048 MB por <60 segundos**
- Pior caso: 2.048 MB * 60 segundos = `122.880 MB-segundos = 120 GB-segundos`.
- O dobro da configuração base, **mas ainda dentro de um minuto**, o que reduz o tempo de execução pela metade.
- Pode ser **mais eficiente em custo** se o desempenho for significativamente melhor e o tempo de execução cair substancialmente.
- **Depende da otimização do código**, mas **pode reduzir custo total**.

## ❌ Por que as outras estão erradas?

#### **B: 1.024 MB por <90 segundos**
- 1.024 MB * 90 segundos = `92.160 MB-segundos`.
- Maior que o custo base de 61.440 MB-segundos.
- **Aumenta custo**, não reduz.

#### **C: 256 MB por <240 segundos**
- 256 MB * 240 segundos = `61.440 MB-segundos`.
- **Igual ao custo atual**, **não reduz**.

#### **E: 256 MB por <300 segundos**
- 256 MB * 300 segundos = `76.800 MB-segundos`.
- **Maior que o custo atual**, **aumenta custo**.

## ✅ Conclusão:

Para reduzir custos em funções Lambda executadas milhões de vezes:
- **Aumentar memória pode reduzir custo total se o tempo de execução cair proporcionalmente**.
- A chave é encontrar o **ponto ideal entre memória e tempo**.
- Opções que dobram a memória e **reduzem o tempo pela metade** (ou mais) tendem a **reduzir o custo total**.

> 🧮 **Dica de custo Lambda:**  
> `Custo = Memória (GB) * Tempo (segundos)`  
> Portanto, `1 GB * 60 s = 0.5 GB * 120 s` — o custo é o mesmo.  
> Mas `1 GB * 50 s = 50` é **menor que** `0.5 GB * 120 s = 60`.

> 🚀 **Melhor prática:** Teste diferentes configurações de memória e monitore o tempo de execução. Ajuste para o ponto de menor custo total.

---

## Questão 61

Uma empresa executa uma aplicação em três instâncias muito grandes do Amazon EC2 em uma única zona de disponibilidade na região `us-east-1`. Vários volumes do Amazon Elastic Block Store (Amazon EBS) de **16 TB** são anexados a cada instância do EC2. A equipe de operações usa um script do AWS Lambda acionado por uma regra do Amazon EventBridge baseada em programação para **interromper as instâncias nas noites e nos fins de semana** e **iniciar as instâncias nas manhãs dos dias da semana**.

Antes de implantar a solução, a empresa usou a documentação pública de preço da AWS para estimar os custos gerais da execução dessa solução de data warehouse **cinco dias por semana durante dez horas por dia**. Ao examinar as cobranças mensais do Cost Explorer para essa nova conta, as cobranças gerais são **maiores do que a estimativa**.

**Qual é o fator de custo MAIS provável que a empresa negligenciou?**

- **A.** As cobranças de transferência de dados do EC2 entre as instâncias são muito maiores do que o esperado.  
- **B.** As taxas do EC2 e do EBS são mais altas na região `us-east-1` do que na maioria das outras Regiões da AWS.  
- **C.** As cobranças do Lambda para interromper e iniciar as instâncias são muito maiores do que o esperado.  
- **D.** A empresa está sendo cobrada pelo armazenamento do EBS nas noites e nos fins de semana.

## ✅ Resposta Correta: **D**

### 📌 **D. A empresa está sendo cobrada pelo armazenamento do EBS nas noites e nos fins de semana.**

**Por quê?**  
A empresa **interrompe as instâncias EC2** (ou seja, desliga as máquinas virtuais), mas **não desanexa nem exclui os volumes EBS**.

> 🔑 **Importante:** O **Amazon EBS é cobrado por armazenamento, independentemente do estado da instância EC2.**

Mesmo quando a instância está **interrompida (stopped)**:
- Os volumes EBS permanecem **anexados e ativos**,
- A AWS continua cobrando pelo **armazenamento do EBS** (por GB/mês),
- E **não há desconto** por tempo de inatividade da instância.

#### Cálculo aproximado do custo negligenciado:
- 3 instâncias × 16 TB = **48 TB de EBS**
- 48 TB = 48 × 1.024 = ~49.152 GB
- Custo do EBS gp3: ~$0.08/GB/mês
- Custo mensal de armazenamento:  
  `49.152 GB × $0.08 = ~$3.93 por dia`  
  → **$118 por mês** só em armazenamento EBS

Mas isso é só o começo.

Se a empresa estimou custos **somente para o tempo em que as instâncias estavam ligadas** (5 dias × 10h = 50h/semana), **esqueceu que os volumes EBS estão sendo cobrados 24/7** — ou seja, **30 dias × $3.93 = $118**, mesmo sem as instâncias rodando.

> 💡 **Analogia:** É como alugar um apartamento e desligar a luz, mas ainda pagar pela manutenção do prédio todos os dias — mesmo quando você não está lá.

#### Comparação com estimativa:
- Estimativa da empresa: **custo da instância EC2 apenas durante 50h/semana**
- Realidade:  
  - EC2: cobrado apenas durante 50h/semana ✅  
  - EBS: cobrado **24/7** ❌ — e **isso representa o maior custo residual**

## ❌ Por que as outras estão erradas?

### **A. Transferência de dados entre instâncias**
→ As instâncias estão na mesma AZ e não estão rodando simultaneamente (interrompidas à noite).  
→ Não há tráfego de rede significativo entre elas.  
→ Transferência de dados **dentro da mesma AZ é gratuita**.  
→ **Não é o fator principal.**

### **B. Taxas mais altas na região us-east-1**
→ `us-east-1` é uma das regiões **mais baratas** da AWS, não mais caras.  
→ É a região mais popular e com maior economia de escala.  
→ **Incorreto factualmente.**

### **C. Cobranças do Lambda para interromper/iniciar**
→ Lambda é **muito barato**:  
  - Milhões de execuções mensais = ~$0.20 a $0.50 por mês (mesmo com 2000 chamadas/dia).  
  - O custo de EventBridge também é mínimo.  
→ **Insignificante** comparado ao custo de 48 TB de EBS.  
→ Não explica a discrepância significativa observada.

## ✅ Conclusão:

A empresa **esqueceu que o armazenamento EBS é cobrado mesmo quando as instâncias EC2 estão interrompidas**.  

Com 48 TB de EBS, o custo mensal de armazenamento pode facilmente ultrapassar **$100**, enquanto o custo da instância EC2 durante 50h/semana pode ser bem menor — especialmente se for uma instância de grande porte com desconto por uso intermitente.

> 🚨 **Dica de arquiteto:**  
> Sempre lembre-se:  
> - **EC2** é cobrado por tempo de execução.  
> - **EBS** é cobrado por armazenamento — **24/7**, mesmo quando a instância está parada.  
>  
> Para reduzir custos em ambientes de desenvolvimento ou data warehouse com uso intermitente:  
> ✅ **Interrompa as instâncias**  
> ✅ **Mas também considere criar snapshots e excluir volumes EBS quando não forem necessários**, ou usar **EBS Snapshots + criação dinâmica de volumes** em vez de manter volumes permanentes.

> 💡 **Melhor prática:** Para cargas de trabalho intermitentes, use **EBS Snapshots** e **crie volumes novos a partir de snapshots ao iniciar** — assim você paga apenas pelo armazenamento dos snapshots (mais barato que volumes ativos).

---

## Questão 62

Durante uma análise de aplicações empresariais, um arquiteto de soluções identifica uma aplicação crítica com banco de dados relacional que foi construída por um usuário empresarial e está sendo executada **no desktop do usuário**. Para reduzir o risco de interrupção dos negócios, o arquiteto de soluções quer **migrar a aplicação para uma solução de várias camadas e altamente disponível na AWS**.

**O que o arquiteto de soluções deve fazer para conseguir isso com interrupções MÍNIMAS nos negócios?**

- **A.** Criar um pacote de importação do código da aplicação para upload no AWS Lambda e incluir uma função com o objetivo de criar outra função do Lambda para migrar dados para um banco de dados do Amazon RDS.  
- **B.** Criar uma imagem do desktop do usuário e migrá-la para o Amazon EC2 usando o VM Import. Colocar a instância do EC2 em um grupo do Auto Scaling.  
- **C.** Preparar antecipadamente novas instâncias do Amazon EC2 executando o código da aplicação na AWS por trás de um Application Load Balancer e uma instância de banco de dados multi-AZ do Amazon RDS.  
- **D.** Usar o AWS Database Migration Service (AWS DMS) para migrar o banco de dados de backend para uma instância de banco de dados multi-AZ do Amazon RDS. Migrar o código da aplicação para o AWS Elastic Beanstalk.

## ✅ Resposta Correta: **D**

### 📌 **D. Usar o AWS DMS para migrar o banco de dados + migrar o código da aplicação para o AWS Elastic Beanstalk**

**Por quê?**  
A meta é:
- Migrar uma aplicação **crítica** de um **desktop local** para a AWS,
- Fazer isso com **interrupção mínima**,
- Resultar em uma **arquitetura de várias camadas e altamente disponível**.

A opção **D** atende a todos esses critérios:

#### ✅ Banco de dados:
- **AWS DMS** permite migração **contínua e quase em tempo real** do banco de dados relacional (ex: MySQL, PostgreSQL, SQL Server local) para o **Amazon RDS Multi-AZ**.
- Suporta **CDC (Change Data Capture)**, o que permite manter o banco de origem e destino sincronizados até o momento do cutover.
- Isso reduz drasticamente o tempo de interrupção (frequentemente para **poucos segundos**).

#### ✅ Camada de aplicação:
- **Elastic Beanstalk** é uma plataforma gerenciada que:
  - Simplifica o deployment de aplicações web,
  - Pode implantar **várias instâncias** atrás de um **Application Load Balancer**,
  - Integra-se facilmente com RDS,
  - Fornece **alta disponibilidade** (múltiplas AZs), **escalabilidade automática** e **monitoramento integrado**.

#### ✅ Integração e minimização de downtime:
- Você pode testar a nova aplicação no Elastic Beanstalk com o RDS **antes do cutover**.
- No momento certo, muda o DNS (ex: via Route 53) ou redireciona os usuários — com **impacto mínimo**.

> 💡 **Analogia:** É como construir uma réplica moderna do seu escritório em um prédio novo, enquanto continua trabalhando no antigo — até que tudo esteja pronto, aí você simplesmente muda as chaves da porta em um fim de semana.

## ❌ Por que as outras estão erradas?

### **A. AWS Lambda para migrar a aplicação**
→ Lambda é **serverless sem estado** e adequado para funções curtas — **não para hospedar aplicações web tradicionais** de múltiplas camadas.
→ Criar “funções para criar funções” é complexo e não resolve a hospedagem da aplicação.
→ Não oferece uma arquitetura de 3 camadas (web, app, banco) real.
→ **Inadequado para aplicações legadas de desktop** com interface gráfica ou servidor web embutido.

### **B. VM Import + Auto Scaling**
→ Migrar o **desktop inteiro como uma VM**:
   - Mantém a aplicação monolítica (não é várias camadas),
   - Não garante alta disponibilidade (um desktop não é projetado para ser escalado),
   - Auto Scaling **não funciona bem com aplicações stateful** rodando em uma única VM desktop.
→ Isso é apenas “move to cloud” (lift-and-shift), **não “re-architect”** — e não atende ao requisito de **solução de várias camadas e altamente disponível**.

### **C. Preparar instâncias EC2 antecipadamente sem migração de dados**
→ Preparar instâncias EC2 e RDS é bom, **mas não menciona como os dados serão migrados** com baixo downtime.
→ Sem uma ferramenta como **AWS DMS**, a migração de dados exigiria downtime significativo (ex: parar a aplicação, exportar, importar).
→ Além disso, gerenciar instâncias EC2 manualmente é mais complexo que usar **Elastic Beanstalk**, especialmente para uma equipe empresarial.

## ✅ Conclusão:

Para migrar uma aplicação crítica de um desktop local para uma arquitetura **altamente disponível, de várias camadas e com mínima interrupção**, a melhor abordagem é:

> 🔁 **Migrar o banco de dados com AWS DMS (replicação contínua)**  
> 🌐 **Hospedar a aplicação no Elastic Beanstalk (gerenciado, escalável, com ALB)**  
> 🏗️ **Resultando em uma arquitetura nativa da nuvem, resiliente e de baixo esforço operacional**

> 🎯 **Dica de arquiteto:** Quando o objetivo é **minimizar downtime + modernizar a arquitetura**, combine **AWS DMS + plataforma gerenciada (como Elastic Beanstalk)**. É a receita clássica para migrações empresariais bem-sucedidas.

---

## Questão 63

Um *solutions architect* em uma empresa de comércio eletrônico quer armazenar **dados de log de aplicativos** usando o Amazon S3. O *solutions architect* não tem certeza da frequência com que os logs serão acessados ou quais registros serão mais acessados. A empresa deseja **manter os custos o mais baixo possível**, usando a classe de armazenamento S3 apropriada.

**Qual classe de armazenamento do S3 deve ser implementada para atender a esses requisitos?**

- **A.** Recuperação flexível do S3 Glacier (anteriormente S3 Glacier)  
- **B.** S3 Intelligent-Tiering  
- **C.** S3 Standard-Infrequent Access (S3 Standard-IA)  
- **D.** S3 One Zone-Infrequent Access (S3 One Zone-IA)

## ✅ Resposta Correta: **B**

### 📌 **B. S3 Intelligent-Tiering**

**Por quê?**  
O cenário apresenta duas condições-chave:

| Requisito | Implicação |
|---------|-----------|
| Acessos aos logs são **desconhecidos ou imprevisíveis** | Não se sabe se serão frequentes, raros ou sazonais |
| Objetivo é **minimizar custos** | Precisa de eficiência econômica automática |

A **classe de armazenamento S3 Intelligent-Tiering** foi projetada exatamente para esse tipo de uso:

> 🔍 Ele **monitora automaticamente** o padrão de acesso e **move objetos entre camadas** sem intervenção.

#### Como funciona:
- **Camada padrão (frequente):** Para objetos acessados regularmente.
- **Camada infrequente:** Para objetos raramente acessados (menor custo de armazenamento, taxa de recuperação).
- Pode integrar camadas de arquivamento (como S3 Glacier Instant Retrieval, S3 Glacier Flexible Retrieval) para maior economia.

✅ **Sem tempo de recuperação** (ao contrário do Glacier).  
✅ **Sem taxas adicionais por cálculo de acesso**.  
✅ **Economiza até 90% dos custos** quando os dados ficam inativos.

> 💡 **Analogia:** É como ter um assistente inteligente que guarda seus documentos: os que você usa todo dia ficam na mesa; os antigos vão para o arquivo no chão — tudo sozinho, sem você precisar decidir.

## ❌ Por que as outras estão erradas?

### **A. S3 Glacier (Flexível)**
→ Ideal para arquivamento de longo prazo com **acesso muito raro**.
→ Exige **tempo de recuperação** (de minutos a horas), o que **não é adequado para logs** que podem precisar ser consultados rapidamente.
→ Custo operacional alto se houver acesso frequente.
→ **Não atende ao requisito de "acesso desconhecido" com baixa latência.**

### **C. S3 Standard-IA**
→ Mais barato que S3 Standard, mas **mais caro que necessário se os dados forem acessados raramente**.
→ Tem **taxa de recuperação por acesso**, então se você acessar com frequência, o custo sobe.
→ Você precisa **gerenciar manualmente** quando mover dados para IA — inviável com padrão de acesso desconhecido.

### **D. S3 One Zone-IA**
→ Mais barato que Standard-IA, mas **armazena dados em apenas uma zona de disponibilidade**.
→ **Perda permanente se a AZ falhar** — inaceitável para logs importantes.
→ Não recomendado para dados críticos ou sem backup.
→ Risco desnecessário para pouca economia.

## ✅ Conclusão:

Quando o **padrão de acesso é desconhecido ou variável**, a melhor escolha para minimizar custos sem comprometer desempenho é o **Amazon S3 Intelligent-Tiering**.

É a única classe que:
- ✅ Adapta automaticamente a camadas de custo,
- ✅ Sem tempo de espera para recuperação,
- ✅ Com durabilidade de 11 noves (99.999999999%),
- ✅ Economiza dinheiro mesmo quando o comportamento muda.

> 🏆 **Regra prática:**  
> - Se você **não sabe** como vai acessar os dados → use **Intelligent-Tiering**.  
> - Se você **sabe que será raro** → Standard-IA.  
> - Se for **quase nunca** → Glacier.  
> - Se for **crítico e variável** → **Intelligent-Tiering é o campeão**.

> 🚀 **Dica de arquiteto:** Ative o S3 Intelligent-Tiering com camadas de arquivamento (archive tiers) para obter o menor custo possível à medida que os logs envelhecem.

---

# Questão 64

Um administrador deseja aplicar uma política baseada em recursos ao bucket do S3 chamado **"iam-policy-testbucket"** para restringir o acesso e permitir que as contas gravem apenas objetos no bucket.  
Quando o administrador tenta aplicar a seguinte política ao bucket, o S3 retorna um erro:

```json
{
  "Version": "2012-10-17",
  "Id": "Policy1646946718956",
  "Statement": [
    {
      "Sid": "Stmt1646946717210",
      "Effect": "Allow",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::iam-policy-testbucket/*"
    }
  ]
}
```

Como o administrador pode corrigir a política para resolver o erro e aplicá-la com sucesso?

A  
Alterar o elemento Action de `s3:PutObject` para `s3:*`.

B  
Remover o elemento Resource porque ele é desnecessário para políticas baseadas em recursos.

C  
Alterar o elemento Resource para `NotResource`.

D  
Adicionar um elemento `Principal` à política para declarar quais contas têm acesso.

Resposta correta: D
📝 Explicação

Toda política baseada em recursos (como políticas de bucket S3) deve incluir o elemento Principal, indicando quem está recebendo a permissão — caso contrário, a política é inválida e o S3 retorna erro.
A política original permite a ação, mas não especifica quem pode realizá-la. Portanto, o erro ocorre devido à ausência de Principal.

Exemplo corrigido:
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111122223333:root"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::iam-policy-testbucket/*"
    }
  ]
}
```
---

## Questão 65

Uma empresa está elaborando um aplicativo de armazenamento de documentos na AWS. O aplicativo é executado em instâncias do Amazon EC2 em várias zonas de disponibilidade. A empresa exige que:

- O **armazenamento de documentos esteja altamente disponível**,
- Os documentos precisem estar **disponíveis para todas as instâncias do EC2**,
- Devem ser **retornados imediatamente quando solicitados** (baixa latência),
- São acessados **várias vezes por mês** (acesso moderado, não raro).

O engenheiro líder configurou o uso do **Amazon EBS**, mas está disposto a considerar outras opções.

**O que um solutions architect deve recomendar?**

- **A.** Fazer snapshots dos volumes do EBS regularmente e criar novos volumes usando essas capturas instantâneas em zonas de disponibilidade adicionais.  
- **B.** Usar o Amazon EBS para os volumes raiz da instância do EC2. Configurar o aplicativo para criar o armazenamento de documentos no Amazon S3 Standard.  
- **C.** Usar o Amazon EBS para os volumes raiz da instância do EC2. Configurar o aplicativo para criar o armazenamento de documentos no Amazon S3 Glacier Flexible Retrieval.  
- **D.** Usar pelo menos três volumes do EBS de IOPS provisionados para instâncias do EC2. Montar os volumes nas instâncias do EC2 em uma configuração RAID 5.

## ✅ Resposta Correta: **B**

### 📌 **B. Usar o Amazon EBS para os volumes raiz da instância do EC2. Configurar o aplicativo para criar o armazenamento de documentos no Amazon S3 Standard.**

**Por quê?**  
Vamos analisar os requisitos críticos:

| Requisito | Implicação |
|---------|-----------|
| Altamente disponível | Dados devem sobreviver a falhas de instância ou AZ |
| Disponível para todas as instâncias | Armazenamento compartilhado entre múltiplas EC2s |
| Retorno imediato | Sem tempo de restauração (ex: minutos/horas) |
| Acesso várias vezes por mês | Não é raro — padrão S3 Standard é adequado |

#### Problema com EBS:
- O **EBS é anexado a uma única instância em uma única zona de disponibilidade**.
- Mesmo com snapshots ou RAID, **não é nativamente compartilhado nem multi-AZ**.
- Se a instância falhar ou a AZ cair, o volume pode ficar inacessível até recuperação manual.

> ❌ Portanto, **EBS sozinho não atende ao requisito de alta disponibilidade e acesso compartilhado**.

### ✅ Solução correta: **Amazon S3 Standard**

O **S3 Standard** oferece:
- ✅ **Alta durabilidade (99,999999999%) e disponibilidade (99,99%)**
- ✅ **Acesso imediato** aos objetos (sem espera)
- ✅ **Acessível por todas as instâncias EC2**, independentemente da AZ
- ✅ **Escalável, gerenciado e econômico** para dados usados com frequência moderada

Ao migrar o armazenamento de documentos do EBS para o **S3 Standard**, o aplicativo passa a:
- Armazenar documentos centralmente,
- Garantir que todos os servidores tenham acesso idêntico,
- Eliminar pontos únicos de falha.

> 💡 **Analogia:** É como trocar um HD externo conectado a um único computador por um drive na nuvem — todo mundo acessa, ninguém perde se um PC quebrar.

## ❌ Por que as outras estão erradas?

### **A. Snapshots regulares + novos volumes em outras AZs**
→ Isso é **backup**, não alta disponibilidade.
→ Para acessar dados em outra AZ, você precisa **criar um novo volume e anexar manualmente** → alto RTO.
→ **Não fornece acesso imediato nem automático**.
→ Viola o requisito de "imediatamente quando solicitado".

### **C. S3 Glacier Flexible Retrieval**
→ Ideal para arquivamento com acesso raro (ex: <1x/ano).
→ Exige **tempo de recuperação de minutos a horas**.
→ **Não fornece acesso imediato** — viola requisito claro.
→ Inadequado para documentos acessados várias vezes por mês.

### **D. Volumes EBS em RAID 5**
→ **RAID 5 não é viável com EBS em múltiplas instâncias**:
   - Cada volume EBS só pode ser anexado a **uma instância por vez**.
   - Impossível montar o mesmo volume em múltiplos EC2s simultaneamente.
   - RAID 5 requer sincronização direta entre discos — inviável entre AZs.
→ Além disso, EBS não foi projetado para esse tipo de configuração distribuída.
→ **Tecnicamente inviável e frágil**.

## ✅ Conclusão:

Para um aplicativo que exige **armazenamento de documentos altamente disponível, compartilhado e com acesso imediato**, a melhor escolha é **migrar do EBS para o Amazon S3 Standard**.

> ✅ Use **EBS** para volumes raiz (SO, aplicação).  
> ✅ Use **S3 Standard** para dados compartilhados (documentos, mídia, logs).  

É uma **melhor prática arquitetural fundamental** na AWS.

> 🚀 **Dica de arquiteto:** Quando você pensar em “compartilhar armazenamento entre EC2s”, pense primeiro em **S3**, **EFS** ou **FSx** — nunca em EBS diretamente.

---
