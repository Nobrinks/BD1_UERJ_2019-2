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

DynamoDB é um banco de dados NoSQL fornecido pela Amazon Web Service (AWS).Visto de longe, 
DynamoDB é basicamente um armazenamento de chaves-valores. Pode ser entendido como
um "hash-map" ajudado por alguns armazenamentos resistentes. 
A duas operações mais importantes são: Get e Put.

O que diferencia o DynamoDB é sua extrema rapidez em *particionamentos*.
Para conseguir um escalonamento horizontal, ele atribui dados para diferentes
partições que são hospedados em maquinas distintas, e quanto mais dados 
mais partições e maquinas são utilizadas, sob demanda.


-----------------

A PRIMEIRA IMAGEM DO CIRCULO! 

-----------------
Quando inserimos um par de valores-chave no DynamoDB,a chave é primeiro transformada em um inteiro, I.
O par de valores-chave é então armazenado na máquina que encontraremos 
primeiro se começarmos de I e andarmos no sentido horário ao redor do anel . 
Portanto, as chaves que são hash para (1000, 2⁶⁴-1] e [0, 100] 
são armazenadas na máquina A. Portanto, a máquina B armazena as chaves
cujos valores de hash estão entre 100 e 2000. O resto é armazenado na máquina C.


Quando se recebe muitos dados, a ponto de sobrecarregar uma maquina pode-se
adicionar outra maquina que irá pegar parte dos dados da maquina sobrecarregada.
Um Token então é gerado de maneira diferente para a nova maquina para demonstrar que
contém os parte dos dados da máquina sobrecarregada.

Se um conjunto de Tokens recebe muitos dados pode-se progressivamente criar mais máquinas,
e fazer o dado ficar distribuido entre mais máquinas do mesmo conjunto de tokens.
Se muitos dados começarem a serem deletados, e algumas maquinas começarem a ficar liberadas,
pode-se juntar várias maquinas para comportar todos os dados dessas maquinas com poucos dados

DynamoDB também replica os dados N vezes pelas outras maquinas.
(Assumiremos N = 3 pois é o mais comum no AWS)

Quando se recebe um request de salvar dados, ele passa ele para as duas próximas máquinas.
Logo o dado estará armazenado na máquina A, B e C. Então A só manda uma resposta pro cliente de
salvo, quando ele receber uma resposta de B e C comprovando que neles foram salvos também.

Porém não necessita de resposta de todas as máquinas que foram feitas as replicas, ele só precisa
da resposta de W maquinas garantir que foi salvo.(W geralmente é 2 no AWS). Dessa forma haverá um retorno
para o cliente quando 2 dos 3 eventos for verdade:

* A escreveu no disco.
* Recebeu uma resposta de B.
* Recebeu uma resposta de C.

No entanto pode ocorrer de A não conseguir escrever o dado devido a um erro. DynamoDB então,
necessita de R copias do dado e retornar a copia mais atualizada para o cliente. Dessa forma 
A é necessário pegar uma cópia do dado de pelo menos um de B ou C e então retornar para o Cliente.

Com particionamento e replicação, o DynamoDB é,
portanto, capaz de fornecer um serviço escalonável e confiável.

## Teoria: descrever como ocorre o processamento de consultas no BD, componente envolvidos;
[How it works](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_Query.html)
## Teoria: descrever como ocorre a otimização de consultas no BD, componente envolvidos;
[Index](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/SQLtoNoSQL.Indexes.Creating.html)

[Scans vs queries](https://medium.com/redbox-techblog/tuning-dynamodb-scans-vs-queries-110ef6c3f671)
## Teoria: descrever como ocorre o controle de transações no BD, confirmação, rollback, tipos de bloqueios, níveis de isolamento;
[Transaction](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/transaction-apis.html)
## Teoria: descrever como ocorre o controle de concorrência no BD;
[Concurrency using optimistic locking](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBMapper.OptimisticLocking.html)
[Read Consistency](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadConsistency.html)
## Teoria: descrever sobre a segurança no BD, controle de acesso, concessão e revogação de privilégio, existência ou não de criptografia de dados;
[Database security](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/security.html)
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
