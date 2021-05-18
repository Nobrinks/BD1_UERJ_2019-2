# Commands

## Scan

`aws dynamodb scan --table-name Music --endpoint-url http://localhost:8000`

## Query

* List tables

`aws dynamodb list-tables --endpoint-url http://localhost:8000`

* Put items

`aws dynamodb put-items --table-name Music --item file://insert_item.json --endpoint-url http://localhost:8000`

* Get items
`aws dynamodb get-items --table-name Music --key file://key.json --endpoint-url http://localhost:8000`

* Update

`aws dynamodb update-item --table-name Music --key file://key.json --endpoint-url http://localhost:8000`
