## Arquitetura

O ADDB armazena todos os seus dados em "blocos" de memória chamados de "partitions" ou particões. O endereçamento desses dados funciona como um hash-map, onde cada elemento terá uma partition alvo e essas partitions são distribuídas e replicadas em diversos servidores da AWS da região.

### Particão

O AWS aloca máquinas o banco de dados de acordo com a demanda e cada máquina que participa do "cluster" recebe uma "tag" com um valor inteiro no intervalo [0,2^64). Quando ocorre uma requisição para inserção de dados no banco, a chave do elemento passa por uma função hash que retorna um inteiro no intervalo anterior, o dado é então guardado na primeira máquina encontrada, a busca pela máquina é realizada em um esquema de "relógio" de acordo com a imagem seguinte.

<figure class="image">
    <img src="imagens\arquitetura_dynamo.PNG" alt='Busca da máquina-alvo'>
    <figcaption>Busca da máquina-alvo</figcaption>
</figure>

No exemplo da imagem acima, se a função hash retornar 100000, o dado será armazenado na máquina A.

Caso uma máquina fique sobrecarregada de dados o AWS aloca máquinas adicionais para o banco. A nova máquina armazenará todos os dados do seu novo intervalo.

<figure class="image">
    <img src="imagens\arquitetura_dynamo_nova_maquina.PNG" alt='adicao de uma nova maquina'>
    <figcaption>Adição da máquina "D" ao cluster</figcaption>
</figure>

No exemplo acima todos os dados entre os endereços 10000 e 15000 seão transferidos para a máquina D. Caso uma máquina fique subutilizada, o banco pode remover uma máquina e transferir seus dados para a próxima máquina.

### Replicação

Para evitar perda de dados caso haja um problema em alguma máquina, quando ocorre uma requisição de inserção no banco, a AWS replica a inserção da máquina-alvo nas N-1 próximas máquinas.

<figure class="image">
    <img src="imagens\arquitetura_dynamo_replicacao.PNG" alt='Replicação da requisição nas N-1 próximas máquinas (N=3)'>
    <figcaption>Replicação da requisição nas N-1 próximas máquinas (N=3)</figcaption>
</figure>

Para evitar perda de desempenho na operação de escrita, o banco enviará a resposta de conclusão da escrita caso uma quantidade W de máquinas termine a operação.

Um problema desse método é que uma requisição de leitura pode tentar recuperar dados de uma máquina que não tenha completado o processo de escrita, retornando dados inválidos ou desatualizados. Para resolver este problema o banco lê R cópias do dado entre as máquinas do cluster, retornando o valor mais recente.

Para garantir que a operação seja um sucesso, o valor de R+W deve ser superior ao valor de N, dessa maneira garante-se que o dado mais recente será recuperado.