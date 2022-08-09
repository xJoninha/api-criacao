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
"scripts": {
"start": "node ./bin/www",
"dev": "nodemon ./bin/www"
},
```
