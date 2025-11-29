## AWS Well-Architected Framework

O **AWS Well-Architected Framework** é um guia que fornece uma abordagem consistente para avaliar arquiteturas de nuvem e orientações para implementar 
projetos bem estruturados.  
Ele apresenta um conjunto de perguntas fundamentais e práticas recomendadas que permitem analisar se uma arquitetura está alinhada aos padrões da AWS.

A AWS desenvolveu este framework após estudar milhares de workloads de clientes, consolidando suas melhores práticas.

O framework é organizado em **seis pilares**:

1. **Excelência operacional (Operational Excellence)**
2. **Segurança (Security)**
3. **Confiabilidade (Reliability)**
4. **Eficiência de desempenho (Performance Efficiency)**
5. **Otimização de custos (Cost Optimization)**
6. **Sustentabilidade (Sustainability)**

---

## 📘 1. Pilar de Excelência Operacional

**Objetivos principais:**
- Executar e monitorar sistemas para fornecer valor comercial.  
- Melhorar continuamente processos e procedimentos de suporte.  
- Tratar todo o workload como código.  

A Excelência Operacional aborda a capacidade de operar sistemas com eficiência, obter insights operacionais por meio de logs, métricas e instrumentação, além de aprimorar continuamente as operações.

Na AWS, é possível definir toda a infraestrutura — aplicações, políticas, operações e governança — como código. Isso possibilita aplicar os mesmos princípios de engenharia usados para aplicações em cada elemento da pilha.

Investir em operações como código reduz erros, aumenta a produtividade e permite respostas automatizadas.

---

## 🔐 2. Pilar de Segurança

**Princípios fundamentais:**
- Implementar um princípio de identidade sólido.  
- Manter rastreabilidade.  
- Aplicar segurança em todas as camadas.  
- Avaliar riscos e implementar estratégias de mitigação.  

O foco do pilar Segurança é proteger informações, sistemas e ativos, garantindo valor para o negócio. Isso inclui automatizar práticas recomendadas, proteger dados em trânsito e repouso, registrar ações e manter controles rigorosos de acesso.

---

## 🔄 3. Pilar de Confiabilidade

**Princípios fundamentais:**
- Recuperar rapidamente.  
- Atender dinamicamente à demanda de computação.  
- Reduzir interrupções.  

A Confiabilidade aborda a capacidade de um sistema de se recuperar de falhas de infraestrutura e de adquirir recursos para atender à demanda. Também foca em mitigar interrupções causadas por configurações incorretas ou problemas de rede.

Arquiteturas confiáveis na AWS são projetadas com alta disponibilidade, tolerância a falhas e redundância.

---

## 🚀 4. Pilar de Eficiência de Desempenho

**Princípios fundamentais:**
- Escolher e manter recursos eficientes.  
- Democratizar tecnologias avançadas.  
- Usar afinidade mecânica (escolher a tecnologia mais adequada).  

A Eficiência de Desempenho trata de usar recursos computacionais da forma mais eficiente possível, ajustando a arquitetura conforme a demanda.

Também implica adotar tecnologias fornecidas como serviço — quando o fornecedor gerencia a complexidade — e escolher ferramentas que melhor se ajustem ao padrão de acesso e processamento de dados.

---

## 💰 5. Pilar de Otimização de Custos

**Princípios fundamentais:**
- Medir a eficiência.  
- Eliminar despesas desnecessárias.  
- Adotar o modelo de consumo correto.  
- Usar serviços gerenciados sempre que possível.  

A otimização de custos é contínua. Requer avaliar regularmente o uso dos recursos, remover desperdícios, escolher o modelo de pagamento adequado e adotar serviços gerenciados para aproveitar economias de escala da AWS.

---

## 🌱 6. Pilar de Sustentabilidade

**Princípios fundamentais:**
- Estabelecer metas de sustentabilidade.  
- Maximizar a utilização.  
- Escolher hardware e software eficientes.  
- Reduzir impactos posteriores.  

A Sustentabilidade foca em reduzir o impacto ambiental, econômico e social das cargas de trabalho. Isso envolve melhorar a eficiência energética, otimizar a utilização de recursos, escolher tecnologias eficientes e projetar workloads que reduzam desperdícios.

Inclui desde a escolha da linguagem de programação até o uso de algoritmos modernos e armazenamento eficiente.

---

# AWS Well-Architected Tool (WA Tool)

A **AWS Well-Architected Tool** ajuda você a analisar workloads e compará-los com as práticas recomendadas mais recentes da AWS.

**A ferramenta oferece:**
- Avaliação do estado atual da arquitetura.  
- Acesso ao conhecimento e melhores práticas da AWS sob demanda.  
- Plano de ação detalhado com passos para aprimorar workloads.  
- Processo consistente para medir e revisar arquiteturas.  

Dentro da ferramenta, você define o workload e responde perguntas relacionadas aos pilares:

- Excelência operacional  
- Segurança  
- Confiabilidade  
- Eficiência de desempenho  
- Otimização de custos  

Com base nas respostas, a WA Tool gera recomendações e próximos passos, ajudando a reduzir falhas, melhorar processos e embasar decisões de arquitetura e governança.

---

Para mais detalhes, consulte o **Guia do Usuário da AWS Well-Architected Tool** e o conteúdo oficial do curso AWS Training and Certification.

