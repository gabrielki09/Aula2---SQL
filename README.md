## DDL (Data Definition Language)

Dentro do SQL existem subconjuntos de comandos auxiliares com propriedades e utilidades específicas.

- <h4 style="color: #00897B">CREATE</h4>
O comando CREATE do SQL será o responsável basicamente por criar a estrutura do banco de dados, como tabelas e index com um:
```sql
CREATE TABLE pessoas(); -- Criação de uma tabela
CREATE INDEX idx_pessoa_email ON pessoas(email) -- Criação de um index de e-mail da tabela pessoas
```
Ou objetos mais complexos como procedures para outro tipo de mecanismo:

```sql
CREATE PROCEDURE <NomeDaProcedure>
    @<Parametro1> <tipo de dado>,
    @<Parametro2> <tipo de dado>
AS

    SET NOCOUNT ON;
    SELECT <seu select>;
GO
```

---

- <h4 style="color: #F57C00">ALTER</h4>
Depois que objetos são criados, por exemplo:
```sql
CREATE TABLE pessoas(
    id INT IDENTITY(1, 1) NOT NULL,
    CONSTRAINT pk_pessoas_id PRIMARY KEY(id)
); -- Criação de uma tabela
```
suas colunas já foram definidas, onde nesse ponto já não seria mais possível inserir novas colunas pelo create table, por ser apenas o responsável pela criação da tabela, com isso o comando ALTER entra para podermos modificar objetos já criados, por exemplo:

```sql
CREATE TABLE pessoas(
    id INT IDENTITY(1, 1) NOT NULL,
    CONSTRAINT pk_pessoas_id PRIMARY KEY(id)
); -- Criação inicial da tabela pessoas

ALTER TABLE pessoas ADD email VARCHAR(255) NOT NULL; -- Adição de uma nova coluna chamada e-mail, do tipo VARCHAR com tamanho máximo de 255 caracteres e a mesma não pode ser null;

ALTER TABLE pessoas ADD CONSTRAINT uq_email UNIQUE(email); -- Adição de uma constraint de restrição individual, de e-mails únicos na tabela pessoas, isso indica que só poderá haver 1 e-mail sem o mesmo estar repetido;

ALTER TABLE pessoas ADD last_name VARCHAR(120) NOT NULL DEFAULT ''; -- Adição de mais uma nova coluna chamada last_name do tipo VARCHAR com tamanho máximo de 120 caracteres, não podendo ser null e com valor padrão '';

-- ALTER TABLE pessoas DROP CONSTRAINT DF__pessoas__last_na__38B96646; -- Constraint Default foi criada então a mesma também precisará ser removida antes

ALTER TABLE pessoas DROP COLUMN last_name; -- Do mesmo jeito que colunas podem ser adicionadas, colunas podem ser removidas, com
```

Nesse processo a tabela de pessoas teve a adição de uma coluna chamada e-mail e outra de last_name, caso um select fosse rodado antes de ser 'dropado', a coluna de last_name, teriamos o resultado de:
```sql
SELECT
    * -- Seleciona todas as colunas presentes na tabela
FROM
    pessoas
```
| id | email | last_name |
|-------:|--------------------------------------|------------------------------------------------|

Após:
```sql
ALTER TABLE pessoas DROP CONSTRAINT DF__pessoas__last_na__38B96646; 
ALTER TABLE pessoas DROP last_name;
```

O resultado será:
| id | email | 
|-------:|--------------------------------------|


> É importante ressaltar que deves-se possuir um cuidado com colunas que outras tabelas tenham dependência usando em conjunto o comando `CASCADE`


--- 

- <h4 style="color: #C62828">DROP</h4>
Objetos que são criados, também pode ser deletados por completo, comando DROP, podemos remover esses objetos com o comando:
```sql
DROP TABLE pessoas; -- Este comando irá 'dropar' diretamente a tabela específicada, se a mesma não existir será retornado erro;

DROP TABLE IF EXISTS pessoas; -- Este comando irá 'dropar' diretamente a coluna caso a mesma exista, se não existir não é retornado erro;

DROP INDEX IF EXISTS idx_pessoa_email ON pessoas; -- E também outros tipos de objetos como indexs caso o mesmo exista, informado qual o nome do index e de qual tabela;
```

