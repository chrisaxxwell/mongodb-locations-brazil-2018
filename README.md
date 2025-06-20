# Localizações do Brazil para MongoDB

Lista de localizações do brasil em formato 2dsphare para mongoDB por @chrisaxxwell

## Adicione ao seu banco de dados mongodb assim:

- mongoimport --db seuBanco --collection suaColecao --file mongodb-locations-brazil-2018.json --jsonArray

## Informações

- Meu arquivo contém **1.108.505** (Um milhão, cento e oito mil, quinhentos e cinco) localizações do brasil.
- Uma estimativa dos correios aponta que existem aproximadamente **1.450.000** (Um milhão, quatrocentos e cinquenta) localizações.
- Para corrigir essa pequena diferença recomendo que defina um raio de de 1 a 5km pra achar por geolicalização mais proximos.

## Recomendações

- Crie 2 indices. Um para **2dsphere**, e outro **indice para slug**

- await suaCollecaoDasLocalizacoes.createIndex({ location: "2dsphere" });
- await suaCollecaoDasLocalizacoes.createIndex({ slug: 1});

## Formato dos documentos

```javascript
{
  "_id": {
    "$oid": "6854528599cebc2847759e06"
  },
  "postal_code": "01001000", //Cep
  "type": "Praça", //Tipo
  "street_name": "da Sé", //Nome da rua
  "street": "Praça da Sé", //Tipo da rua mais nome
  "district": "Sé", //Bairro
  "city": "São Paulo", //Cidade
  "state": "SP", //Estado em UFs
  "complement": "lado ímpar", //Complemento
  "special_users": "", //Cep expecifico de empresa ou orgao: eg. Prefeitura, Bancos
  "ibge": 3550308, //IBGE
  "city_area": 1521.11, //Área territorial da cidade
  "area_code": 11, //DDD
  "active": true, //Cep ativo?
  //Coordenadas para usar com 2dshpare https://www.mongodb.com/docs/manual/core/indexes/index-types/geospatial/2dsphere/
  "location": {
    "type": "Point",
    "coordinates": [
      -46.6342179,
      -23.5502784
    ]
  },
  //Dados brutos/limpos para facilitar pesquisa
  "raw_street_name": "da se",
  "raw_street": "praca da se",
  "raw_district": "se",
  "raw_city": "sao paulo",
  "raw_special_users": "",
  "slug": "praca da se se sao paulo" //Para pesquisa rapida [rua + bairro + cidade] //Rapido acesso
}
```

## Como buscar

```javascript
//Para encontrar por CEP
const query = { postal_code: { $eq: "01001000" } };
const result = await collection.findOne(query);

//Para encontrar por rua, bairro e cidade (recomendo [rua + bairro + cidade]) para facilitar;
var address = "Praça da sé Sé São Paulo"
  .normalize("NFD")
  .replace(/[\u0300-\u036f]/g, "")
  .toLowerCase();

const query = {
      $or: [
            { slug: { $regex: address } },
            { raw_special_users: { $regex: address } }
      ],
};

const result =await collection .find(query, signal: signal }).limit(20).toArray();

//Por 2dsphare
var query = {
  location: {
    $near: {
      $geometry: {
        type: "Point",
        coordinates: [-40.42343, -30.555], //Deve obrigatoriamente ser [longitude, latitude]
      },
      $maxDistance: 1000, //1km, mas pode ser 2, 3, 4...
    },
  },
};
const result = await collection.findOne(query);
```

## Autores

- [@chrisaxxwell](https://www.github.com/chrisaxxwell)
- [@paulo-henrique](https://www.instagram.com/euphbeta)
