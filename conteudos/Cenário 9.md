## 🎬 Cenário 9 - Criptografia de Dados em Repouso no Amazon EBS

A empresa está implantando uma nova aplicação em instâncias da Amazon EC2.  
A aplicação escreve dados nos volumes do Amazon Elastic Block Store (Amazon EBS).  
A empresa precisa garantir que todos os dados escritos nos volumes do EBS sejam criptografados em repouso.  

### 🤔 Qual solução atenderá a esse requisito?

### ➡️ Resposta Correta
Criar os volumes do EBS como volumes criptografados. Anexar os volumes do EBS às instâncias da EC2.

## 🧠 Explicação

A criptografia de dados em repouso no **Amazon EBS** é configurada **no momento da criação do volume**.

Quando um volume EBS é criado como criptografado:

- Todos os dados gravados no volume são **automaticamente criptografados**
- A criptografia ocorre **em repouso, em trânsito dentro da AWS e nos snapshots**
- É utilizada:
  - A **chave padrão gerenciada pela AWS (AWS managed key)**  
  **ou**
  - Uma **chave personalizada do AWS KMS (Customer Managed Key)**

📌 **Importante:**  
A criptografia é **transparente para a aplicação**. Nenhuma alteração no código é necessária.

## 🛡️ O que essa solução garante?
- Criptografia automática de dados em repouso
- Conformidade com requisitos de segurança
- Simplicidade operacional
- Integração nativa com o **AWS Key Management Service (KMS)**

## ⚠️ Observações Importantes
- Um volume **não criptografado não pode ser criptografado diretamente**
- Para criptografar um volume existente:
  1. Criar um snapshot do volume
  2. Copiar o snapshot habilitando criptografia
  3. Criar um novo volume criptografado a partir do snapshot

## 📝 Dica para Provas AWS
Se o cenário mencionar:
- EC2
- EBS
- Criptografia em repouso

👉🏻 A resposta correta quase sempre envolve **criar volumes EBS criptografados**.

## 📌 Resumo Final
A forma correta de garantir criptografia de dados em repouso no Amazon EBS é **criar os volumes como criptografados desde o início**.  
Essa abordagem é simples, segura e totalmente gerenciada pela AWS.

---

👉🏻 [Clique aqui para voltar ao Readme](https://github.com/DrikaDev/Estudando-AWS-Solutions-Architect-Associate/blob/main/README.md) 📒
