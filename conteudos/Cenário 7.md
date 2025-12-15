## 🎬 Cenário 7 - Mitigação de Ponto Único de Falha em Arquitetura Multi-VPC

Uma empresa possui duas VPCs denominadas Management e Production.  
A VPC Management usa VPNs por meio de um gateway do cliente para se conectar a um único dispositivo no data center.  
A VPC Production usa um gateway privado virtual com duas conexões AWS Direct Connect anexadas.  
As VPCs Management e Production usam uma única conexão de VPC peering para permitir a comunicação entre as aplicações.  

### 🤔 O que um arquiteto de soluções deve fazer para mitigar qualquer **ponto único de falha** nesta arquitetura?

### ➡️ Resposta:
Adicionar um segundo conjunto de VPNs à VPC Management a partir de um segundo dispositivo de gateway do cliente.

## 🧠 Explicação Geral

O principal ponto único de falha - **SPOF - Single Point of Failure** - nessa arquitetura está na **conexão entre a VPC Management e o data center**, que depende de:
- Um **único dispositivo de gateway do cliente**
- Um **único conjunto de VPNs**

Se esse dispositivo falhar, a conectividade com a VPC Management será totalmente perdida.

## 🔁 Por que adicionar um segundo gateway do cliente?

### ✔️ Benefícios da solução

- Fornece **redundância** para a conexão VPN
- Elimina o SPOF causado por um único dispositivo on-premises
- Permite **failover automático** entre túneis VPN
- Aumenta a **resiliência e disponibilidade** da arquitetura

👉🏻 A AWS recomenda **sempre múltiplos gateways do cliente** para arquiteturas críticas.

## 🧩 Análise das outras partes da arquitetura

### 🟢 VPC Production
- Utiliza **duas conexões AWS Direct Connect**
- Já possui **alta disponibilidade e redundância**

### 🟢 VPC Peering
- O **VPC peering é inerentemente redundante**
- Não é necessário criar múltiplos peerings para alta disponibilidade

👉🏻 Portanto, **nenhuma alteração é necessária** nessas partes da arquitetura.

## 📌 Resumo Final

| Componente | Situação | Ação Necessária |
|----------|--------|----------------|
VPC Management | VPN com único gateway do cliente | ➕ Adicionar segundo gateway |
VPC Production | Duas conexões Direct Connect | ✅ Já redundante |
VPC Peering | Conexão única | ✅ Já redundante |

## 🧠 Dica de Prova

Se o cenário mencionar:
- VPN
- Um único gateway do cliente
- SPOF
- Alta disponibilidade

👉🏻 A resposta quase sempre envolve **adicionar redundância no gateway do cliente**.

---

👉🏻 [Clique aqui para voltar ao Readme](https://github.com/DrikaDev/Estudando-AWS-Solutions-Architect-Associate/blob/main/README.md) 📒
