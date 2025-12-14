## 🎬 Cenário 1 - Redis x Memcached

Uma empresa desenvolveu um novo jogo de vídeo como uma aplicação Web.  
A aplicação está numa arquitetura de três camadas numa VPC com Amazon RDS para MySQL na camada de base de dados.  
Vários jogadores irão competir online em simultâneo.  
Os criadores do jogo pretendem apresentar um painel de avaliação dos 10 melhores em tempo real e oferecer a capacidade de parar e restaurar o jogo, preservando as pontuações atuais.

🤔 O que um arquiteto de soluções deve fazer para atender a esses requisitos?

➡️ Configurar um *cluster Amazon ElastiCache for *Redis para calcular e armazenar em cache as pontuações a serem exibidas pelo aplicativo da Web.

## Redis ✖️ Memcached: você sabe a diferença? 👀

Se você está estudando para a SAA, assim como eu, é fundamental entender a diferença entre Redis e Memcached.

### 🔴 Redis (Remote Dictionary Server)  
É um banco de dados in-memory, usado para armazenar e acessar dados com altíssima performance.  

⚡ Extremamente rápido (dados ficam na memória RAM)  
🧠 Estruturas de dados avançadas (listas, strings, *sets, hashes, sorted sets — este último perfeito para rankings)  
💾 Pode ter persistência  
🔄 Suporta replicação e alta disponibilidade  

👉 Redis é rápido e inteligente: ideal para ranking em tempo real, sessões de usuários, contadores e estado de aplicações (como jogos online 🎮).  

> - **Set** é uma coleção de valores únicos / uma lista sem repetição, ou seja, não permite elementos duplicados.  
> Ex: ` jogadores_online = {Ana, João, Maria} `  
> Se tentar adicionar ` Ana ` novamente, o Redis ignora, pois o valor já existe.  

> - **Hash** armazena dados no formato campo → valor, semelhante a um objeto ou registro em um banco relacional. Ex:  
> ``` 
> jogador:123
> nome = Ana
> nivel = 7
> pontos = 980
> ```
> Hash pode ser entendido como um objeto em memória.

> **Sorted Set** é semelhante a um Set, porém cada elemento possui um score (pontuação) associado. Os dados são mantidos automaticamente ordenados pelo score. Ex:
> ```
> João → 1200
> Ana → 980
> Maria → 870
> ```
> Sorted Set pode ser entendido como um ranking automático.  

### 🔵 Memcached  
É um sistema de cache distribuído em memória, mais simples e direto ao ponto.  

⚡ Muito rápido  
🧱 Armazena apenas pares chave–valor  
❌ Não tem persistência (se reiniciar, perde tudo)  
❌ Não tem replicação nem failover  

👉 Memcached é rápido, mas simples e descartável: ideal para cache temporário, como consultas ao banco de dados ou páginas web.  

🎯 Exemplos práticos:  
- 🎮 Jogos online  
Ranking em tempo real → ✅ Redis  
Pontuação não pode ser perdida → ✅ Redis  

- 🌐 Site institucional  
Cache de páginas → ✅ Memcached

## 🧠 O que é um cluster?

Um cluster é um conjunto de servidores (nós) que trabalham juntos como se fossem um único sistema.  

👉 Em vez de depender de uma única máquina, o cluster distribui:  
- carga de trabalho
- dados
- processamento

### Por que usar um cluster?
⚡ Mais performance  
🔄 Alta disponibilidade  
📈 Escalabilidade  
🛡️ Tolerância a falhas  

👉 Resumo simples:  
Cluster = várias máquinas trabalhando juntas para ser mais rápido, estável e escalável.  

## 🔴 O que é um cluster Amazon ElastiCache for Redis?

<img width="784" height="277" alt="image" src="https://github.com/user-attachments/assets/d6f4368b-ec03-4d67-aacc-daacce13ca6d" />

No Amazon ElastiCache for Redis, um cluster é composto por:  
🔹Nós (nodes)
Primary node: recebe escritas  
Replica nodes: replicam dados e assumem em caso de falha  
🔹Pode ter:
Replicação
Failover automático
Sharding (em clusters maiores)

### 📌 Agora, interpretando a frase do simulado  
“Configure um cluster Amazon ElastiCache for Redis para calcular e armazenar em cache as pontuações a serem exibidas pelo aplicativo da Web.”

### O que isso significa na prática?  
✅ Configure um cluster Redis  
Não é uma instância única  
É um ambiente com alta disponibilidade  

✅ Calcular as pontuações  
A lógica de pontuação acontece na aplicação  
O Redis armazena e organiza esses dados rapidamente  

✅ Armazenar em cache as pontuações  
As pontuações ficam em memória  
Evita consultas constantes ao RDS  
Garante baixa latência  

✅ Exibir no aplicativo Web  
O front-end consulta o Redis  
O ranking aparece em tempo real  

🎮 Ligando com o exemplo do jogo  
- No jogo online:  
Cada jogador tem uma pontuação  
O Redis usa Sorted Sets para:  
armazenar jogador + score  
retornar rapidamente o Top 10  

- O cluster garante que:  
o ranking não caia  
os dados não sejam perdidos  
o sistema suporte muitos jogadores  

### 🎯 Frase-chave para prova AWS
Um cluster ElastiCache for Redis fornece cache em memória altamente disponível e escalável para armazenar dados de baixa latência, como rankings e estado de aplicações.  

---

👉🏻 [Clique aqui para voltar ao Readme](https://github.com/DrikaDev/Estudando-AWS-Solutions-Architect-Associate/blob/main/README.md) 📒
