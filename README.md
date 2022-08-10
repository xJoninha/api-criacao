# Criando api e modelando CRUD

Repositório criado para colocar em 
pratica a criação de api junto a modelagem do CRUD

## Criando pasta do projeto usando ``express-generator``

1- No terminal rodar o comando ``express nome-do-servidor --view=ejs``,
repare que junto ao comando de criação já definimos nossa "Viewengine".

2- Ainda no terminal entrar na pasta que foi criada ``cd nome-do-servidor``,
e rodar o comando ``npm install``, para instalar as dependências.

3- Dentro da pasta rodar o comando ``npm install nodemon --save -D``,
pois usaremos adiante.

## Editando arquivo ``package.json`` 

```json
...
"scripts": {
"start": "node ./bin/www",
"dev": "nodemon ./bin/www"
},
...
```

## Instalando pacotes: ``sequelize`` | ``sequelize-cli`` | ``mysql2``

Será nossa ferramenta para podermos nos conectar
e fazer tratamento de todos os dados armazenados
com o banco de dados **MySQL**.

1- No terminal rodar o comando ``npm install sequelize sequelize-cli --save -g``

2- Instalar pacote que permite o node conectar com o **MySQL**,
ainda no terminar rodar o comando ``npm install mysql2 --save``

## Configurando arquivos para conectarmos ao *BD*

1- Na raiz do projeto "``nome-do-servidor``",
criar arquivo ``.sequelizerc``

2- Dentro do arquivo vamos informar ao **sequelize** o caminho das pastas
que irão conter os arquivos necessários para poder funcionar corretamente.
```js
const path = require('path')

module.exports = {
    config: path.resolve('./database/config', 'config.js'),
    'models-path': path.resolve('./database/models'),
    'seeders-path': path.resolve('./database/seeders'),
    'migrations-path': path.resolve('./database/migrations')
}
```

3- No terminal rodar o comando ``sequelize init``

4- Configurando acesso ao nosso BD.
Editar arquivo ``config.js`` que esta dentro da pasta que o ``sequelize init`` criou
``database/config/config.js``
```js
module.exports = {
  "development": {
    "username": "usuario",        // Usuario, padrão "root"
    "password": "senha",          // Senha, padrão vazio ""
    "database": "nome_do_banco",  // Nome do seu BD
    "host": "127.0.0.1",          // Onde o BD vai rodar, padrão "localhost"
    "dialect": "mysql"            // Dialeto do BD
  }
}
```

## Criando *Models* e relacionando tabelas

Dentro da pasta ``database/models`` criar 3 arquivos com os nomes
``Genre.js``, ``Actor.js``, ``Movie.js``
**É importante saber que no mundo da programação em geral nos utilizamos
sempre o inglês como base em nome de arquivos, variáveis, funções, etc.
então já é bom ir se acostumando com isso**

1- Configurar os arquivos criados.
**Genre.js**
```js
module.exports = function(sequelize, dataTypes) {

    let alias = "Genre";

    let cols = {
        id: {
            type: dataTypes.INTEGER,
            primaryKey: true,
            autoIncrement: true
        },
        nome: {
            type: dataTypes.STRING
        }
    }

    let config = {
        tableName: "genres",
        timestamps: false
    }

    let Genre = sequelize.define(alias, cols, config);

    return Genre
}
```

**Actor.js**
```js
module.exports = function(sequelize, dataTypes) {

    let alias = "Actor";

    let cols = {
        id: {
            type: dataTypes.INTEGER,
            primaryKey: true,
            autoIncrement: true
        },
        first_name: {
            type: dataTypes.STRING
        },
        last_name: {
            type: dataTypes.STRING
        }
    }

    let config = {
        tableName: "actors",
        timestamps: false
    }

    let Actor = sequelize.define(alias, cols, config);

    return Actor
}
```

**Movie.js**
```js
module.exports = function(sequelize, dataTypes) {

    let alias = "Movie";

    let cols = {
        id: {
            type: dataTypes.INTEGER,
            primaryKey: true,
            autoIncrement: true
        },
        title: {
            type: dataTypes.STRING
        },
        awards: {
            type: dataTypes.INTEGER
        },
        rating: {
            type: dataTypes.DOUBLE
        },
        length: {
            type: dataTypes.INTEGER
        },
        genre_id: {
            type: dataTypes.INTEGER
        },
        release_date: {
            type: dataTypes.DATE
        }
    }

    let config = {
        tableName: "movies",
        timestamps: false
    }

    let Movie = sequelize.define(alias, cols, config);

    return Movie
}
```

2- Dizer para o sequelize como essas 3 tabelas se relacionam.

### **Genre.js**
Logo após a definição da variável *Genre*, chamamos o modelo ``Genre`` e usamos o **associate** do sequelize, que irá retornar uma **função** que recebe um parâmetro chamado ``models``, e vamos dizer que o ``Genre`` pode estar associado a muitos filmes, então iremos usar a associação ``hasMany`` "um *Genre* tem muitos *Movie*".
Nesta associação iremos ter dois parâmetros o primeiro vamos pegar o model que vai ser usado nessa relação, no caso ``Movie`` e no segundo parâmetro iremos mandar um objeto falando o nome da associação com o **as** e sua chave estrangeira(foreignKey) **foreignKey**.
```js
Genre.associate = function(models) {
	Genre.hasMany(models.Movie, {
		as: "movies",
		foreignKey: "genre_id"
	})
}
```

### **Actor.js**
Seguindo o mesmo exemplo feito no model ``Genre``, iremos fazer algumas mudanças
o tipo de relação vai ser ``belongsToMany`` "um *Actor* pertence a muitos *Movie*".
Deixamos o **as** como o anterior, mas como *Actor* é uma relação de muitos para muitos(N-M), precisamos de uma tabela intermediária usando a palavra **through** demos o nome de *actor_movie* que terá como chave estrangeira *actor_id* e também uma outra chave estrangeira,
a palavra usada para usar uma segunda chave estrangeira é **otherKey** *movie_id*, e por ultimo como é uma nova tabela temos que informar que ela não tem *timestamps*.
```js
Actor.associate = function(models) {
	Actor.belongsToMany(models.Movie, {
		as: "movies",
        through: "actor_movie",
		foreignKey: "actor_id",
        otherKey: "movie_id",
        timestamps: false
	})
}
```

### **Movie.js**
Seguindo os mesmos exemplos dos models anteriores esta tabela terá *Duas* associações o primeiro vai ser de **Movie** para **Genre**, ``belongsTo`` "um *Movie* pertence a um *Genre*".
O **as** será *genre*, e a chave estrangeira que vai ser usada para trazer essa informação *genre_id*.

A Segunda associação é de **Movie** para **Actor**, ``belongsToMany`` "um *Movie* pertence a muitos *Actor*".
O **as** será *actors*, e como é uma associação N-M precisamos de uma tabela intermediária usaremos a mesma que definimos no model *Actor*, **actor_movie** ela terá duas informações a do *Movie* e *Actor*, iremos apenas mudar as chaves estrangeiras a **foreignKey** vai ser *movie_id* e a **otherKey** será *actor_id* mantenha o *timestamps* como false para a tabela intermediária.

```js
Movie.associate = function(models) {
	Movie.belongsTo(models.Movie, {
		as: "genre",
		foreignKey: "genre_id"
	});

    Movie.belongsToMany(models.Actor, {
        as: "actors",
        through: "actor_movie",
        foreignKey: "movie_id",
        otherKey: "actor_id",
        timestamps: false
    })
}
```