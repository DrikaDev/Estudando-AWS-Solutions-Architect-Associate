## 🧪 Simulado AWS – Amazon EBS

Uma empresa utiliza alta capacidade de armazenamento em bloco para executar suas cargas de trabalho localmente.  
O pico diário de transações de entrada e saída por segundo da empresa não ultrapassa 15.000 IOPS.  
A empresa deseja migrar as cargas de trabalho para o Amazon EC2 e provisionar o desempenho do disco independentemente da capacidade de armazenamento.  

### 🤔 Qual tipo de volume do Amazon Elastic Block Store (Amazon EBS) atenderá a esses requisitos de forma mais econômica?  

### Opções
- Tipo de volume **gp3**
- Tipo de volume **gp2**
- Tipo de volume **io1**
- Tipo de volume **io2**

### ➡️ Resposta
**Tipo de volume gp3**

## 🧠 Explicação
O **gp3** é a opção mais adequada porque:

- ✔️ Permite **provisionar IOPS e throughput independentemente da capacidade** de armazenamento  
- ✔️ Atende facilmente ao requisito de **até 15.000 IOPS**  
- ✔️ É **mais econômico** do que os volumes **io1** e **io2** para esse nível de desempenho  
- ✔️ Oferece um excelente **equilíbrio entre custo e performance** para cargas de trabalho que não exigem IOPS extremamente elevados

## Pra que servem os volumes do EBS?

Esses nomes - gp3, gp2, io1, io2 - são tipos de volumes do Amazon EBS (Elastic Block Store).  
Eles definem desempenho, custo e caso de uso do disco que você anexa a uma instância EC2.  
No total, o EBS possui 6 tipos principais, divididos em:  
- SSD otimizados para IOPS e baixa latência (gp3, gp2, io1 e io2)
- HDD otimizados para throughput, não IOPS (st1 e sc1)

### SSD otimizados para IOPS e baixa latência

#### ➡️ gp3 – General Purpose SSD (mais moderno)
👉🏼 Uso geral (recomendado hoje)

Características:
- Uso geral
- Melhor custo-benefício
- Você pode separar tamanho, IOPS e throughput
- Performance previsível
- Suporta até 16.000 IOPS
- Você pode provisionar exatamente 15.000 IOPS
- Ideal para a maioria dos workloads

Quando usar:
- Aplicações web
- Servidores
- Ambientes de teste e produção
- A maioria dos workloads

> 📌 É o substituto natural do gp2

#### ➡️ gp2 – General Purpose SSD (legado)
👉🏼 Antigo padrão

Características:
- IOPS vinculadas ao tamanho do volume
- Menos flexível
- Performance depende do tamanho do volume
- Até 16.000 IOPS
- Vem sendo substituído pelo gp3

Quando usar:
- Ambientes antigos
- Workloads simples

> ⚠️ Hoje, gp3 é quase sempre melhor

#### ➡️ io1 – Provisioned IOPS SSD
👉🏼 Alta performance com IOPS garantidas

Características:
- SSD
- IOPS provisionadas manualmente
- Latência muito baixa
- Custo alto
- Suporta até 64.000 IOPS
- IOPS totalmente garantidas
- Usado para bancos de dados exigentes

Quando usar:
- Bancos de dados críticos
- Sistemas transacionais (ex: MySQL, Oracle)
- Aplicações que exigem I/O constante

#### ➡️ io2 – Provisioned IOPS SSD (mais avançado)
👉🏼 Versão mais robusta do io1

Características:
- Ideal para cargas mission-critical
- Suporta até 256.000 IOPS
- Maior durabilidade e consistência
- Ideal para workloads mission-critical
- Mais caro

Quando usar:
- Bancos financeiros
- Sistemas críticos
- Ambientes empresariais exigentes

### HDD otimizados para throughput, não IOPS

#### ➡️ st1 – Throughput Optimized HDD
- Ideal para big data, ETL, streaming
- Alto throughput, baixa IOPS
- Mais barato que SSD

#### ➡️ sc1 – Cold HDD
- Para dados acessados raramente
- Muito barato
- Menor performance

## Mas o que é mesmo IOPS?

IOPS significa Input/Output Operations Per Second, ou seja, Operações de Entrada e Saída por Segundo.  
IOPS mede quantas operações de leitura e escrita um disco consegue realizar a cada segundo.  
👉🏼 Quanto maior o IOPS, mais rápido o armazenamento responde a muitas requisições pequenas.  

### 🔍 Exemplos práticos

- Um banco de dados fazendo muitas consultas pequenas → precisa de IOPS alto
- Um sistema de arquivos com muitos acessos simultâneos → precisa de IOPS alto
- Aplicações transacionais (ERP, CRM, e-commerce) → precisam de IOPS alto

## E o que é mesmo Throughput?

Throughput é a quantidade de dados que podem ser transferidos por segundo, geralmente medida em MB/s (megabytes por segundo).  
Enquanto o IOPS mede quantas operações o disco consegue realizar, o throughput mede o volume total de dados que passa pelo disco em um determinado período de tempo.  

## 🆚 IOPS x Throughput

| Métrica      | Mede                          | Exemplo                          |
|--------------|-------------------------------|----------------------------------|
| IOPS         | Quantidade de operações       | Muitas leituras pequenas         |
| Throughput   | Volume de dados (MB/s)        | Arquivos grandes, streaming      |

## 📌 Analogia rápida

- IOPS = quantos carros passam por minuto
- Throughput = quantos quilos de carga esses carros levam

---

👉🏻 [Clique aqui para voltar ao Readme](https://github.com/DrikaDev/Estudando-AWS-Solutions-Architect-Associate/blob/main/README.md) 📒