--- 

- <h4 style="color: #7B1FA2">TRUNCATE</h4>
Truncate quase teria uma mesma ideia de que o comando DROP, porém o comando DROP elimina a tabela especificada, enquanto o comando TRUNCATE elimina apenas os dados, mas mantém a estrutura da tabela presente no banco de dados, sem remover a tabela, exemplo:

```sql
INSERT INTO pessoas(name) VALUES ('teste 1');
INSERT INTO pessoas(name) VALUES ('teste 2');
SELECT * FROM pessoas;
```

O resultado será:
| id | name | 
|-------:|--------------------------------------|
| 1 | teste 1
| 2 | teste 2

Após ser feito a limpa da tabela:
```sql
TRUNCATE TABLE pessoas;
```

```sql
SELECT * FROM pessoas;
```

O resultado será:
| id | name | 
|-------:|--------------------------------------|


> Por padrão as clausulas ON DELETE precisam ser definidas nas tabelas dependentes para que o 'CASCADE' possa ser aplicado

---

- <h4 style="color: #1565C0">RENAME</h4>
Rename poderia ser considerado um comando mais 'baixo', por mexer diretamente nas estruturas do banco de dados, por exemplo para a alteração do nome de uma tabela:

```sql
EXEC sp_rename 'dbo.NomeTabelaAntiga', 'dbo.NovoNomeTabela';

-- Também podendo ser usado para colunas, porém com específicação de que será a alteração em uma coluna com o banco.NomeDaTabela.NomeColuna

EXEC sp_rename 'dbo.TabelaAntiga.NomeColunaAntiga', 'NovoNome', 'COLUMN';
```

E até com as instâncias de serviços do banco de dados em si:
```sql
EXEC sp_dropserver 'NomeAntigo';
GO

EXEC sp_addserver 'NomeNovo', 'local';
GO
```

---

- <h4 style="color: #37474F">COMMENT</h4>
Pode-se fazer comentários nos scripts SQL:

```sql
-- Comentário de linha única;
SELECT * FROM pessoas;

/**
Comentário de mais de uma linha:
 */
SELECT * FROM pessoas;
```

---

## Exercícios:

#### Exercício 1
```sql
CREATE TABLE clientes(
    id INT IDENTITY(1, 1) NOT NULL,
    CONSTRAINT pk_customers_id PRIMARY KEY(id)
);

ALTER TABLE clientes ADD estado CHAR(2) NULL;
ALTER TABLE clientes ADD cep CHAR(8) NULL;
ALTER TABLE clientes ADD complemento VARCHAR(100) NULL;

ALTER TABLE clientes 
ADD CONSTRAINT ck_clientes_estado
CHECK (len(estado) = 2);
```
---
#### Exercício 2
```sql
CREATE TABLE fornecedores (
    id INT IDENTITY(1, 1) NOT NULL,
    nome VARCHAR(255) NOT NULL,
    cnpj CHAR(14) NOT NULL UNIQUE, 
    email VARCHAR(200) NOT NULL,
    ativo BIT DEFAULT 1,
    --deleted_at TIMESTAMP NULL

    CONSTRAINT pk_fornecedores_id PRIMARY KEY(id)
);

CREATE TABLE produtos (
    id INT IDENTITY(1, 1) NOT NULL,
    nome VARCHAR(255) NOT NULL,
    fornecedor_id INT NOT NULL,
    CONSTRAINT pk_produtos_id PRIMARY KEY(id),
    CONSTRAINT fk_produtos_fornecedor_id 
        FOREIGN KEY(fornecedor_id)
        REFERENCES fornecedores(id)
); 
```
---
#### Exercício 3
```sql
CREATE TABLE pedidos (
    id INT IDENTITY(1, 1) NOT NULL,
    data_pedido DATE NOT NULL,
    CONSTRAINT pk_pedidos_id PRIMARY KEY(id)
);

ALTER TABLE pedidos 
ADD CONSTRAINT ck_pedidos_data_pedido
CHECK (data_pedido <= getdate())

INSERT INTO pedidos(data_pedido) VALUES ('2062-01-02'); -- A instrução INSERT conflitou com a restrição do CHECK "CK__pedidos__data_pe__5535A963". O conflito ocorreu no banco de dados "aula_2", tabela "dbo.pedidos", column 'data_pedido'.
INSERT INTO pedidos(data_pedido) VALUES ('2026-01-02'); -- (1 linha afetada)
SELECT * FROM pedidos;

```
---

