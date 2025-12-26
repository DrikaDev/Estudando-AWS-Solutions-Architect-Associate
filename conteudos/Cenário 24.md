## 🎬 Cenário 24 – Amazon VPC e Endereçamento IP

Um arquiteto de soluções configurou uma VPC que possui uma pequena faixa de endereços IP.  
O número de instâncias Amazon EC2 na VPC está aumentando e há um número insuficiente de endereços IP para as cargas de trabalho futuras.

### 🤔 Qual solução resolve esse problema com o **MENOR overhead operacional**?

### ➡️ Resposta
Adicionar um bloco CIDR IPv4 adicional à VPC para aumentar o número de endereços IP e criar novas sub-redes utilizando esse novo CIDR.  
Em seguida, criar novos recursos nas novas sub-redes.  

## 🧠 Justificativa
Essa é a opção mais direta e com menor overhead operacional porque:

- ✔️ Não requer a criação de uma nova VPC, simplificando a gestão do ambiente.
- ✔️ Não necessita configurar conexões entre VPCs (como VPC Peering, Transit Gateway ou VPN), reduzindo a complexidade.
- ✔️ Permite expandir o range de IPs dentro da VPC existente, mantendo todos os recursos na mesma estrutura lógica.
- ✔️ É uma operação relativamente simples de executar pelo console da AWS ou via AWS CLI.
- ✔️ Não exige mudanças significativas nas configurações de rede ou nas políticas de roteamento existentes.
- ✔️ Mantém a consistência operacional, pois os novos recursos são gerenciados da mesma forma que os atuais.

## ❌ Por que não as outras opções?
As demais alternativas envolvem a criação de novas VPCs e a configuração de conectividade entre elas, o que aumenta significativamente:

- O overhead operacional
- A complexidade da arquitetura
- O esforço de manutenção e governança

---

👉🏻 [Clique aqui para voltar ao Readme](https://github.com/DrikaDev/Estudando-AWS-Solutions-Architect-Associate/blob/main/README.md) 📒
