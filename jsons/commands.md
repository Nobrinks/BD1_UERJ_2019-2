# Prática

## Instalação do DynamoDB localmente

Para instalar o aws-cli:

1) Tem que instalar o java jdk e jre>8
2) https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-windows.html

Para configurar o aws cli

``` txt
aws configure
    AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
    AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
    Default region name [None]: us-west-2
    Default output format [None]: json
```

Para rodar o servidor do banco localmente:

``` txt
java -Djava.library.path=./DynamoDBLocal_lib -jar DynamoDBLocal.jar -sharedDb
```

Para criar o tabela:

`aws dynamodb create-table --table-name Music --attribute-definitions file://create_table_attribute_def.json --key-schema file://create_table_key_schema.json --billing-mode PAY_PER_REQUEST --endpoint-url http://localhost:8000`

## Executar UMA inserção de dados

`aws dynamodb put-item --table-name Music --item file://insert_item.json --endpoint-url http://localhost:8000`

## Executar UMA consulta aos dados do BD

`aws dynamodb get-item --table-name Music --key file://query_keys.json --endpoint-url http://localhost:8000`

## Como gerar plano de execução no BD, caso tenha a possibilidade

Não tem como

## Exemplo de 1 cenário de otimização de consulta

* Criação
`aws dynamodb update-table --table-name Music --attribute-definitions file://index_creation_keys.json --global-secondary-index-updates file://create_index.json --endpoint-url http://localhost:8000`

* Consulta
`aws dynamodb query --table-name Music --index-name TrackTitleIndex --key-condition-expression "Album = :albumName" --expression-attribute-values file://expression_att_values.json --endpoint-url http://localhost:8000`


* Deleção
`aws dynamodb update-table --table-name Music --global-secondary-index-updates file://delete_index.json --endpoint-url http://localhost:8000`


## Exemplo de 1 concessão de privilégio com execução de um comando que demonstre esse privilégio com sucesso


## Exemplo de 1 revogação de privilégio com tentativa de execução de um comando proibindo e gerando erro


## Transact

`aws dynamodb transact-write-items --transact-items file://populate.json --endpoint-url http://localhost:8000`

## Delete

`aws dynamodb delete-table --table-name Music --endpoint-url http://localhost:8000`

## Scan

`aws dynamodb scan --table-name Music --endpoint-url http://localhost:8000`
