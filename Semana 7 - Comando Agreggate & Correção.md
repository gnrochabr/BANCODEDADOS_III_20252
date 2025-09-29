# Função Aggregate

A função **`aggregate()`** permite processar dados de forma avançada em MongoDB. Diferente do `find()`, que apenas retorna documentos, `aggregate` possibilita **filtrar, agrupar, ordenar, unir coleções e calcular estatísticas** em múltiplos estágios.

A sintaxe básica é:

```javascript
db.colecao.aggregate([
  { estágio1 },
  { estágio2 },
  ...
])
```

Cada **estágio** transforma os documentos e pode usar operadores específicos, como:

* `$match`: filtrar documentos.
* `$group`: agrupar e calcular somas, médias, máximos e mínimos.
* `$sort`: ordenar os resultados.
* `$project`: selecionar ou renomear campos.
* `$lookup`: unir coleções.
* `$unwind`: “desdobrar” arrays para processar cada elemento separadamente.

---

## Exemplos

### 1 – Contar doadores por tipo sanguíneo

```javascript
db.doador.aggregate([
  {
    $group: {
      _id: "$indTipoSangDoador", // Agrupa os documentos por tipo sanguíneo
      totalDoadores: { $sum: 1 } // Conta 1 para cada documento dentro de cada grupo
    }
  }
])
```

**Explicação detalhada:**

* `$group` é usado para **agrupar documentos** com base em um campo específico.
* `_id` define a chave do agrupamento (neste caso, `indTipoSangDoador`).
* `$sum` soma valores dentro do grupo; usando 1, estamos apenas **contando os documentos**.

---

### 2 – Listar doadores do sexo feminino e contar

```javascript
db.doador.aggregate([
  { 
    $match: { indSexoDoador: " F" } // Filtra apenas os doadores do sexo feminino
  },
  { 
    $count: "totalFeminino" // Conta quantos documentos passaram pelo filtro
  }
])
```

**Explicação detalhada:**

* `$match` é semelhante ao `find()`; **filtra os documentos** de acordo com condições.
* `$count` retorna **quantos documentos** atenderam à condição do `$match`.

---

### 3 – Média de doações por doador

```javascript
db.doacao.aggregate([
  {
    $group: {
      _id: "$idDoador",           // Agrupa por doador
      mediaDoacoes: { $avg: "$qtdSangueDoada" } // Calcula a média das doações
    }
  }
])
```

**Explicação detalhada:**

* `$avg` calcula a **média de valores** de um campo dentro de cada grupo.
* Útil para **analisar comportamento por doador** ou por qualquer outra categoria.

---

### 4 – Maior quantidade de doações por doador

```javascript
db.doacao.aggregate([
  {
    $group: {
      _id: "$idDoador",             // Agrupa por doador
      maxDoacoes: { $max: "$qtdSangueDoada" } // Encontra a maior doação
    }
  }
])
```

**Explicação detalhada:**

* `$max` retorna **o maior valor** dentro de cada grupo.
* Outros operadores semelhantes: `$min` (menor), `$sum` (soma), `$avg` (média).

---

### 5 – Total de doações por estado

```javascript
db.doador.aggregate([
  { 
    $unwind: "$enderecoDoador" // "Desdobra" cada item do array enderecoDoador em documentos separados
  },
  {
    $group: {
      _id: "$enderecoDoador.dscUFDoador", // Agrupa por estado (UF)
      totalDoadores: { $sum: 1 }  // Conta quantos doadores existem por estado
    }
  }
])
```

**Explicação detalhada:**

* `$unwind` transforma **cada elemento de um array** em um documento individual.
* Isso permite agrupar por campos que estão **dentro de arrays**.

---

### 6 – Doadores com mais de 5 doações

```javascript
db.doacao.aggregate([
  {
    $group: {
      _id: "$idDoador",                 // Agrupa por doador
      totalDoacoes: { $sum: "$qtdSangueDoada" } // Soma todas as doações de cada doador
    }
  },
  { 
    $match: { totalDoacoes: { $gt: 5 } } // Filtra doadores com mais de 5 doações
  }
])
```

