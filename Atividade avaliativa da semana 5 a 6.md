## Resolução das Consultas MongoDB (Questões 1 a 9) - Semana 5 e 6

### 1. Mostrar os três primeiros documentos da coleção `doacao`.

| Comando | Explicação |
| :--- | :--- |
| `db.doacao.find()` | Inicia a consulta na coleção `doacao`. |
| `.limit(3)` | Limita o conjunto de resultados a retornar apenas os **3 primeiros** documentos. |

**Comando Completo**
```js
db.doacao.find().limit(3)
```
-----

### 2. Mostrar os doadores que moram nos estados do **Ceará, Alagoas ou São Paulo**, e que tenham tipo sanguíneo **A ou O**, listando código, nome, cidade, estado e tipo sanguíneo.

| Comando | Explicação |
| :--- | :--- |
| `db.doador.find(` | Inicia a consulta. |
| `   { $and: [ ` | Abre o operador **`$and`** (lógico E) para combinar as duas grandes condições (estado E tipo sanguíneo). |
| `     { "enderecoDoador.dscUFDoador": { $in: ["CE", "AL", "SP"] } }, ` | **Filtro de Estado:** Usa **`$in`** (relacional) no campo aninhado para selecionar documentos onde o estado é CE **OU** AL **OU** SP. |
| `     { "indTipoSangDoador": { $in: ["A", "O"] } } ` | **Filtro de Tipo:** Usa **`$in`** para selecionar documentos onde o tipo sanguíneo é A **OU** O. |
| `   ] }, ` | Fecha o filtro. |
| `   { idDoador: 1, nomDoador: 1, ` | **Projeção:** Inclui o código e nome (`1`). |
| `     "enderecoDoador.dscCidadeDoador": 1, ` | Inclui a cidade, usando **ponto-notação** para campos dentro do array de endereço. |
| `     "enderecoDoador.dscUFDoador": 1, indTipoSangDoador: 1, _id: 0 ` | Inclui o estado e o tipo sanguíneo, e **exclui** o campo `_id` (`0`). |

**Comando Completo**
```js
db.doador.find(
{
$and: [
{ "enderecoDoador.dscUFDoador": { $in: ["CE", "AL", "SP"] } },
{ "indTipoSangDoador": { $in: ["A", "O"] } }
]
},
{
idDoador: 1,
nomDoador: 1,
"enderecoDoador.dscCidadeDoador": 1,
"enderecoDoador.dscUFDoador": 1,
indTipoSangDoador: 1,
_id: 0
}
)
```


---

### 3. Mostrar os doadores dos tipos sanguíneos **A, B ou O** e que sejam do estado de **São Paulo**. Listar código, nome, cidade, tipo sanguíneo e Rh.

| Comando | Explicação |
| :--- | :--- |
| `db.doador.find({` | Inicia a consulta. |
| `  "indTipoSangDoador": { $in: ["A", "B", "O"] },` | Filtra por tipo sanguíneo A **OU** B **OU** O. |
| `  "enderecoDoador.dscUFDoador": "SP"` | Filtra por estado igual a 'SP'. O **`$and` é implícito** entre esta condição e a anterior. |
| `}, {` | Abre a projeção. |
| `  idDoador: 1, nomDoador: 1,` | Inclui código e nome. |
| `  "enderecoDoador.dscCidadeDoador": 1,` | Inclui a cidade. |
| `  indTipoSangDoador: 1, indFatoRhDoador: 1, _id: 0` | Inclui tipo sanguíneo e Rh, e exclui `_id`. |


**Comando Completo**
```js
db.doador.find(
  {
    "indTipoSangDoador": { $in: ["A", "B", "O"] },
    "enderecoDoador.dscUFDoador": "SP"
  },
  {
    idDoador: 1,
    nomDoador: 1,
    "enderecoDoador.dscCidadeDoador": 1,
    indTipoSangDoador: 1,
    indFatoRhDoador: 1,
    _id: 0
  }
)
```
---

### 4. Listar código, quantidade de sangue doada e data das doações feitas **entre janeiro e julho de 2021** e **acima de 690 ml**.

| Comando | Explicação |
| :--- | :--- |
| `db.doacao.find({` | Inicia a consulta. |
| `  datDoacao: {` | Abre o filtro para o campo de data. |
| `    $gte: new Date("2021-01-01T00:00:00Z"),` | Filtra datas **maiores ou iguais** (**`$gte`**) ao início de janeiro de 2021. |
| `    $lte: new Date("2021-07-31T23:59:59Z")` | Filtra datas **menores ou iguais** (**`$lte`**) ao final de julho de 2021. |
| `  },` | Fecha o filtro de data. |
| `  qtdSangueDoada: { $gt: 690 }` | Filtra a quantidade de sangue **maior que** (**`$gt`**) 690 ml. |
| `}, {` | Abre a projeção. |
| `  idDoacao: 1, qtdSangueDoada: 1, datDoacao: 1, _id: 0` | Inclui os campos solicitados e exclui `_id`. |


