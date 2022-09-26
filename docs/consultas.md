
###### [VOLVER AL INICIO](https://github.com/cell-framework-project/sps-elastic-search-exam)

#### II HACIENDO CONSULTAS:

- Tenemos como base este **mapeado optimizado** que hemos generado

| name                 | type    | 
|----------------------|---------|
| @timestamp           | date    |
| estado_consulta      | keyword |
| servicio             | keyword |
| administrador        | text    |
| consultas_realizadas | long    |

##### 1) : Conteo de registros con **estado_consulta** igual a error y/o consumo:

- Utilizamos **count**
- Utilizamos la clausula **terms** para varios valores

###### Request :

```json
GET log_consultas/_count
{
  "query": {
    "bool": {
      "must": [
        {
          "terms": {
            "estado_consulta": ["modificacion", "error"]
          }
        }
      ]
    }
  }
}
```
###### Response :

```json
{
  "count": 78,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  }
}
```

#### Registros : 78


##### 2) : Conteo de registros con **administrador**  **Juan Lara**

- Utilizamos **count**
- Siendo el nombre considerado como dato de texto, decidimos utilizar la clausula **match_phrase**

###### Request :

```json
GET log_consultas/_count
{
  "query": {
    "match_phrase": {
      "administrador": "Juan Lara"
    }
  }
}
```

###### Response :

```json
{
  "count": 98,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  }
}
```

#### Registros : 98


##### 3) : Conteo de registros con **estado_consulta** igual a **informativo** y **servicio** igual a **borrado**

- Utilizamos **count**
- Anidamos ambos **match** en **bool** para poder utilizarlos y usamos **must**

###### Request :

```json
GET log_consultas/_count
{
  "query": {
    "bool": {
      "must": [
        { "match": { "estado_consulta":  "informativo" } },
        { "match": { "servicio":  "borrado" } }
      ]
    }
  }
}
```
###### Response :

```json
{
  "count": 52,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  }
}
```

#### Registros : 52


##### 4) Sumatoria **consultas_realizadas** con **estado_consulta** igual a error  :

- Utilizamos **search**
- Utilizamos la clausula **size** para solo ver el **agg** y no toda la tabla y llamamos a nuestra sumatoria de **consultas_realizadas**  como **total_consultas**

###### Request :

```json
GET log_consultas/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "match": { "estado_consulta": "error" }
      }
    }
  },
  "size": 0,
  "aggs": {
    "total_consultas": {
      "sum": {
        "field": "consultas_realizadas"
      }
    }
  }
}
```

###### Response :

```json
{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 78,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "total_consultas": {
      "value": 2865
    }
  }
}
```

#### Consultas Totales : 2865