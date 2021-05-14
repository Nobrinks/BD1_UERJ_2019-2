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

![Arquitetura Dynamo DB](https://github.com/Nobrinks/BD2_UERJ_2020-2/blob/trabalho2/imagens/arquitetura_dynamo.PNG)

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

Com as transações da Amazon DynamoDB, você pode agrupar várias ações e submetê-las como uma única operação de tudo-ou-nada com a TransactWriteItems ou TransactGetItems. As seções seguintes descrevem operações da API, gerenciamento de capacidade e outros detalhes sobre o uso de operações transacionais no DynamoDB. 

### TransactWriteItems 
TransactWriteItems é uma operação de escrita síncrona e idempotente que agrupa até 25 ações de escrita em uma única operação de tudo-ou-nada. Essas ações podem ter como alvo até 25 itens distintos em uma ou mais tabelas DynamoDB dentro da mesma conta AWS e na mesma Região. O tamanho agregado dos itens na transação não pode exceder 4 MB. As ações são completadas atomicamente de forma que ou todas elas sejam bem sucedidas ou nenhuma delas seja bem sucedida. 

Uma operação TransactWriteItems difere de uma operação BatchWriteItem em que todas as ações nela contidas devem ser completadas com sucesso, ou nenhuma alteração é feita. Com uma operação BatchWriteItem, é possível que apenas algumas das ações do lote sejam bem sucedidas enquanto as outras não.
Não é possível visar o mesmo item com múltiplas operações dentro da mesma transação. Por exemplo, você não pode realizar um ConditionCheck e também uma ação de atualização sobre o mesmo item na mesma transação. 

Você pode adicionar os seguintes tipos de ações a uma transação: 

* Put - Inicia uma operação PutItem para criar um novo item ou substituir um item antigo por um novo item, condicionalmente ou sem especificar qualquer condição. 

* Update - Inicia uma operação UpdateItem para editar os atributos de um item existente ou adicionar um novo item à tabela se ele ainda não existir. Use esta ação para adicionar, excluir ou atualizar atributos em um item existente condicionalmente ou sem uma condição. 

* Delete - Inicia uma operação DeleteItem para excluir um único item em uma tabela identificada por sua chave primária. 

* ConditionCheck - Verifica se um item existe ou verifica a condição de atributos específicos do item. 

Uma vez concluída uma transação, as mudanças feitas dentro dessa transação são propagadas para índices secundários globais (GSIs), fluxos e backups. Como a propagação não é imediata ou instantânea, se uma tabela for restaurada do backup para um ponto médio de propagação, ela pode conter algumas, mas não todas, as mudanças feitas durante uma transação recente. 

#### Idempotência

Opcionalmente, você pode incluir um _token_ de acesso ao fazer uma chamada TransactWriteItems para garantir que a solicitação seja idempotente. Fazer suas transações idempotentes ajuda a evitar erros de aplicação se a mesma operação for submetida várias vezes devido a um timeout de conexão ou outro problema de conectividade. 

Se a chamada TransactWriteItems original foi bem sucedida, as chamadas TransactWriteItems subsequentes com o mesmo _token_ de acesso retornam com sucesso sem fazer nenhuma alteração. Se o parâmetro ReturnConsumedCapacity for definido, a chamada TransactWriteItems inicial retorna o número de unidades de capacidade de gravação consumidas na realização das mudanças. Chamadas TransactWriteItems subsequentes com o mesmo _token_ de acesso retornam o número de unidades de capacidade de leitura consumidas na leitura do item. 

Pontos importantes sobre a idempotência: 

* Uma ficha de cliente é válida por 10 minutos após o término da solicitação que a utiliza. Após 10 minutos, qualquer pedido que utilize o mesmo _token_ de acesso é tratado como um novo pedido. Não se deve reutilizar o mesmo _token_ de acesso para a mesma solicitação após 10 minutos. 

* Se você repetir uma solicitação com o mesmo _token_ de acesso dentro da janela de idempotência de 10 minutos, mas alterar algum outro parâmetro de solicitação, a DynamoDB retorna uma exceção IdempotentParameterMismatch. 

#### Tratamento de erros de escrita 

As transações de escrita não são bem sucedidas nas seguintes circunstâncias: 

* Quando uma condição em uma das expressões de condição não é satisfeita. 

* Quando um erro de validação de transação ocorre porque mais de uma ação na mesma TransactWriteItems operação visa o mesmo item.  

* Quando uma solicitação TransactWriteItems entra em conflito com uma operação TransactWriteItems em andamento em um ou mais itens da solicitação TransactWriteItems. Neste caso, a solicitação falha com uma TransactionCanceledException.  

* Quando não há capacidade provisionada suficiente para que a transação seja concluída. 

* Quando um item se torna muito grande (maior que 400 KB), ou um índice secundário local (LSI) torna-se muito grande, ou um erro de validação similar ocorre devido a mudanças feitas pela transação. 

* Quando há um erro do usuário, tal como um formato de dados inválido. 

 

### TransactGetItems API 

A TransactGetItems é uma operação de leitura síncrona que agrupa até 25 ações. Essas ações podem direcionar até 25 itens distintos em uma ou mais tabelas DynamoDB dentro da mesma conta AWS e Região. O tamanho agregado dos itens na transação não pode exceder 4 MB.  

As ações Get são realizadas atomicamente para que ou todas elas tenham sucesso ou todas elas falhem: 

* Get - Inicia uma operação GetItem para recuperar um conjunto de atributos para o item com a chave primária dada. Se nenhum item correspondente for encontrado, Get não retorna nenhum dado. 

#### Tratamento de erros de leitura 

As transações lidas não têm sucesso sob as seguintes circunstâncias: 

* Quando uma solicitação TransactGetItems entra em conflito com uma operação TransactWriteItems em andamento em um ou mais itens da solicitação TransactGetItems. Neste caso, a solicitação falha com uma TransactionCanceledException. 
* Quando não há capacidade provisionada suficiente para que a transação seja concluída. 
* Quando há um erro do usuário, tal como um formato de dados inválido. 

### Níveis de Isolamento para Transações DynamoDB 

Os níveis de isolamento das operações transacionais (TransactWriteItems ou TransactGetItems) e outras operações são os seguintes. 

  

### SERIALIZÁVEL 

Isolamento serializável garante que os resultados de múltiplas operações simultâneas sejam os mesmos como se nenhuma operação tivesse começado até que a anterior tivesse terminado. 

Existe um isolamento serializável entre os seguintes tipos de operação: 

* Entre qualquer operação transacional e qualquer operação de escrita padrão (PutItem, UpdateItem, ou DeleteItem). 

* Entre qualquer operação transacional e qualquer operação de leitura padrão (GetItem). 

* Entre uma operação de TransactWriteItems e uma operação de TransactGetItems. 

Embora haja um isolamento serializável entre as operações transacionais, e cada padrão individual escreve em uma operação BatchWriteItem, não há isolamento serializável entre a transação e a operação BatchWriteItem como uma unidade. 

Da mesma forma, o nível de isolamento entre uma operação transacional e uma GetItems individual em uma operação BatchGetItem é serializável. Mas o nível de isolamento entre a transação e a operação BatchGetItem como uma unidade é _read-committed_. 

Uma única solicitação GetItem é serializável com relação a uma solicitação TransactWriteItems em uma das duas formas, antes ou depois da solicitação TransactWriteItems. Múltiplas solicitações GetItem, contra chaves em uma solicitação TransactWriteItems concorrente, podem ser executadas em qualquer ordem e, portanto, os resultados são readcomissionados. 

Por exemplo, se as solicitações GetItem para o item A e item B forem executadas simultaneamente com uma solicitação TransactWriteItems que modifica tanto o item A quanto o item B, há quatro possibilidades:  

* Ambas as solicitações GetItem são executadas antes da solicitação TransactWriteItems. 
* Ambas as solicitações GetItem são executadas após a solicitação TransactWriteItems. 
* A solicitação GetItem para o item A é executada antes da solicitação TransactWriteItems. Para o item B, a solicitação GetItem é executada após a solicitação TransactWriteItems. 
* A solicitação GetItem para o item B é executada antes da solicitação TransactWriteItems. Para o item A, a solicitação GetItem é executada após a TransactWriteItems. 

Se o nível de isolamento serializável for preferível para múltiplas solicitações GetItem, então use TransactGetItems. 

### READ-COMMITTED 

O isolamento do _read-committed_ garante que as operações lidas sempre retornem valores comprometidos para um item. O isolamento de leitura-compromisso não impede modificações do item imediatamente após a operação lida.O nível de isolamento é _read-committed_ entre qualquer operação transacional e qualquer operação de leitura que envolva múltiplas leituras padrão (BatchGetItem, Query, ou Scan). Se uma transação escrita atualiza um item no meio de uma operação BatchGetItem, Query ou Scan, a operação lida retorna o novo valor comprometido.

## Teoria: descrever como ocorre o controle de concorrência no BD;
[Concurrency using optimistic locking](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBMapper.OptimisticLocking.html)
[Read Consistency](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadConsistency.html)
## Teoria: descrever sobre a segurança no BD, controle de acesso, concessão e revogação de privilégio, existência ou não de criptografia de dados;
[Database security](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/security.html)
## Teoria: descrever formas de backup e recuperação do BD, exemplificando com os comandos necessários;
## Backup e Restauração
o ADDB oferece duas formas de back-up a partir da AWS, Point-in-time backup e On-demand backup, ambos descritos a seguir.

### Point-in-time backup

Este modo de backup oferece a opção de restaurar o banco de dados para qualquer ponto a partir do momento em que ele é ativado (até no máximo 35 dias). O processo restaura a tabela antiga em uma nova tabela. Vale ressaltar que todas as configurações da tabela se mantarão iguais a tabela mais recente.

No exemplo abaixo ativa-se o Point-in-time backup para a tabela music:

```txt
aws dynamodb update-continuous-backups \
    --table-name Music \
    --point-in-time-recovery-specification PointInTimeRecoveryEnabled=True 
```
Logo após recupera-se o backup mais recente disponível(usualmente retorna os ultimos 5 minutos):

```txt
aws dynamodb restore-table-to-point-in-time \
    --source-table-name Music \
    --target-table-name MusicMinutesAgo \
    --use-latest-restorable-time
```

Também é possível recuperar para um horário específico:

```txt
aws dynamodb restore-table-to-point-in-time \
    --source-table-name Music \
    --target-table-name MusicEarliestRestorableDateTime \
    --no-use-latest-restorable-time \
    --restore-date-time 1519257118.0
```

### On-demand backup

É possível criar backups sob demanda com comandos como:

```txt
aws dynamodb create-backup --table-name Music \
 --backup-name MusicBackup
```

O backup on-demand é criado de maneira assíncrona, ò usuário ainda pode executar querys nas tabelas que estão sendo salvas, mas não é possível pausar ou cancelar a operação e remover a tabela.<br><br>
Após a criação do backup é possível listar todos backups:
```txt
aws dynamodb list-backups
```
E por fim é possível restaurar a tabela Music:

```txt
aws dynamodb restore-table-from-backup \
--target-table-name Music \
--backup-arn arn:aws:dynamodb:us-east-1:123456789012:table/Music/backup/01489173575360-b308cd7d 
```

<!--
Temos duas formas de fazer [backup](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/BackupRestore.html). A [on-demand](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/backuprestore_HowItWorks.html) e a [point-in-time](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/PointInTimeRecovery_Howitworks.html).
A on-demand tem que fazer o [backup](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Backup.Tutorial.html) e o [restore](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Restore.Tutorial.html).
<<<<<<< HEAD
A point-int-time o backup é automatico e o [restore](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/PointInTimeRecovery.Tutorial.html) pode ser por console ou AWS CLI -->
=======

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