#### Exercício 4
```sql
CREATE TABLE itens_pedido(
    id INT IDENTITY(1, 1) NOT NULL,
    data_pedido DATE NOT NULL,
    quantidade DECIMAL(14, 2) NOT NULL,
    preco_unitario DECIMAL(14, 2) NOT NULL,
    pedido_id INT NOT NULL,
    CONSTRAINT pk_itens_pedidos_id PRIMARY KEY(id),
    CONSTRAINT fk_itens_pedido_pedido_id 
        FOREIGN KEY (pedido_id)
        REFERENCES pedidos(id)
);

ALTER TABLE itens_pedido ADD subtotal AS (quantidade * preco_unitario) PERSISTED;

SELECT is_computed, is_persisted FROM sys.computed_columns WHERE object_id = OBJECT_ID('itens_pedido');
```
---

## Desafio

```sql
CREATE TABLE fornecedores (
    id INT IDENTITY(1, 1) NOT NULL,
    nome VARCHAR(255) NOT NULL,
    cnpj CHAR(14) NOT NULL UNIQUE, 
    email VARCHAR(200) NOT NULL,
    ativo BIT DEFAULT 1,
    CONSTRAINT pk_fornecedores_id PRIMARY KEY(id)
);
CREATE TABLE produtos (
    id INT IDENTITY(1, 1) NOT NULL,
    nome VARCHAR(255) NOT NULL,
    fornecedor_id INT NOT NULL,
    CONSTRAINT pk_produtos_id PRIMARY KEY(id),
    CONSTRAINT fk_produtos_fornecedor_id 
        FOREIGN KEY(fornecedor_id)
        REFERENCES fornecedores(id)
); 

CREATE TABLE clientes(
    id INT IDENTITY(1, 1) NOT NULL,
    CONSTRAINT pk_customers_id PRIMARY KEY(id)
);

ALTER TABLE clientes ADD estado CHAR(2) NULL;
ALTER TABLE clientes ADD cep CHAR(8) NULL;
ALTER TABLE clientes ADD complemento VARCHAR(100) NULL;
ALTER TABLE clientes ADD nome VARCHAR(100) NOT NULL;
ALTER TABLE clientes ADD sobre_nome VARCHAR(100) NOT NULL;
ALTER TABLE clientes ADD email VARCHAR(100) NOT NULL;

ALTER TABLE clientes 
ADD CONSTRAINT ck_clientes_estado
CHECK (len(estado) = 2);

CREATE INDEX idx_clientes_email ON clientes(email);

CREATE TABLE enderecos (
    id INT IDENTITY(1, 1) NOT NULL,
    cliente_id INT NOT NULL,
    CONSTRAINT pk_enderecos_id PRIMARY KEY(id),
    CONSTRAINT fk_enderecos_cliente_id 
        FOREIGN KEY (cliente_id)
        REFERENCES clientes(id)
);

CREATE TABLE historico_precos(
    id INT IDENTITY(1, 1) NOT NULL,       
    produto_id INT NOT NULL,
    preco_ant DECIMAL(12, 4) NOT NULL,
    preco_nov DECIMAL(12, 4) NOT NULL,
    data_alteracao DATE NOT NULL,
    CONSTRAINT pk_historico_precos_id PRIMARY KEY(id),
    CONSTRAINT fk_historico_precos_produtos_id 
        FOREIGN KEY (produto_id)
        REFERENCES produtos(id)
);
```
