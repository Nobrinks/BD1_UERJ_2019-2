# Prática

## Executar UMA inserção de dados

`aws dynamodb put-items --table-name Music --item file://insert_item.json --endpoint-url http://localhost:8000`

## Executar UMA consulta aos dados do BD

`aws dynamodb get-items --table-name Music --key file://keys.json --endpoint-url http://localhost:8000`

## Como gerar plano de execução no BD, caso tenha a possibilidade

Não tem como

## Exemplo de 1 cenário de otimização de consulta

* Criação
`aws dynamodb update-table --table-name Music --attribute-definitions file://index_creation_keys.json --endpoint-url http://localhost:8000`

* Consulta
`aws dynamodb query --index-name SongTitleIndex --key-condition-expression "Song = Beyond the Gates of Infinity" --endpoint-url http://localhost:8000`

## Exemplo de 1 concessão de privilégio com execução de um comando que demonstre esse privilégio com sucesso de

## Exemplo de 1 revogação de privilégio com tentativa de execução de um comando proibindo e gerando erro