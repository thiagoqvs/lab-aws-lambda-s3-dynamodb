# ☁️ Laboratório AWS — Lambda, S3 e DynamoDB

> Repositório de documentação técnica desenvolvido como entrega do desafio prático da [DIO](https://www.dio.me/), consolidando conhecimentos sobre serverless e gerenciamento de serviços na AWS.

---

## 📋 Sobre o Projeto

Este laboratório tem como objetivo praticar a integração entre os principais serviços da AWS em um fluxo orientado a eventos:

- Upload de arquivo via **AWS CLI** para o **Amazon S3**
- Disparo automático de uma **função Lambda** via trigger de evento S3
- Persistência dos dados processados no **Amazon DynamoDB**

---

## 🗺️ Arquitetura do Fluxo

```
Sistema de Arquivos Local
        │
        ▼
   [Arquivo local]
        │
        ▼
AWS CLI — aws s3 cp (Transfer file)
        │
        ▼
  Amazon S3 (Bucket)
        │
     Trigger (evento PutObject)
        ▼
  AWS Lambda
  (Node.js / .NET Core / Python)
        │
        ▼
  Amazon DynamoDB (Tabela)
```

> 📌 Diagrama visual criado com [draw.io](https://draw.io) — disponível na pasta `/images`.

---

## 🛠️ Serviços AWS Utilizados

| Serviço | Função no Fluxo |
|---|---|
| **Amazon S3** | Armazenamento do arquivo enviado via CLI; origem do evento |
| **AWS Lambda** | Função serverless acionada pelo evento de upload no S3 |
| **Amazon DynamoDB** | Banco NoSQL que persiste os dados processados pela Lambda |
| **AWS CLI** | Ferramenta de linha de comando usada para transferir o arquivo ao S3 |

---

## ⚙️ Passo a Passo Prático

### 1. Pré-requisitos

- Conta AWS ativa
- AWS CLI instalada e configurada (`aws configure`)
- Python 3.x ou Node.js instalado (para escrever a função Lambda)

---

### 2. Criar o Bucket S3

```bash
aws s3 mb s3://meu-bucket-lab-dio --region us-east-1
```

Habilitar notificações de evento no bucket (via Console AWS ou CLI) apontando para a função Lambda.

---

### 3. Criar a Tabela no DynamoDB

```bash
aws dynamodb create-table \
  --table-name ArquivosProcessados \
  --attribute-definitions AttributeName=arquivo_id,AttributeType=S \
  --key-schema AttributeName=arquivo_id,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region us-east-1
```

---

### 4. Criar a Função Lambda (Python)

```python
import boto3
import json
import os
from datetime import datetime

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('ArquivosProcessados')

def lambda_handler(event, context):
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key    = record['s3']['object']['key']
        size   = record['s3']['object']['size']

        table.put_item(Item={
            'arquivo_id': key,
            'bucket':     bucket,
            'tamanho':    size,
            'processado_em': datetime.utcnow().isoformat()
        })
        print(f"Registrado: {key} do bucket {bucket}")

    return {'statusCode': 200, 'body': json.dumps('OK')}
```

**Permissões necessárias para a role da Lambda:**
- `AmazonS3ReadOnlyAccess`
- `AmazonDynamoDBFullAccess`
- `AWSLambdaBasicExecutionRole`

---

### 5. Configurar o Trigger S3 → Lambda

No Console AWS:
1. Acesse a função Lambda criada
2. Clique em **"Adicionar trigger"**
3. Selecione **S3**
4. Escolha o bucket criado
5. Tipo de evento: **PUT** (ou `s3:ObjectCreated:*`)
6. Salve

---

### 6. Enviar o Arquivo via AWS CLI

```bash
# Criar arquivo de teste
echo '{"nome":"Thiago","evento":"DIO Lab"}' > dados.json

# Upload para o S3
aws s3 cp dados.json s3://meu-bucket-lab-dio/dados.json
```

Após o upload, o trigger dispara automaticamente a Lambda.

---

### 7. Verificar os Dados no DynamoDB

```bash
aws dynamodb scan --table-name ArquivosProcessados
```

Resultado esperado:

```json
{
  "Items": [
    {
      "arquivo_id": {"S": "dados.json"},
      "bucket":     {"S": "meu-bucket-lab-dio"},
      "tamanho":    {"N": "38"},
      "processado_em": {"S": "2026-06-04T12:00:00"}
    }
  ]
}
```

---

## 💡 Principais Aprendizados

- **Serverless na prática**: a Lambda elimina a necessidade de gerenciar servidores — o código só executa quando há evento.
- **Event-driven architecture**: o trigger S3 → Lambda é um padrão muito usado em pipelines de dados e automações na nuvem.
- **DynamoDB como destino**: banco NoSQL ideal para registros de eventos por ter baixa latência e escala automática.
- **AWS CLI**: ferramenta essencial para automatizar tarefas que seriam manuais no Console.
- **IAM Roles**: toda integração entre serviços AWS exige permissões explícitas — princípio do menor privilégio.

---

## 🔗 Referências

- [Documentação AWS Lambda](https://docs.aws.amazon.com/pt_br/lambda/latest/dg/welcome.html)
- [Documentação Amazon S3](https://docs.aws.amazon.com/pt_br/AmazonS3/latest/userguide/Welcome.html)
- [Documentação Amazon DynamoDB](https://docs.aws.amazon.com/pt_br/amazondynamodb/latest/developerguide/Introduction.html)
- [AWS CLI — Referência S3](https://docs.aws.amazon.com/cli/latest/reference/s3/)
- [GitHub Quick Start — DIO](https://github.com/digitalinnovationone/github-quickstart)

---

## 👤 Autor

**Thiago** — Estudante de Análise e Desenvolvimento de Sistemas | Transição para Data Engineering  
[![GitHub](https://img.shields.io/badge/GitHub-thiagoqvs-181717?logo=github)](https://github.com/thiagoqvs)
