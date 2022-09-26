###### [VOLVER AL INICIO](https://github.com/cell-framework-project/sps-elastic-search-exam)

##### Creamos antes que nada el servicio sps_practica :

![mapa de calor](https://github.com/cell-framework-project/sps-elastic-search-exam/blob/master/img/deployment.png)


##### 1) Creando índice **log_consultas**

- Request :

```json
PUT log_consultas
```

- Response :

```json
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "log_consultas"
}
```

##### 2) Insertando registro **log_consultas** 

- Request :


```json
POST log_consultas/_doc
{
  "@timestamp":"2010-05-15T22:00:54",
  "estado_consulta":"consumo",
  "servicio":"consulta",
  "administrador":"Juan Carlos",
  "consultas_realizadas":52
}
```
- Response :

```json
{
  "_index": "log_consultas",
  "_id": "zXfDbYMBi394gU72WjWg",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 2,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}

```


- Buscamos el registro insertado

###### Request :


```json
GET log_consultas/_search
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
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "log_consultas",
        "_id": "zXfDbYMBi394gU72WjWg",
        "_score": 1,
        "_source": {
          "@timestamp": "2010-05-15T22:00:54",
          "estado_consulta": "consumo",
          "servicio": "consulta",
          "administrador": "Juan Carlos",
          "consultas_realizadas": 52
        }
      }
    ]
  }
}
```

##### 3) Consultamos ahora el mapeado de datos del registro insertado en nuestro índice

###### Request :


```json
GET log_consultas/_mapping
```

###### Response :


```json
{
  "log_consultas": {
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date"
        },
        "administrador": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "consultas_realizadas": {
          "type": "long"
        },
        "estado_consulta": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "servicio": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        }
      }
    }
  }
}
```

- Tenemos el siguiente mapeado:

| name                 | type | 
|----------------------|------|
| @timestamp           | date |
| estado_consulta      | text |
| servicio             | text |
| administrador        | text |
| consultas_realizadas | long |

- Podemos hacer correciones en este mapeado para hacerlo más eficiente:

* por defecto los datos aparecen anidados en la respuesta, entonces estamos tomando en contacto el anidarlos
* @timestamp : **date** es el tipo correcto, por lo mismo permanece inalterada
* estado_consulta : al ser un dato categórico, **keyword** es mejor opción que **text**
* servicio : igualmente siendo un dato categórico, **keyword** es mejor que **text**
* administrador : podría ser tanto un dato categórico como uno de texto, pero para dar flexibilidad, lo consideraremos como **text**
* consultas_realizadas : podría usarse tanto **int** como **long** , pero para dejar más libertad, usaremos **long**

###### Por eso podemos definir el mapeado y tipado de datos de esta forma:

| name                 | type    | 
|----------------------|---------|
| @timestamp           | date    |
| estado_consulta      | keyword |
| servicio             | keyword |
| administrador        | text    |
| consultas_realizadas | long    |

##### 4) Carga en lote **BULK api**


- Eliminamos el índice y volvemos a crearlo, para ajustar el template y posteriormente cargar en lote

```json
DELETE log_consultas
```

```json
PUT log_consultas
```

- Con base al mapeado optimizado que planetamos, generaremos nuestro **template**


- Request:

```json
PUT _index_template/log_consultas
{
  "index_patterns" : ["log_consultas*"],
  "template": {
    "mappings" : {
      "properties": {
  
        "@timestamp": {
          "type": "date"
        },
        "estado_consulta": {
          "type": "keyword"
        },
        "servicio": {
          "type": "keyword"
        },
        "administrador": {
          "type": "text"
        },
        "consultas_realizadas": {
          "type": "long"
        }
      }
    }
  }
}
```

- Response :

```json
{
  "acknowledged": true
}
```

- Si volvemos a consultar el índice, ahora ya aparece con el mapeado determinado

```json
GET log_consultas/_mapping
```

```json
{
  "log_consultas": {
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date"
        },
        "administrador": {
          "type": "date"
        },
        "consultas_realizadas": {
          "type": "long"
        },
        "estado_consulta": {
          "type": "text"
        },
        "servicio": {
          "type": "keyword"
        }
      }
    }
  }
}
```

- Finalmente insertamos los registros en  lote

###### Request :

```json
POST log_consultas/_bulk
{"index":{"_index":"log_consultas","_id":1}}
{"@timestamp":"2010-05-15T22:00:54","estado_consulta":"consumo","servicio":"consulta","administrador":"Juan Carlos","consultas_realizadas":52}
{"index":{"_index":"log_consultas","_id":2}}
{"@timestamp":"2010-05-15T12:55:04","estado_consulta":"consumo","servicio":"modificacion","administrador":"Juan Lara","consultas_realizadas":10}
{"index":{"_index":"log_consultas","_id":3}}
{"@timestamp":"2010-05-15T14:56:48","estado_consulta":"consumo","servicio":"consulta","administrador":"Juan Lara","consultas_realizadas":20}
{"index":{"_index":"log_consultas","_id":4}}
...
```

###### Response :

```json
{
  "took": 74,
  "errors": false,
  "items": [
    {
      "index": {
        "_index": "log_consultas",
        "_id": "1",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 2,
          "failed": 0
        },
        "_seq_no": 0,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "log_consultas",
        "_id": "2",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 2,
          "failed": 0
        },
        "_seq_no": 1,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "index": {
        "_index": "log_consultas",
        "_id": "3",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 2,
          "failed": 0
        },
        "_seq_no": 2,
        "_primary_term": 1,
        "status": 201
      }
    },
    ...
  ]
}
```
- Hacemos una breve consulta para verificar el conteo de registros :

###### Request:

```json
GET log_consultas/_count
```
###### Response:

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

#### Registros insertados 299