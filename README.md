Documentação: Implementação de State Machines com AWS Step Functions
Este documento descreve as melhores práticas para implementar State Machines usando AWS Step Functions em um ambiente controlado, com CI/CD, versionamento, deploy com Canary e Blue/Green, e integração com GitHub Actions e AWS CLI.

1. Deployment: Canary e Blue/Green
1.1. O que é Canary Deployment?
Canary Deployment é uma estratégia de deploy onde uma nova versão é liberada para uma pequena parte do tráfego, enquanto a maioria continua usando a versão atual.

Se a nova versão funcionar corretamente, ela é gradualmente liberada para todo o tráfego.

1.2. O que é Blue/Green Deployment?
Blue/Green Deployment é uma estratégia onde duas versões do ambiente (Blue e Green) são mantidas simultaneamente.

O tráfego é alternado entre as versões, permitindo rollback rápido em caso de problemas.

1.3. Implementação com AWS Step Functions
Canary: Use AWS Lambda Aliases e Step Functions Versions para liberar gradualmente a nova versão da State Machine.

Blue/Green: Utilize Step Functions Versions e CloudFormation Stacks para manter duas versões da State Machine e alternar entre elas.

Exemplo de Canary Deployment
Crie uma nova versão da State Machine.

Configure um alias (por exemplo, Prod) para apontar para a versão atual e a nova versão, com uma divisão de tráfego (90% para a versão atual, 10% para a nova).

Monitore a nova versão e, se estiver estável, aumente gradualmente o tráfego.

Exemplo de Blue/Green Deployment
Crie uma nova Stack no CloudFormation com a nova versão da State Machine.

Teste a nova Stack em um ambiente isolado.

Altere o DNS ou o endpoint para apontar para a nova Stack.

Se houver problemas, volte para a Stack anterior.

2. Versionamento
2.1. O que é Versionamento?
Versionamento é o controle de mudanças no código e na infraestrutura, permitindo rastrear alterações, colaborar com a equipe e reverter para versões anteriores, se necessário.

2.2. Versionamento com GitHub
Utilizamos GitHub para armazenar o código da State Machine, templates de infraestrutura e scripts de deploy.

Cada mudança é rastreada em um repositório Git, com branches, pull requests e code reviews.

2.3. Estrutura do Repositório
A estrutura do repositório deve ser organizada da seguinte forma:

Copy
state-machine/
  ├── state-machine.json          # Definição da State Machine
  ├── cloudformation-template.yml # Template de CloudFormation
  ├── terraform/                  # Configuração do Terraform
  │   ├── main.tf
  │   ├── variables.tf
  │   └── outputs.tf
  ├── scripts/                    # Scripts de deploy
  │   └── deploy.sh
  ├── tests/                      # Testes automatizados
  │   └── test-state-machine.js
  └── README.md                   # Documentação do projeto
3. CI/CD com GitHub Actions e AWS CLI
3.1. O que é GitHub Actions?
GitHub Actions é uma ferramenta de CI/CD integrada ao GitHub, que permite automatizar workflows de build, teste e deploy.

3.2. Workflow de CI/CD
O workflow de CI/CD é responsável por:

Build: Validar o código e os templates de infraestrutura.

Testes: Executar testes automatizados.

Deploy: Implantar a State Machine e os recursos associados usando o AWS CLI.

3.3. Exemplo de Workflow com GitHub Actions
yaml
Copy
name: State Machine CI/CD

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Validate State Machine
        run: |
          aws stepfunctions validate-state-machine-definition --definition file://state-machine.json

      - name: Validate CloudFormation Template
        run: |
          aws cloudformation validate-template --template-body file://cloudformation-template.yml

  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Run Tests
        run: |
          node tests/test-state-machine.js

  deploy-dev:
    runs-on: ubuntu-latest
    needs: [build, test]
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Configure AWS CLI
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy to DEV
        run: |
          aws cloudformation deploy --template-file cloudformation-template.yml --stack-name MyStateMachineStack-DEV --parameter-overrides Environment=DEV

  deploy-pre-prod:
    runs-on: ubuntu-latest
    needs: [deploy-dev]
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Configure AWS CLI
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy to PRE-PROD
        run: |
          aws cloudformation deploy --template-file cloudformation-template.yml --stack-name MyStateMachineStack-PRE-PROD --parameter-overrides Environment=PRE-PROD

  deploy-prod:
    runs-on: ubuntu-latest
    needs: [deploy-pre-prod]
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Configure AWS CLI
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy to PROD
        run: |
          aws cloudformation deploy --template-file cloudformation-template.yml --stack-name MyStateMachineStack-PROD --parameter-overrides Environment=PROD