**Explicação detalhada:**

* Primeiro agrupamos e somamos todas as doações.
* Depois usamos `$match` para **filtrar com base em cálculos agregados**, mostrando apenas os doadores que atenderem ao critério.

---

## 7 – Ordenar doadores por quantidade total de doações e filtrar os top 5

```javascript
db.doacao.aggregate([
  {
    $group: {
      _id: "$idDoador",                 // Agrupa por doador
      totalDoacoes: { $sum: "$qtdSangueDoada" } // Soma total das doações
    }
  },
  { 
    $sort: { totalDoacoes: -1 } // Ordena do maior para o menor número de doações
  },
  {
    $limit: 5 // Limita o resultado aos 5 doadores com mais doações
  }
])
```

**Explicação detalhada:**

* Primeiro agrupamos as doações por doador.
* `$sort` organiza do maior para o menor número de doações.
* `$limit` **filtra apenas os top 5**, mostrando como combinar estágios de agregação para análises mais precisas.

---

## 8 – Listar doadores com suas doações e calcular a soma total de cada um

```javascript
db.doador.aggregate([
  {
    $lookup: {
      from: "doacao",           // Coleção a ser unida
      localField: "idDoador",   // Campo na coleção principal
      foreignField: "idDoador", // Campo na coleção secundária
      as: "minhasDoacoes"       // Nome do array que conterá as doações
    }
  },
  {
    $addFields: {
      totalDoacoes: { $sum: "$minhasDoacoes.quantidadeDoacao" } // Soma total das doações de cada doador
    }
  },
  {
    $project: {
      nomDoador: 1,
      totalDoacoes: 1,
      _id: 0
    }
  }
])
```

**Explicação detalhada:**

* `$lookup` une doadores e doações.
* `$addFields` adiciona **um novo campo calculado** com a soma das doações do doador.
* `$project` seleciona apenas campos relevantes para a visualização.
* Combinação de estágios permite análises mais complexas **sem alterar os dados originais**.

---

## 9 – Contar doadores por sexo, tipo sanguíneo e estado, filtrando apenas combinações com mais de 2 doadores

```javascript
db.doador.aggregate([
  { 
    $unwind: "$enderecoDoador" // Desdobra arrays de endereço
  },
  {
    $group: {
      _id: { 
        sexo: "$indSexoDoador", 
        tipo: "$indTipoSangDoador",
        estado: "$enderecoDoador.dscUFDoador:"
      }, // Agrupa por múltiplos campos
      total: { $sum: 1 } // Conta quantos doadores existem em cada combinação
    }
  },
  {
    $match: { total: { $gt: 2 } } // Filtra combinações com mais de 2 doadores
  },
  {
    $sort: { total: -1 } // Ordena do maior para o menor número de doadores
  }
])
```

**Explicação detalhada:**

* `$unwind` permite trabalhar com arrays (endereços) como se fossem documentos separados.
* `$group` agrupa por **três campos ao mesmo tempo**, possibilitando análises multidimensionais.
* `$match` após o `$group` **filtra com base em resultados agregados**, não apenas nos dados originais.
* `$sort` organiza os resultados para fácil interpretação.

---

## 10 – Selecionar doadores com suas doações recentes, apenas e-mails e nomes, ordenados pela última doação

```javascript
db.doador.aggregate([
  {
    $lookup: {
      from: "doacao", 
      localField: "idDoador", 
      foreignField: "idDoador", 
      as: "minhasDoacoes"
    }
  },
  {
    $addFields: {
      ultimaDoacao: { $max: "$minhasDoacoes.dataDoacao" } // Calcula a data da última doação
    }
  },
  {
    $project: {
      _id: 0,
      nomDoador: 1,
      dscEmailDoador: 1,
      ultimaDoacao: 1
    }
  },
  {
    $sort: { ultimaDoacao: -1 } // Ordena do mais recente para o mais antigo
  }
])
```

**Explicação detalhada:**

