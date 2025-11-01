# Desafio de Projeto DIO: Executando Tarefas Automatizadas com Lambda Function e S3

Este repositório documenta a solução do Desafio de Projeto da [Digital Innovation One (DIO)](https://www.dio.me/) sobre a automação de tarefas usando AWS Lambda e Amazon S3.

O objetivo foi aplicar os conceitos de **arquitetura orientada a eventos** e **infraestrutura como código (IaC)**. Utilizamos um template **AWS CloudFormation** para provisionar e "conectar" todos os recursos necessários para que um upload de arquivo em um bucket S3 dispare automaticamente uma função Lambda para processamento.

---

## 🎯 Objetivo

O desafio consistiu em implementar um pipeline de automação serverless. O foco é documentar o processo, a arquitetura e os principais conceitos de permissão e triggers que fazem essa integração funcionar.

---

## 📖 Conceitos-Chave da Arquitetura

Esta arquitetura é um dos padrões mais comuns e poderosos da nuvem AWS.

* **Amazon S3 (Simple Storage Service):** É o nosso serviço de armazenamento de objetos. Neste projeto, ele atua como o **"gatilho" (trigger)**. Nós o usamos para duas finalidades:
    1.  `Bucket de Input`: Recebe os arquivos originais. O upload de um novo arquivo *inicia* o workflow.
    2.  `Bucket de Output`: Armazena o resultado do processamento da Lambda.
* **AWS Lambda:** É o nosso serviço de computação "serverless" (sem servidor). Ele fornece o código que executa em resposta ao evento do S3. Não precisamos gerenciar servidores; a função apenas "acorda", executa e "dorme".
* **AWS CloudFormation (IaC):** Em vez de criar os buckets, a função e as permissões manualmente pelo console (o que é sujeito a erros), nós definimos todos eles em um único arquivo **YAML**. O CloudFormation lê esse "mapa" e constrói toda a infraestrutura e suas conexões de forma automatizada e 100% repetível.
* **IAM (Permissões):** A parte mais crítica da automação. Para que isso funcione, duas permissões são necessárias:
    1.  A *Lambda* precisa de permissão para ler do bucket de input e escrever no bucket de output (via **IAM Role**).
    2.  O *S3* precisa de permissão para invocar a função Lambda (via **Lambda Permission**).

---

## ⚙️ O Projeto: Pipeline de Processamento de Arquivos

O workflow implementado por este template é o seguinte:

1.  Um usuário (ou sistema) faz o upload de um arquivo (ex: `teste.txt`) no `BucketDeInput`.
2.  O S3 detecta o evento `s3:ObjectCreated:*` (criação de novo objeto).
3.  O S3, que tem permissão, invoca a `MinhaFuncaoLambda`, enviando os detalhes do evento (qual arquivo, em qual bucket).
4.  A `MinhaFuncaoLambda` (escrita em Python) é executada. Ela:
    * Lê o `teste.txt` do `BucketDeInput`.
    * Processa o conteúdo (neste exemplo, converte todo o texto para MAIÚSCULAS).
    * Salva um novo arquivo (ex: `resultado-teste.txt`) no `BucketDeOutput`.
5.  O fluxo termina, e a Lambda é finalizada.

### Diagrama Conceitual do Fluxo

```mermaid
graph TD;
    style Input fill:#D82233,color:#fff
    style Output fill:#D82233,color:#fff
    style Lambda fill:#FF9900,color:#fff
    
    Usuario[Usuário] -->|1. Upload 'teste.txt'| Input(S3 - Bucket de Input);
    Input -->|2. Evento s3:ObjectCreated| Lambda(AWS Lambda - ProcessarTexto);
    Lambda -->|3. GetObject 'teste.txt'| Input;
    Lambda -->|4. PutObject 'RESULTADO.txt'| Output(S3 - Bucket de Output);