**Comando Completo**
```js
db.doacao.find(
  {
    datDoacao: {
      $gte: new Date("2021-01-01T00:00:00Z"), 
      $lte: new Date("2021-07-31T23:59:59Z")
    },
    qtdSangueDoada: { $gt: 690 }
  },
  {
    idDoacao: 1,
    qtdSangueDoada: 1,
    datDoacao: 1,
    _id: 0
  }
)
```


---

### 5. Listar `_id`, código, nome, email, cidade e lanche do doador que tenha tipo sanguíneo **A, B ou AB**, com **Rh negativo**.

| Comando | Explicação |
| :--- | :--- |
| `db.doador.find({` | Inicia a consulta. |
| `  "indTipoSangDoador": { $in: ["A", "B", "AB"] },` | Filtra por tipo sanguíneo A **OU** B **OU** AB. |
| `  "indFatoRhDoador": "-"` | Filtra por Rh negativo (igualdade). |
| `}, {` | Abre a projeção. |
| `  idDoador: 1, nomDoador: 1, dscEmailDoador: 1,` | Inclui código, nome e email. |
| `  "enderecoDoador.dscCidadeDoador": 1,` | Inclui a cidade. |
| `  dscLancheDoador: 1` | Inclui o array completo de lanches. O campo `_id` é incluído por padrão. |

**Comando Completo**
```js
db.doador.find(
  {
    "indTipoSangDoador": { $in: ["A", "B", "AB"] },
    "indFatoRhDoador": "-"
  },
  {
    idDoador: 1,
    nomDoador: 1,
    dscEmailDoador: 1,
    "enderecoDoador.dscCidadeDoador": 1,
    dscLancheDoador: 1
  }
)
```
## 6. Pesquisar se é possível selecionar o segundo lanche do doador que teve mais de dois lanches, utilizando o comando `find`.

**Resposta**:
**Não é possível** com o `db.collection.find()` puro, pois ele não suporta a lógica de filtragem complexa de tamanho (`$gt` em `$size`) nem a projeção de elementos específicos do array por índice (`$arrayElemAt`). É necessário o **Aggregation Framework** (`.aggregate()`). |

| Comando | Explicação |
| :--- | :--- |
| `db.doador.aggregate([` | Inicia o pipeline de agregação. |
| `  { $addFields: { lanchesValidos: { $ifNull: ["$dscLancheDoador", []] } } },` | **Estágio 1 (`$addFields`):** Cria o campo temporário `lanchesValidos`. O operador **`$ifNull`** garante que, se `dscLancheDoador` for nulo ou ausente, ele seja tratado como um **array vazio `[]`**, prevenindo o erro de `$size`. |
| `  { $match: { $expr: { $gt: [ { $size: "$lanchesValidos" }, 2 ] } } },` | **Estágio 2 (`$match`):** Filtra doadores. O **`$expr`** permite a comparação (`$gt`) entre o **tamanho** (`$size`) do array e o número `2`. |
| `  { $project: {` | **Estágio 3 (`$project`):** Remodela o documento final. |
| `    segundoLanche: { $arrayElemAt: ["$dscLancheDoador", 1] },` | Usa o operador **`$arrayElemAt`** para criar o campo `segundoLanche`, extraindo o elemento no **índice 1** (o segundo item). |
| `    idDoador: 1, nomDoador: 1, dscEmailDoador: 1, ...` | Inclui outros campos. |

**Comando Completo**
```js
db.doador.aggregate([
    {
        // 1. Cria um campo temporário, garantindo que dscLancheDoador seja um array [] se for nulo/ausente
        $addFields: {
            lanchesValidos: { $ifNull: ["$dscLancheDoador", []] }
        }
    },
    { 
        // 2. Filtra documentos onde o tamanho do array lanchesValidos é maior que 2
        $match: { $expr: { $gt: [ { $size: "$lanchesValidos" }, 2 ] } } 
    },
    {
        // 3. Projeta os campos desejados, incluindo o segundo lanche
        $project: {
            _id: 0,
            idDoador: 1,
            nomDoador: 1,
            dscEmailDoador: 1,
            // Projeta a cidade e o estado do primeiro (e único) elemento do array enderecoDoador
            cidade: { $arrayElemAt: ["$enderecoDoador.dscCidadeDoador", 0] },
            estado: { $arrayElemAt: ["$enderecoDoador.dscUFDoador", 0] },
            // Projeta o elemento de índice 1 (o segundo item) de dscLancheDoador
            segundoLanche: { $arrayElemAt: ["$dscLancheDoador", 1] }
        }
    }
])
```

---

