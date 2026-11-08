

Para las pruebas de Canaritos se debe usar la notebook z635_exp6_controlador_canaritos, que realiza las corridas de la notebook base z635_WorkFlow_01_senior_exp6_canaritos.
Se debe setear los siguientes parametros:
- semillas: lista con las semillas.
- experimento_inicial: nro del primer experimento de la lista de semillas, luego cuando avance a cada semilla, se incrementara el nro de experimento en 1 unidad.
- notebook_senior= "z635_WorkFlow_01_senior_exp6_canaritos.ipynb".
- "PARAM$CN$desvios":  definir segun la prueba.
- "PARAM$CN$ratio":  definir segun la prueba.
- "PARAM$boruta$aplicacion_boruta":  Dejar en FALSE.

Para la prueba de Boruta se debe usar la anotebook z635_exp6_controlador_boruta, que realiza las corridas de la notebook base z635_WorkFlow_01_senior_exp6_boruta.
Se debe setear los siguientes parametros:
- semillas: lista con las semillas.
- experimento_inicial: nro del primer experimento de la lista de semillas, luego cuando avance a cada semilla, se incrementara el nro de experimento en 1 unidad.
- notebook_senior= "z635_WorkFlow_01_senior_exp6_canaritos.ipynb".
- "PARAM$boruta$max_runs": Se corrio con valor 100.
- "PARAM$boruta$alpha": Se corrio con valor 0,01.
- "PARAM$boruta$num_trees": Se corrio con valor 500.
- "PARAM$boruta$importancia": Se corrio con valor "gini".
- "PARAM$boruta$rough_fix": Se corrio con valor TRUE.