* Combina `$lookup` + `$addFields` para **incluir cálculo baseado em arrays** (data da última doação).
* `$project` seleciona apenas informações úteis (nome, e-mail e data da última doação).
* `$sort` ordena os doadores **pela recência da última doação**, útil para campanhas de doação ou acompanhamento.
* Demonstra como combinar agregações, cálculos e projeções para análises **complexas e práticas**.

---

## Exercícios da Semana 7
1) O comando de agregação abaixo gera uma nova coleção, verificando primeiro se o
doador tem lanche, necessário quando se tem o array na mesma coleção.
db.doador.aggregate([
 {
 $addFields: {
 qtdLanche: { $size: { $ifNull: ["$dscLancheDoador", []] } }
 }
 },
 {
 $out: "doadorLanche"
 }
])
Executar o comando e após gerar a coleção doadorLanche, fazer uma pesquisa para
mostrar os doadores que doaram sangue exatamente 3 (três vezes) e receberam
mais de 2 lanches.
Sugestão: Fazer uma junção da nova coleção criada com a coleção doacao.
2) Fazer uma consulta para listar os doadores que moram no estado de São Paulo ou
Ceará e que tenha doado sangue em 2022.
3) Mostrar o id, nome, email, cidade e estado dos doadores que doaram acima de 400
ml de sangue e que não tenha recebido nenhum lanche.
4) Listar o id, nome, quantidade de sangue doada, tipo e rh sanguíneo dos doadores (5
primeiros documentos) que tenham doado sangue entre 2020 e 2022, acima de 600 ml.
Ordenar em ordem de nome do doador.
5) Listar a quantidade de doadores por fator Rh.
6) Listar o id, nome, email, bairro,cidade e estado dos doadores que não doaram
sangue dos estados de Ceará, São Paulo, Espírito Santo e Rio de Janeiro.

---

# Resumo das principais funções do `aggregate` no MongoDB que deverão ser usadas na sua resolução

### 1. `$group`

* **Função:** Agrupa documentos de acordo com um ou mais campos e permite calcular somas, médias, contagens ou valores máximos/mínimos.
* **Exemplo de uso:** Contar a quantidade de doadores por tipo sanguíneo, calcular a soma total de doações de cada doador.
* **Aplicação nas questões:**

  * **Questão 5:** Para listar a quantidade de doadores por fator Rh (`_id: "$indFatoRhDoador"` e `total: { $sum: 1 }`).
  * **Questão 1 (junção e filtro):** Agrupar doações por doador para verificar quem doou exatamente 3 vezes.

---

### 2. `$lookup`

* **Função:** Realiza uma junção entre duas coleções, semelhante a `JOIN` em SQL. Cria um array com os documentos relacionados.
* **Exemplo de uso:** Juntar doadores e suas doações, ou doadores e os lanches recebidos.
* **Aplicação nas questões:**

  * **Questão 1:** Unir a coleção `doadorLanche` com a coleção `doacao` para filtrar doadores com 3 doações e mais de 2 lanches.
  * **Questão 2:** Poderia ser usado se tivermos outra coleção com dados de doações por ano.
  * **Questão 10 dos exercícios anteriores:** Listar doadores com suas últimas doações.

---

### 3. `$addFields`

* **Função:** Adiciona novos campos ao documento ou modifica campos existentes, podendo incluir cálculos ou transformações.
* **Exemplo de uso:** Somar quantidades de doação, calcular a quantidade de lanches recebidos (`$size` de um array).
* **Aplicação nas questões:**

  * **Questão 1:** Criar campo `qtdLanche` usando `$size` para contar os elementos do array de lanches.

---

### 4. `$project`

* **Função:** Seleciona quais campos incluir ou excluir no resultado, podendo também criar campos calculados.
* **Exemplo de uso:** Mostrar apenas o nome, email e quantidade de doações, sem exibir o `_id`.
* **Aplicação nas questões:**

  * **Questão 3:** Selecionar id, nome, email, cidade e estado dos doadores com critérios de sangue e lanche.
  * **Questão 4:** Exibir campos específicos e ordenar os resultados.
  * **Questão 6:** Mostrar informações do doador, excluindo dados desnecessários.

---

### 5. `$match`