### 7. Mostrar as doações efetuadas **entre os anos de 2021 e 2022**, cuja quantidade de sangue doada tenha sido **maior que 690 ml**. Listar o código da doação, a data da doação e a quantidade de sangue doada.

| Comando | Explicação |
| :--- | :--- |
| `db.doacao.find({` | Inicia a consulta. |
| `  datDoacao: {` | Abre o filtro de data. |
| `    $gte: new Date("2021-01-01T00:00:00Z"),` | Filtra datas **maiores ou iguais** ao início de 2021. |
| `    $lte: new Date("2022-12-31T23:59:59Z")` | Filtra datas **menores ou iguais** ao final de 2022. |
| `  },` | Fecha o filtro de data. |
| `  qtdSangueDoada: { $gt: 690 }` | Filtra a quantidade de sangue **maior que** 690 ml. |
| `}, {` | Abre a projeção. |
| `  idDoacao: 1, datDoacao: 1, qtdSangueDoada: 1, _id: 0` | Inclui os campos solicitados e exclui `_id`. |

**Comando Completo**
```js
db.doacao.find(
  {
    datDoacao: {
      $gte: new Date("2021-01-01T00:00:00Z"),
      $lte: new Date("2022-12-31T23:59:59Z")
    },
    qtdSangueDoada: { $gt: 690 }
  },
  {
    idDoacao: 1,
    datDoacao: 1,
    qtdSangueDoada: 1,
    _id: 0
  }
)
```

---

### 8. Listar código, nome, endereço (bairro, cidade e estado) e lanche dos doadores do **sexo feminino**, **Rh negativo**, dos estados de **SP, ES ou CE**, e que **não tenha lanche**.

| Comando | Explicação |
| :--- | :--- |
| `db.doador.find({` | Inicia a consulta. |
| `  indSexoDoador: " F",` | Filtra por sexo feminino. |
| `  indFatoRhDoador: "-",` | Filtra por Rh negativo. |
| `  "enderecoDoador.dscUFDoador": { $in: ["SP", "ES", "CE"] },` | Filtra por estado SP **OU** ES **OU** CE. |
| `  dscLancheDoador: { $exists: false }` | Filtro crucial: usa o operador **`$exists`** com valor `false` para selecionar documentos onde o campo **não existe** (ou seja, doadores que não têm lanche). |
| `}, {` | Abre a projeção. |
| `  idDoador: 1, nomDoador: 1,` | Inclui código e nome. |
| `  "enderecoDoador.dscBairroDoador": 1,` | Inclui o bairro usando ponto-notação. |
| `  "enderecoDoador.dscCidadeDoador": 1,` | Inclui a cidade. |
| `  "enderecoDoador.dscUFDoador": 1, dscLancheDoador: 1, _id: 0` | Inclui o estado e o lanche, e exclui `_id`. |

**Comando Completo**
```js
db.doador.find(
  {
    indSexoDoador: " F",
    indFatoRhDoador: "-",
    "enderecoDoador.dscUFDoador": { $in: ["SP", "ES", "CE"] },
    dscLancheDoador: { $exists: false }
  },
  {
    idDoador: 1,
    nomDoador: 1,
    "enderecoDoador.dscBairroDoador": 1,
    "enderecoDoador.dscCidadeDoador": 1,
    "enderecoDoador.dscUFDoador": 1,
    dscLancheDoador: 1,
    _id: 0
  }
)
```

---

### 9. Listar código, nome e endereço completo de doadores do **sexo feminino** que moram em **CE, SP, ES ou AL** e que tenha **“mar”** no nome (**insensível a maiúsculas/minúsculas**).

| Comando | Explicação |
| :--- | :--- |
| `db.doador.find({` | Inicia a consulta. |
| `  indSexoDoador: " F",` | Filtra por sexo feminino. |
| `  "enderecoDoador.dscUFDoador": { $in: ["CE", "SP", "ES", "AL"] },` | Filtra por estado CE **OU** SP **OU** ES **OU** AL. |
| `  nomDoador: { $regex: "mar", $options: "i" }` | **Filtro de Nome:** Usa **`$regex`** (expressão regular) para buscar a substring "mar". O modificador **`$options: "i"`** torna a busca *case-insensitive* (ignora maiúsculas/minúsculas). |
| `}, {` | Abre a projeção. |
| `  idDoador: 1, nomDoador: 1,` | Inclui código e nome. |
| `  enderecoDoador: 1, _id: 0` | Inclui o **array de subdocumentos** `enderecoDoador` completo e exclui `_id`. |

**Comando Completo**
```js
db.doador.find(
  {
    indSexoDoador: " F",
    "enderecoDoador.dscUFDoador": { $in: ["CE", "SP", "ES", "AL"] },
    nomDoador: { $regex: "mar", $options: "i" }
  },
  {
    idDoador: 1,
    nomDoador: 1,
    enderecoDoador: 1,
    _id: 0
  }
)
```
