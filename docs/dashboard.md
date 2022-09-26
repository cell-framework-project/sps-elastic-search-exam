#### DASHBOARDS EN KIBANA:

##### 1) Mapa de calor :
 - Eje Horizontal : **servicio**
 - Eje Vertical : **administrador**
 - Métrica : **consultas_realizadas**

![mapa de calor](https://github.com/cell-framework-project/sps-elastic-search-exam/blob/master/img/chart_1.png)

##### 2) Histograma :
 - Eje Horizontal : **@timestamp (en horas)**
 - Métrica : **consultas_realizadas**
 - Condición : **estado_consulta = error**
 - Segmentación Stacks(iniciativa propia) : **servicio**

![historgrama](https://github.com/cell-framework-project/sps-elastic-search-exam/blob/master/img/chart_2.png)

##### 3) Dasboard :
###### Adicionalmente se crearon 2 gráficos :

- Histórico completo de **consultas_realizadas** por **@timestamp(en horas)**
- Conteo de consultas_realizadas **consultas_realizadas** y **conteo de registros**