4. Conclusão
Seguindo esta documentação, você poderá implementar State Machines com AWS Step Functions de forma controlada, automatizada e segura, utilizando:

Canary e Blue/Green Deployment para liberações graduais e rollback rápido.

GitHub para versionamento e colaboração.

GitHub Actions e AWS CLI para CI/CD e deploy automatizado.

Essa abordagem garante a qualidade, rastreabilidade e conformidade com os processos de mudança da organização.

5. Códigos ASCII para Copiar e Colar
5.1. Estrutura do Repositório
plaintext
Copy
state-machine/
  ├── state-machine.json
  ├── cloudformation-template.yml
  ├── terraform/
  │   ├── main.tf
  │   ├── variables.tf
  │   └── outputs.tf
  ├── scripts/
  │   └── deploy.sh
  ├── tests/
  │   └── test-state-machine.js
  └── README.md
5.2. Exemplo de Workflow GitHub Actions
yaml
Copy
name: State Machine CI/CD

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Validate State Machine
        run: |
          aws stepfunctions validate-state-machine-definition --definition file://state-machine.json

      - name: Validate CloudFormation Template
        run: |
          aws cloudformation validate-template --template-body file://cloudformation-template.yml

  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Run Tests
        run: |
          node tests/test-state-machine.js

  deploy-dev:
    runs-on: ubuntu-latest
    needs: [build, test]
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Configure AWS CLI
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy to DEV
        run: |
          aws cloudformation deploy --template-file cloudformation-template.yml --stack-name MyStateMachineStack-DEV --parameter-overrides Environment=DEV

  deploy-pre-prod:
    runs-on: ubuntu-latest
    needs: [deploy-dev]
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Configure AWS CLI
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy to PRE-PROD
        run: |
          aws cloudformation deploy --template-file cloudformation-template.yml --stack-name MyStateMachineStack-PRE-PROD --parameter-overrides Environment=PRE-PROD

  deploy-prod:
    runs-on: ubuntu-latest
    needs: [deploy-pre-prod]
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Configure AWS CLI
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Deploy to PROD
        run: |
          aws cloudformation deploy --template-file cloudformation-template.yml --stack-name MyStateMachineStack-PROD --parameter-overrides Environment=PROD
5.3. Exemplo de Template CloudFormation
yaml
Copy
Resources:
  MyStateMachine:
    Type: AWS::StepFunctions::StateMachine
    Properties:
      StateMachineName: MyStateMachine
      DefinitionS3Location: state-machine.json
      RoleArn: arn:aws:iam::123456789012:role/StepFunctionsExecutionRole

  MySQSQueue:
    Type: AWS::SQS::Queue
    Properties:
      QueueName: DynamoQueue
5.4. Exemplo de Script de Deploy (deploy.sh)
bash
Copy
#!/bin/bash

# Configura o ambiente
ENVIRONMENT=$1

# Valida o ambiente
if [[ ! "$ENVIRONMENT" =~ ^(DEV|PRE-PROD|PROD)$ ]]; then
  echo "Ambiente inválido. Use DEV, PRE-PROD ou PROD."
  exit 1
fi

# Deploy usando AWS CLI
aws cloudformation deploy \
  --template-file cloudformation-template.yml \
  --stack-name MyStateMachineStack-$ENVIRONMENT \
  --parameter-overrides Environment=$ENVIRONMENT
Com esses códigos e práticas, você terá uma base sólida para implementar State Machines com AWS Step Functions de forma eficiente e controlada.
