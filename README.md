# mbauspesalq-ecommerce-infra

## Objetivo

Este repositório disponibiliza o `docker-compose.yml` para subir a infraestrutura necessária para rodar os
microsserviços
[ecommerce-pagamentos](https://github.com/mbauspesalq-ecommerce/ecommerce-pagamentos) e
[ecommerce-estoque](https://github.com/mbauspesalq-ecommerce/ecommerce-estoque).

Para mais informações sobre cada microsserviço em específico, acessar os respectivos repositórios:

- https://github.com/mbauspesalq-ecommerce/ecommerce-pagamentos
- https://github.com/mbauspesalq-ecommerce/ecommerce-estoque

Juntos, esses microsserviços orquestram o fluxo de pagamento do e-commerce, garantindo a separação clara de domínios:

- **ecommerce-estoque**: Gerencia a disponibilidade e controle do estoque dos produtos.
- **ecommerce-pagamentos**: Processa pagamentos, verifica a disponibilidade dos produtos em estoque através de APIs do
  microsserviço ecommerce-estoque dos produtos e mantém o estado das transações.

## Arquitetura e Integrações

Os microsserviços se comunicam entre si para garantir um fluxo coeso de compra e pagamento, com separação explícita do
domínio de cada aplicação, e com APIs bem definidas de comunicação:

1. O **ecommerce-pagamentos** recebe uma solicitação de pagamento contendo `idCliente`, `idCarrinho`, `produtos`
   e `pagamento`. Ele então calcula o valor total do carrinho. Abaixo está o payload de entrada do endpoint
   POST http://localhost:8001/api/pagamentos.

      ```json
      {
        "idCliente": "123456",
        "idCarrinho": "654321",
        "produtos": [
          {
            "id": 1,
            "precoUnitario": 100.00,
            "quantidadeRequerida": 1
          },
          {
            "id": 2,
            "precoUnitario": 200.00,
            "quantidadeRequerida": 3
          }
        ],
        "dadosPagamento": {
          "tipo": 1,
          "parcelas": 2,
          "simbolo": "VISA"
        }
      }
      ```
2. O serviço consulta o **ecommerce-estoque** para verificar e reservar a quantidade necessária dos produtos.
3. Se os produtos estiverem disponíveis, o estoque é reduzido temporariamente e a solicitação de pagamento segue para
   uma **API de pagamento parceira**.
4. Caso o pagamento seja aprovado:
    - O estado da transação é salvo no **DynamoDB** com `estado = "APROVADO"`.
5. Se o pagamento for negado:
    - O estoque dos produtos reservados é devolvido.
    - O estado da transação é salvo no **DynamoDB** com `estado = "NEGADO"`.

Obs.: para que o pagamento seja NEGADO pela API parceira (wiremock), o atributo `dadosPagamento::tipo` deve ser
diferente de 1. Assim conseguimos validar se o microsserviço de pagamentos devolve os produtos para o microsserviço de
estoque, disponibilizando-os para outros clientes comprar.

## Tecnologias Utilizadas

- Docker & Docker Compose
- PostgreSQL (para o serviço de estoque)
- DynamoDB (via LocalStack para o serviço de pagamentos)
- Spring Boot (para ambos os microsserviços)
- Kotlin
- JDK 17
- Wiremock (para simulação da API de pagamento externa)

## Como Executar

### Requisitos

Antes de iniciar, certifique-se de ter instalado:

- Docker
- Docker Compose

### Passos para execução

1. Clone este repositório:

   ```bash
   git clone https://github.com/mbauspesalq-ecommerce/ecommerce-infra.git
   cd ecommerce-infra
   ```

2. Suba toda a infraestrutura do e-commerce com o seguinte comando:

   ```bash
   docker-compose up -d
   ```

3. Verifique se os serviços estão rodando corretamente:

   ```bash
   docker ps
   ```

4. Agora, você pode acessar os microsserviços:
    - **ecommerce-estoque**: `http://localhost:8000/api/produtos`
    - **ecommerce-pagamentos**: `http://localhost:8001/api/pagamentos`
    - Utilizar collection do insomnia em: `/collection`

## Conclusão

Este repositório fornece toda a infraestrutura necessária para rodar os microsserviços de estoque e pagamentos de forma
integrada, permitindo testar e validar o fluxo completo do e-commerce com microsserviços bem definidos e independentes.
