# Trabalho da disciplina banco de dados II - UERJ 2020/2

## Integrantes

* Dennis Ribeiro Paiva - 201610050611
* Igor Sousa Silva - 201610053411
* Paulo Victor Coelho - 201610049711
* Vinicius Sathler - 201610051611
  
## Introdução

A Amazon DynamoDB é um banco de dados de Chave-Valor (Key-Value) e de documentos. Cada chave só pode ter um valor.

Na ADDB as tabelas são guardadas em partições (Partitions) que seriam a parte fisica, como o ADDB lida com isso é totalmente transparente para a nossa aplicação. Só sabemos que ele tem SSDs fazendo a alocação e replicados automatricamente entre diferentes Data Centers da AWS. Lembrando que ele é alocado inicialmente no Data center mais proximo e depois replicado.
[here](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.Partitions.html)

Precisamos definir uma Partition Key, que seria uma Primary Key no relacional.
Caso tenhamos o mesmo valor de Partition Key, existe a Sort Key e então it's possible for two items to have the same partition key value. However, those two items must have different sort key values.


## Teoria: Descrição do BD e sua estrutura (relacional, orientado a colunas, documentos, XML etc);

Amazon DynamoDB is a key-value and document database that delivers single-digit-millisecond performance at any scale. It's a fully managed, multi-region, multi-master, durable database with built-in security, backup and restore, and in-memory caching for internet-scale applications

## Teoria: Arquitetura (Desenho da arquitetura dos componentes do BD);
[Arquitetura da vida](https://medium.com/swlh/architecture-of-amazons-dynamodb-and-why-its-performance-is-so-high-31d4274c3129#:~:text=DynamoDB%20uses%20a%20cluster%20of,the%20DynamoDB%20has%203%20machines.)
## Teoria: descrever como ocorre o processamento de consultas no BD, componente envolvidos;
[How it works](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_Query.html)
## Teoria: descrever como ocorre a otimização de consultas no BD, componente envolvidos;
[Index](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/SQLtoNoSQL.Indexes.Creating.html)
[Scans vs queries](https://medium.com/redbox-techblog/tuning-dynamodb-scans-vs-queries-110ef6c3f671)
## Teoria: descrever como ocorre o controle de transações no BD, confirmação, rollback, tipos de bloqueios, níveis de isolamento;
## Teoria: descrever como ocorre o controle de concorrência no BD;
## Teoria: descrever sobre a segurança no BD, controle de acesso, concessão e revogação de privilégio, existência ou não de criptografia de dados;
## Teoria: descrever formas de backup e recuperação do BD, exemplificando com os comandos necessários;
Temos duas formas de fazer [backup](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/BackupRestore.html). A [on-demand](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/backuprestore_HowItWorks.html) e a [point-in-time](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/PointInTimeRecovery_Howitworks.html).
A on-demand tem que fazer o [backup](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Backup.Tutorial.html) e o [restore](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Restore.Tutorial.html).
A point-int-time o backup é automatico e o [restore](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/PointInTimeRecovery.Tutorial.html) pode ser por console ou AWS CLI
## Prática: instalação do BD localmente;
## Prática:
* Executar UMA inserção de dado;
* Executar UMA consulta aos dados do BD;
* Como gerar plano de execução no BD, caso tenha a possibilidade;
* Exemplo de 1 cenário de otimização de consulta;
* Exemplo de criação de um usuário;
* Exemplo de uma concessão de privilégio com execução de um comando que demonstre esse privilégio com sucesso;
* Exemplo de uma revogação de privilégio com tentativa de execução de comando proibido e gerando erro;
## Exemplos retirados da própria documentação do banco - NÃO ESQUECER DE COLOCAR AS REFERÊNCIAS! CUIDADO COM PLÁGIOS!