* **Função:** Filtra documentos de acordo com condições, semelhante ao `WHERE` em SQL.
* **Exemplo de uso:** Filtrar doadores que doaram sangue acima de uma determinada quantidade, ou moram em estados específicos.
* **Aplicação nas questões:**

  * **Questão 1:** Filtrar doadores com 3 doações e mais de 2 lanches.
  * **Questão 2:** Filtrar doadores de São Paulo ou Ceará que doaram em 2022.
  * **Questão 3:** Filtrar doadores com doações acima de 400 ml e sem lanches.
  * **Questão 4:** Filtrar doações entre 2020 e 2022 acima de 600 ml.
  * **Questão 6:** Filtrar doadores que **não** doaram sangue e estão em estados específicos.

---

### 6. `$sort`

* **Função:** Ordena os resultados de acordo com um ou mais campos, crescente (1) ou decrescente (-1).
* **Exemplo de uso:** Ordenar doadores pelo nome, quantidade de doações ou data da última doação.
* **Aplicação nas questões:**

  * **Questão 4:** Ordenar por `nomDoador`.
  * **Questão 10 dos exercícios anteriores:** Ordenar doadores pela última doação.

---

### 7. `$limit`

* **Função:** Limita a quantidade de documentos retornados.
* **Exemplo de uso:** Mostrar apenas os 5 primeiros doadores que atendem aos critérios.
* **Aplicação nas questões:**

  * **Questão 4:** Selecionar apenas os 5 primeiros doadores que doaram acima de 600 ml entre 2020 e 2022.

---

### 8. `$unwind`

* **Função:** Desmembra arrays em documentos individuais, permitindo trabalhar com cada elemento separadamente.
* **Exemplo de uso:** Separar cada endereço ou cada lanche do doador.
* **Aplicação nas questões:**

  * **Questão 1:** Contar corretamente a quantidade de lanches recebidos pelo doador.
  * **Questão 6:** Trabalhar com doadores que podem ter múltiplos endereços, filtrando por estado.

---

### 9. `$ifNull`

* **Função:** Substitui valores nulos ou inexistentes por um valor padrão, útil para evitar erros em arrays vazios.
* **Exemplo de uso:** Garantir que `$size` de um array não retorne erro quando o array não existe.
* **Aplicação nas questões:**

  * **Questão 1:** `$ifNull: ["$dscLancheDoador", []]` garante que contagem de lanches funcione mesmo se não houver nenhum lanche registrado.

---

### 10. `$size`

* **Função:** Conta quantos elementos existem em um array.
* **Exemplo de uso:** Calcular quantos lanches um doador recebeu.
* **Aplicação nas questões:**

  * **Questão 1:** Criar campo `qtdLanche` para posteriormente filtrar os doadores que receberam mais de 2 lanches.

---

### 11. `$max` (usado nos exercícios anteriores)

* **Função:** Retorna o valor máximo de um campo numérico ou de data dentro de um array.
* **Exemplo de uso:** Encontrar a data da última doação do doador.
* **Aplicação nas questões:**

  * Poderia ser usado na **Questão 2 ou 3** para verificar a data mais recente de doação de cada doador.

---

### Como essas funções ajudam a resolver as questões listadas:

1. **Criar `doadorLanche` e filtrar doadores com 3 doações e mais de 2 lanches**
   → `$addFields` + `$size` + `$ifNull` + `$out` + `$lookup` + `$match`

2. **Filtrar doadores de São Paulo ou Ceará que doaram em 2022**
   → `$match` com condições de estado e ano de doação

3. **Mostrar doadores que doaram acima de 400 ml e não receberam lanche**
   → `$lookup` (para lanches) + `$match` + `$project`

4. **Listar 5 doadores entre 2020 e 2022 que doaram acima de 600 ml e ordenar por nome**
   → `$match` + `$sort` + `$limit` + `$project`

5. **Contar doadores por fator Rh**
   → `$group` + `$sum`

6. **Listar doadores que não doaram sangue e são de estados específicos**
   → `$lookup` (para doações) + `$match` + `$project` + `$unwind` (se necessário)

---

## Bons estudos!
