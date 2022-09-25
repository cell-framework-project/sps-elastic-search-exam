#### II HACIENDO CONSULTAS:

##### Tenemos como base este mapeado optimizado que hemos generado

| name                 | type    | 
|----------------------|---------|
| @timestamp           | date    |
| estado_consulta      | keyword |
| servicio             | keyword |
| administrador        | text    |
| consultas_realizadas | long    |

##### 1) : Conteo de registros con **estado_consulta** igual a error y/o consumo:

- Request :

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
- Response :

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

##### 2) : Conteo de registros con **administrador**  **Juan Lara**

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

```json
{
  "count": 299,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  }
}
```

##### 3) : Conteo de registros con **estado_consulta** igual a **informativo** y **servicio** igual a **borrado**

- Request :

```json
GET log_consultas/_count
{
  "query": {
    
  }
}
```
- Response :

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

##### 4) Sumatoria **consultas_realizadas** con **estado_consulta** igual a error  :

```json
GET log_consultas/_search

```