# Scudo - Red de Comportamiento: Servicio de riesgo de fraude

## Tabla de Contenidos

1. [Descripción General](#descripcion_general)
2. [Rest Api](#rest_api)
    1. [Autenticación](#auth)
    2. [Consulta de riesgo de fraude](#consulta_riesgo_fraude)

## Descripción General <a name="descripcion_general"></a>

Scudo Red de Comportamiento es una solución para la detección de fraudes que se adapta a los diferentes patrones de fradues bajo un escenario de desbalance de clases (donde el numero de trasacciones legitimas es mucho mas grande de las trasacciones fraudulentas)

El proposito de esta solucion es el de la construccion de un sistema de evaluacion del riesgo basado en metodos de aprendizaje de maquina que estimé un puntaje de sospecha de fraude e índique cuales son los grupos de caracteristicas que llevaron a estimar tal puntaje

## REST API <a name="rest_api"></a>

### Autenticación <a name="auth"></a>

Debes autentícarte antes para realizar invocaciones del servicio de predicción.
Para auntenticarte primero debes crear un token JWT autofirmado usando la cuenta de servicio proporcionada 'json_cred.json' para intercambiarlo por un token de ID firmado por Google. La cuenta de servicio es la misma que se esta utilizando para realizar el cargue de los deltas desde on-premise - Google Cloud

1. Obtenga la clave privada y correo electronico que vienen del service account:

    pkey_id = json_cred['private_key'] - Ej: "-----BEGIN PRIVATE KEY-----\nMIGEAgEAMBAGByqGSM49AgEGBS..."

    email = json_cred['client_email'] - Ej: "sa-evertec@escudo-redcomp.iam.gserviceaccount.com"  

2. Genere el JWT autofirmado. En este paso puede configurar el tiempo de expiración del token, tambien debe tener el 'target_audience' que es el endpoint del servicio predictor:

    additional_headers = 
    ```json
    {
        "kid":"-----BEGIN PRIVATE KEY-----\nMIGEAgEAMBAGByqGSM49AgEGBS...",
        "alg":"RS256",
        "typ":"JWT"
    }    
    ```
    
    payload = 
    ```json
    {
        "iss":"sa-evertec@escudo-redcomp.iam.gserviceaccount.com",
        "sub":"sa-evertec@escudo-redcomp.iam.gserviceaccount.com",
        "aud":"https://www.googleapis.com/oauth2/v4/token",
        "iat":"tiempo_de_generacion",
        "exp":"issued + tiempo_de_expiración",
        "target_audience":"https://us-central1-escudo-redcomp.cloudfunctions.net/function-1"
    }
    ```

3. Codifique `aditional_headers` y `payload` y **fírmelos** creando un JWT usando `RS256` 

Con lo anterior ya se puede intercambiar el JWT autofirmado por el ID token firmado por google:

#### Request URL

<https://www.googleapis.com/oauth2/v4/token>

### Parámetros

**Headers**: Contiene el tipo de autorizacion y contenido de la cabecera.

| **Requerido**  | **Nombre**   | **Tipo**     | **Descripción**  |
| :------------- | :----------: | :----------- | :----------- |
| Sí | Autorization   | String    | "Bearer  + **signed_jwt**"  |
| Sí | Content-Type      | String    | application/x-www-form-urlencoded          |

**Query Params**:

| **Requerido**  | **Nombre**   | **Tipo**     | **Descripción**  |
| :------------- | :----------: | :----------- | :----------- |
| Sí | grant_type   | String    | urn:ietf:params:oauth:grant-type:jwt-bearer   |
| Sí | assertion      | String    | **signed_jwt**          |

#### Ejemplo Request

**Headers** :
```json
{
   "Autorization":"Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IjUxY2VjM2JiZTJiODAwNWVlYTA1NDQ4MTE5NGExMDQxZTllZDRjZTgifQ.eyJpc3MiOiJzYS1ldmVydGVjQGVzY3Vkby1yZWRjb21wLmlhbS5nc2VydmljZWFjY291bnQuY29tIiwic3ViIjoic2EtZXZlcnRlY0Blc2N1ZG8tcmVkY29tcC5pYW0uZ3NlcnZpY2VhY2NvdW50LmNvbSIsImF1ZCI6Imh0dHBzOi8vd3d3Lmdvb2dsZWFwaXMuY29tL29hdXRoMi92NC90b2tlbiIsImlhdCI6MTYzNTQ0NzkzMywiZXhwIjoxNjM1NDUxNTMzLCJ0YXJnZXRfYXVkaWVuY2UiOiJodHRwczovL3VzLWNlbnRyYWwxLWVzY3Vkby1yZWRjb21wLmNsb3VkZnVuY3Rpb25zLm5ldC9mdW5jdGlvbi0xIn0.atmUByDD9UHsMs3pVjoqvUDDQYMLhxbb0c_VUoRYnIohIcRHC4uZLYpOr9tUZmhxllqKVS43kh7KSHepvm507HATHEjFWb8zg1hamgjtoDULxplMU82jo7CHC6HMRg4oj41LMSlKBxXC8fKVsdovtXDPY7XPgRPRPQRcAz4vDxnjUvEiM4x-grU6EUZ_VPzRs68WOZMGx0a-ELOir7UOIBniHRz3xDOf-g14voZqv5vm_acJea9yOpQQNxEhU345VZ6Vd2jVDW0xMeIsq4PaMjxP7E3CU3D0KBxL703aNwVb7tOAaXc2iqDA7Uz-GW2Ar4a_ct8Fv62FcTWaPjxmcw",
   "Content-Type":"application/x-www-form-urlencoded"
}
```
**Query Params** :
```json
{
   "grant_type":"urn:ietf:params:oauth:grant-type:jwt-bearer",
   "assertion":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IjUxY2VjM2JiZTJiODAwNWVlYTA1NDQ4MTE5NGExMDQxZTllZDRjZTgifQ.eyJpc3MiOiJzYS1ldmVydGVjQGVzY3Vkby1yZWRjb21wLmlhbS5nc2VydmljZWFjY291bnQuY29tIiwic3ViIjoic2EtZXZlcnRlY0Blc2N1ZG8tcmVkY29tcC5pYW0uZ3NlcnZpY2VhY2NvdW50LmNvbSIsImF1ZCI6Imh0dHBzOi8vd3d3Lmdvb2dsZWFwaXMuY29tL29hdXRoMi92NC90b2tlbiIsImlhdCI6MTYzNTQ0NzkzMywiZXhwIjoxNjM1NDUxNTMzLCJ0YXJnZXRfYXVkaWVuY2UiOiJodHRwczovL3VzLWNlbnRyYWwxLWVzY3Vkby1yZWRjb21wLmNsb3VkZnVuY3Rpb25zLm5ldC9mdW5jdGlvbi0xIn0.atmUByDD9UHsMs3pVjoqvUDDQYMLhxbb0c_VUoRYnIohIcRHC4uZLYpOr9tUZmhxllqKVS43kh7KSHepvm507HATHEjFWb8zg1hamgjtoDULxplMU82jo7CHC6HMRg4oj41LMSlKBxXC8fKVsdovtXDPY7XPgRPRPQRcAz4vDxnjUvEiM4x-grU6EUZ_VPzRs68WOZMGx0a-ELOir7UOIBniHRz3xDOf-g14voZqv5vm_acJea9yOpQQNxEhU345VZ6Vd2jVDW0xMeIsq4PaMjxP7E3CU3D0KBxL703aNwVb7tOAaXc2iqDA7Uz-GW2Ar4a_ct8Fv62FcTWaPjxmcw"
}
```

#### Response Data

| Campo      | Tipo     | Descripción |
| :------------- | :---------- | :----------- |
|id_token  | String    | ID Token firmado por Google, necesario para hacer invocacion del servicio de predicción

#### Ejemplo Response

**Response**:

```json
{
    "id_token":"'eyJhbGciOiJSUzI1NiIsImtpZCI6ImJiZDJhYzdjNGM1ZWI4YWRjOGVlZmZiYzhmNWEyZGQ2Y2Y3NTQ1ZTQiLCJ0eXAiOiJKV1QifQ.eyJhdWQiOiJodHRwczovL3VzLWNlbnRyYWwxLWVzY3Vkby1yZWRjb21wLmNsb3VkZnVuY3Rpb25zLm5ldC9mdW5jdGlvbi0xIiwiYXpwIjoic2EtZXZlcnRlY0Blc2N1ZG8tcmVkY29tcC5pYW0uZ3NlcnZpY2VhY2NvdW50LmNvbSIsImVtYWlsIjoic2EtZXZlcnRlY0Blc2N1ZG8tcmVkY29tcC5pYW0uZ3NlcnZpY2VhY2NvdW50LmNvbSIsImVtYWlsX3ZlcmlmaWVkIjp0cnVlLCJleHAiOjE2MzU0NTE3NTQsImlhdCI6MTYzNTQ0ODE1NCwiaXNzIjoiaHR0cHM6Ly9hY2NvdW50cy5nb29nbGUuY29tIiwic3ViIjoiMTE2MzQwNDg5NzQ1NzE0MjczMDM4In0.rLwCdM7I0gdDZWrSxhiFpMfuRPDHCb6Cyn_dqPTRtR43RUUxfWg8Twb-tScl4zCPXbKbWIe8n8c3ZFKF4S5hvihd-GyalTYbKV6ZbSfsCuikg5GykCpGb-A8_V2mtScrM_VybNGgywLxJyURmcLC6uxKLKgECgmAf5brRdZyukT7m1soC8jnBLIaA58pqDO1-MboTXxmjW2lnmx978-Z4q1KchGvvmWzuF9ilo8DpJ3vrfNUNshOgEgQ1KeG98-DAQS6RLq2YUgPEQfMSTXbusYgs-znvS1FEyra3fodbOt7g6Wv_0Z39qMhDZxNW1Z_jM3DlPzgv4jKamP34p5JMQ'"
}

```

### Consulta de riesgo de fraude - servicio de prediccion<a name="consulta_riesgo_fraude"></a>

Esta sección explica como realizar el request al servicio asumiendo que ya se realizo el flujo autenticación.

#### Descripcion de campos

**Headers**: Se debe utilizar el id_token obenido del servicio de autorizacion para identificar el tipo.

| **Requerido**  | **Nombre**   | **Tipo**     | **Descripción**  |
| :------------- | :----------: | :----------- | :----------- |
| Sí | Autorization   | String    | "Bearer  + **id_token**" |


**Body**:
En la siguiente tabla se muestran los campos requeridos para realizar la consulta al servicio de sobre el riesgo de fraude de una transacción.

| Campo      | Requerido | Tipo | Hash | Acepta vacio |Descripción     |
| :------------- | :----------: |:----------: | :----------: | :----------: |:----------: |
| idsesion | Si | String | No | No | identificador unico de la transaccion. Ese valor sera usado para monitorear y auditar la transacción|
| transaction_processing_date | Si | String | No | No | Fecha de procesamiento de la transaccion en formato '%Y-%m-%d %H:%M:%S'. Se asume timezone UTC-5|
| transaction_processing_amount | Si | float/int | No | No | Monto de la transacción. |
| transaction_card_id | Si | string | Si| Si|identificador de la tarjeta. |
| transaction_retail_code | Si | String | No | Si | identificador del retail code de la transacción. |
| transaction_payer_id | Si | String | Si | Si |Identificador del pagador |
| transaction_payer_email | Si | String | Si | Si | Email del pagador |
| ip_location_country  | Si | String | No | Si | Pais donde se esta ejecutando la transacción |
| ip_location_city  | Si | String | No | Si |Ciuidad donde se esta ejecutando la transacción |
| merchant_ciiu  | Si | Int | No | Si |Codigo ciiu del comercio |
| merchant_isic_division_id | Si | Int | No | Si| Codigo isic de la division del comercio |
| card_country | Si | String | No | Si |Codigo Iso del pais de la tarjeta|
| card_bin | Si | String | No | Si |Codigo Bin del de la tarjeta|
| transaction_card_installments  | Si | Int | No | Si |numero de cuotas de la transaccion|
| transaction_payer_phone | Si | String | Si | Si |numero de telefono del pagador|
| transaction_payer_mobile | Si | String | Si | Si |numero de telefono movil del pagador|
| transaction_ip_address | Si | String | No | Si |Ip de la transacción |
| user_agent | Si | String | No | Si |User agent de la transacción. No se comprueba si esl User agent es valido, se usa para extraer el navegador y el sistema operativo|

**NOTA campos vacios**:  Para los campos marcados con `Acepta vacio=Si`, si la transaccion no tiene valor del campo requerido se debe enviar un string vacio '' o vacio en el caso de campos numericos.

**NOTA valores hash**: Para los campos marcados con `Hash=Si` los valores hash deben coincidir con los valores hash previamente enviados mediante el delta que se esta cargando a GCP de manera recurrente.

Una vez el servicio procesa los campos, responde con los siguientes campos de respuesta

| Campo      | Tipo | Rango | Descripción     |
| :------------- | :----------: | :----------: | :----------: |
| fraud_score | float | 0.0-1.0 | Indica el riesgo de fraude calculado |
| fraud_concept | list[str] | F1-F24 | Corresponde a los codigo de las razones de fraude que son explicadas en la siguiente sección |
| legal_score | float | 0.0-1.0 | Indicador legalidad de la transacción, siempre es 1-fraud_score |
| legal_concept | list[str] | L1-L24 | Corresponde a los codigo de las razones por la cual el servicio asigna el `legal_score`. El significado de se encuentra en la siguiente sección |
| fraud_score | Boolean | True, False | Indica si la respuesta es confiable. Un False indica que el proceso ha detectado errores en la actualización de la información en GCP, y por lo tanto el resultado no es confiable |


#### Descripción de fraud_concept

De acuerdo a los patrones detectados, el servicio responde las razones que influyeron en el calculo de riesgo de fraude dado. Estas codificaciones tienen que ser usadas para determinar el riesgo de exposición de fraude y tomar acciones sobre la transacción procesada.

| fraude descripcion                                                      | fraude_code   |
|:------------------------------------------------------------------------|:--------------|
| Diferencia con historial de pago                                        | F1            |
| Baja confiabilidad del documento de cliente                             | F2            |
| Uso No común en los datos de identificación                             | F3            |
| Horario inusual de compra                                               | F4            |
| Ubicación inusual de compra                                             | F5            |
| Frecuencia inusual de compra en el comercio                             | F6            |
| Dispostivo inusual de compra                                            | F7            |
| calidad datos en email                                                  | F8            |
| calidad de datos en ubicacion                                           | F9            |
| calidad de datos en tarjeta                                             | F10           |
| calidad de datos de seccion de comercio                                 | F11           |
| calidad de datos en identificador de comercio                           | F12           |
| calidad de datos en retailcode de comercio                              | F13           |
| calidad de datos del documento                                          | F14           |
| calidad de datos del documento                                          | F15           |
| calidad de datos en la ip                                               | F16           |
| Pais de emision de la tarjeta inusual                                   | F17           |
| Frecuencia inusal de compra en el comercio                              | F18           |
| Frecuencia inusual de la IP                                             | F19           |
| Frecuencia inusual de lote de tarjeta                                   | F20           |
| Empresas fachada - Frecuencia inusal de tx apr-rec en comercio          | F21           |
| Empresas fachada - Frecuencia inusal de pais emisor tarjeta en comercio | F22           |
| calidad de datos en el navegador                                        | F23           |
| calidad de datos en sistema operativo                                   | F24           |

#### Descripción de legal_concept

De acuerdo a los patrones detectados, el servicio responde las razones que influyeron en el calculo del score de legalidad dado. Estas codificaciones tienen que ser usadas para determinar el riesgo de exposición de fraude y tomar acciones sobre la transacción procesada.

| legal descripcion                                   | legal_code   |
|:----------------------------------------------------|:-------------|
| Similitud con historial de pago                     | L1           |
| confiabilidad del documento de cliente              | L2           |
| Uso común en los datos de identificación            | L3           |
| Horario usual de compra                             | L4           |
| Ubicación usual de compra                           | L5           |
| Frecuencia usal de compra en el comercio            | L6           |
| Dispostivo usual de compra                          | L7           |
| calidad datos en email                              | L8           |
| calidad de datos en ubicacion                       | L9           |
| calidad de datos en tarjeta                         | L10          |
| calidad de datos de seccion de comercio             | L11          |
| dato identificador de comercio faltante             | L12          |
| calidad de datos en retailcode de comercio          | L13          |
| calidad de datos del documento                      | L14          |
| calidad de datos del telefono                       | L15          |
| calidad de datos en la ip                           | L16          |
| Pais de emision de la tarjeta usual                 | L17          |
| Frecuencia usual de compra en el comercio           | L18          |
| Frecuencia usual de la IP                           | L19          |
| Frecuencia usual de lote de tarjeta                 | L20          |
| Frecuencia usual de tx apr-rec en comercio          | L21          |
| Frecuencia usal de pais emisor tarjeta  en comercio | L22          |
| calidad de datos en el navegador                    | L23          |
| calidad de datos en sistema operativo               | L24          |

#### Datos de prueba y descripción

En esta sección se describen valores ficticios que pueden ser usados para realizar preubas de integración en el servicio.

##### Request URL de pruebas

<https://centraal.dev>

**NOTA**: Para la integración en sistema productivo se debe solicar la URL y su certificado. Este endpoint no da respuestas que no esten explicadas en las tablas de la siguientes secciones.

##### Ejemplos

La siguiente tabla contiene valores que pueden ser usados para realizar pruebas de integración.

\# Ej | idsesion | transaction_processing_date |transaction_processing_amount | transaction_card_id | transaction_retail_code | transaction_payer_id | transaction_payer_email | ip_location_country | ip_location_city | merchant_ciiu | merchant_isic_division_id | card_country | card_bin | transaction_card_installments | transaction_payer_phone | transaction_payer_mobile | transaction_ip_address | user_agent |
| :------------- | :------------- | :----------: | :----------: |:----------: |:----------: |:----------: |:----------: |:----------: |:----------: |:----------: |:----------: |:----------: |:----------: |:----------: |:----------: |:----------: |:----------: |:----------: |
|         1 | cc733c92685345f68e49bec741188ebb | Fecha de hoy                  |                            1000 | 004C93004C93004C93004C93004C9304C9304C93 |                1111111111 | 43434343434343434343434333 | 0123ABC0123ABC0123ABC0123ABC0123ABC0123A | CO                    | Bogota             |            8888 |                          66 | CO             | 123456     |                               1 |                           | 45678987456547898745       | 59.208.38.243            | Mozilla/5.0 (iPhone; CPU iPhone OS 14_2 like Mac OS X) AppleWebKit/536.1 (KHTML, like Gecko) CriOS/50.0.894.0 Mobile/00Q341 Safari/536.1 |
|         2 | a41f020c2d4d433fb1d3979f1043fae0 | Fecha de hoy                  |                           17900 | 5904590459045904590459045904590459045904 |                2222222222 | 12312312312312312312312312 | 0123ABC0123ABC0123ABC0123ABC0123ABC0123A | CO                    | Medellin           |            7777 |                          72 | CO             | 123456     |                               6 | 32123321233212332132      |                            | 171.50.70.174            | Opera/8.46.(Windows NT 6.0; so-DJ) Presto/2.9.165 Version/12.00                                                                          |
|         3 | aca531aee7ba40c39d0cfbfb5d8ca6c2 | Fecha de hoy                  |                           18900 |                                          |                1111111111 | 43434343434343434343434333 | 0123ABC0123ABC0123ABC0123ABC0123ABC0123A | CO                    | Cali               |            8888 |                          66 | CO             |            |                              36 |                           | 45678987456547898745       | 211.17.156.245           | Opera/9.48.(X11; Linux x86_64; hi-IN) Presto/2.9.178 Version/11.00                                                                       |
|         4 | 9626bf792f974c0c9aaede080adab7df | Fecha de hoy                  |                           10900 | 004C93004C93004C93004C93004C9304C9304C93 |                2222222222 | 12312312312312312312312312 | 0123ABC0123ABC0123ABC0123ABC0123ABC0123A | CO                    | Bogota             |            7777 |                          72 | CO             | 123456     |                               1 | 32123321233212332132      |                            | 59.208.38.243            | Mozilla/5.0 (iPhone; CPU iPhone OS 14_2 like Mac OS X) AppleWebKit/536.1 (KHTML, like Gecko) CriOS/50.0.894.0 Mobile/00Q341 Safari/536.1 |
|         5 | 69261bc24a714de7bc8b1beb0d9320ac | Fecha de hoy                  |                           50900 | 5904590459045904590459045904590459045904 |                1111111111 | 43434343434343434343434333 | 0123ABC0123ABC0123ABC0123ABC0123ABC0123A | CO                    | Medellin           |            8888 |                          66 | CO             | 123456     |                               6 |                           | 45678987456547898745       | 171.50.70.174            | Opera/8.46.(Windows NT 6.0; so-DJ) Presto/2.9.165 Version/12.00                                                                          |
|         6 | a82c1e8766b244a6bb4b76449a53270c | Fecha de hoy                  |                          500000 |                                          |                2222222222 | 12312312312312312312312312 | 0123ABC0123ABC0123ABC0123ABC0123ABC0123A | CO                    | Cali               |            7777 |                          72 | CO             |            |                              36 | 32123321233212332132      |                            | 211.17.156.245           | Opera/9.48.(X11; Linux x86_64; hi-IN) Presto/2.9.178 Version/11.00                                                                       |
|         7 | 7217d7d26f244bf5942d3e4cf15982c1 | Fecha de hoy                  |                         1500000 | 004C93004C93004C93004C93004C9304C9304C93 |                1111111111 | 43434343434343434343434333 | 0123ABC0123ABC0123ABC0123ABC0123ABC0123A | CO                    | Bogota             |            8888 |                          66 | US             | 123456     |                               1 |                           | 45678987456547898745       | 59.208.38.243            | Mozilla/5.0 (iPhone; CPU iPhone OS 14_2 like Mac OS X) AppleWebKit/536.1 (KHTML, like Gecko) CriOS/50.0.894.0 Mobile/00Q341 Safari/536.1 |
|         8 | 01b96a66733e47faabbd3011ea911cb2 | Fecha de hoy                  |                            1000 | 5904590459045904590459045904590459045904 |                2222222222 | 12312312312312312312312312 | 0123ABC0123ABC0123ABC0123ABC0123ABC0123A | CO                    | Medellin           |            7777 |                          72 | CO             | 123456     |                               6 | 32123321233212332132      |                            | 171.50.70.174            | Opera/8.46.(Windows NT 6.0; so-DJ) Presto/2.9.165 Version/12.00                                                                          |
|         9 | 7019f6a5277149718825bdd51d2a12b3 | Fecha de hoy                  |                           17900 |                                          |                1111111111 | 43434343434343434343434333 | 0123ABC0123ABC0123ABC0123ABC0123ABC0123A | CO                    | Cali               |            8888 |                          66 | CO             |            |                              36 |                           | 45678987456547898745       | 211.17.156.245           | Opera/9.48.(X11; Linux x86_64; hi-IN) Presto/2.9.178 Version/11.00                                                                       |
|        10 | a5e24f10dae74e5bbfe023f83bf83840 | Fecha de hoy                  |                           18900 | 004C93004C93004C93004C93004C9304C9304C93 |                2222222222 | 12312312312312312312312312 | 0123ABC0123ABC0123ABC0123ABC0123ABC0123A | CO                    | Bogota             |            7777 |                          72 | CO             | 123456     |                               1 | 32123321233212332132      |                            | 59.208.38.243            | Mozilla/5.0 (iPhone; CPU iPhone OS 14_2 like Mac OS X) AppleWebKit/536.1 (KHTML, like Gecko) CriOS/50.0.894.0 Mobile/00Q341 Safari/536.1 |
|        11 | a03345ac96b34bbebc2dbe2943446578 | Fecha de hoy                  |                           10900 | 5904590459045904590459045904590459045904 |                1111111111 | 43434343434343434343434333 | 0123ABC0123ABC0123ABC0123ABC0123ABC0123A | CO                    | Medellin           |            8888 |                          66 | CO             | 123456     |                               6 |                           | 45678987456547898745       | 171.50.70.174            | Opera/8.46.(Windows NT 6.0; so-DJ) Presto/2.9.165 Version/12.00                                                                          |
|        12 | 566d86878fb645c2b5f82486d147bcd2 | Fecha de hoy                  |                           50900 |                                          |                2222222222 | 12312312312312312312312312 | 0123ABC0123ABC0123ABC0123ABC0123ABC0123A | CO                    | Cali               |            7777 |                          72 | CO             |            |                              36 | 32123321233212332132      |                            | 211.17.156.245           | Opera/9.48.(X11; Linux x86_64; hi-IN) Presto/2.9.178 Version/11.00                                                                       |
|        13 | d9117f5238394641a4707de16f437d8b | Fecha de hoy                  |                          500000 | 004C93004C93004C93004C93004C9304C9304C93 |                1111111111 | 43434343434343434343434333 | 0123ABC0123ABC0123ABC0123ABC0123ABC0123A | CO                    | Bogota             |            8888 |                          66 | CO             | 123456     |                               1 |                           | 45678987456547898745       | 59.208.38.243            | Mozilla/5.0 (iPhone; CPU iPhone OS 14_2 like Mac OS X) AppleWebKit/536.1 (KHTML, like Gecko) CriOS/50.0.894.0 Mobile/00Q341 Safari/536.1 |
|        14 | 4bd9201331c54f87b9c57148993aea1c | Fecha de hoy                  |                         1500000 | 5904590459045904590459045904590459045904 |                2222222222 | 12312312312312312312312312 | 0123ABC0123ABC0123ABC0123ABC0123ABC0123A | CO                    | Medellin           |            7777 |                          72 | US             | 123456     |                               6 | 32123321233212332132      |                            | 171.50.70.174            | Opera/8.46.(Windows NT 6.0; so-DJ) Presto/2.9.165 Version/12.00                                                                          |
|        15 | 59a7546dca9f46c9a033bb9d3bf2b86f | Fecha de hoy                  |                            1000 |                                          |                1111111111 | 43434343434343434343434333 | 0123ABC0123ABC0123ABC0123ABC0123ABC0123A | CO                    | Cali               |            8888 |                          66 | CO             |            |                              36 |                           | 45678987456547898745       | 211.17.156.245           | Opera/9.48.(X11; Linux x86_64; hi-IN) Presto/2.9.178 Version/11.00                                                                       |
|        16 | 11804b641b8c4606a3eb8ba56cadfe2c | Fecha de hoy                  |                           17900 | 004C93004C93004C93004C93004C9304C9304C93 |                2222222222 | 12312312312312312312312312 | 0123ABC0123ABC0123ABC0123ABC0123ABC0123A | CO                    | Bogota             |            7777 |                          72 | CO             | 123456     |                               1 | 32123321233212332132      |                            | 59.208.38.243            | Mozilla/5.0 (iPhone; CPU iPhone OS 14_2 like Mac OS X) AppleWebKit/536.1 (KHTML, like Gecko) CriOS/50.0.894.0 Mobile/00Q341 Safari/536.1 |
|        17 | 9e65861e41bf4b1e95ebea32c7815ad2 | Fecha de hoy                  |                           18900 | 5904590459045904590459045904590459045904 |                1111111111 | 43434343434343434343434333 | 0123ABC0123ABC0123ABC0123ABC0123ABC0123A | CO                    | Medellin           |            8888 |                          66 | CO             | 123456     |                               6 |                           | 45678987456547898745       | 171.50.70.174            | Opera/8.46.(Windows NT 6.0; so-DJ) Presto/2.9.165 Version/12.00                                                                          |
|        18 | 3e845e4658044f4bb1c3296931fb5bd1 | Fecha de hoy                  |                           10900 |                                          |                2222222222 | 12312312312312312312312312 | 0123ABC0123ABC0123ABC0123ABC0123ABC0123A | CO                    | Cali               |            7777 |                          72 | CO             |            |                              36 | 32123321233212332132      |                            | 211.17.156.245           | Opera/9.48.(X11; Linux x86_64; hi-IN) Presto/2.9.178 Version/11.00                                                                       |
|        19 | 4eb8bd82bb8b41329086a9edafd8f8eb | Fecha de hoy                  |                           50900 | 004C93004C93004C93004C93004C9304C9304C93 |                1111111111 | 43434343434343434343434333 | 0123ABC0123ABC0123ABC0123ABC0123ABC0123A | CO                    | Bogota             |            8888 |                          66 | CO             | 123456     |                               1 |                           | 45678987456547898745       | 59.208.38.243            | Mozilla/5.0 (iPhone; CPU iPhone OS 14_2 like Mac OS X) AppleWebKit/536.1 (KHTML, like Gecko) CriOS/50.0.894.0 Mobile/00Q341 Safari/536.1 |
|        20 | af90e67e73f049efaec35b986527fe9d | 2021-07-30 16:17:18           |                          500000 | 5904590459045904590459045904590459045904 |                2222222222 | 12312312312312312312312312 | 0123ABC0123ABC0123ABC0123ABC0123ABC0123A | CO                    | Medellin           |            7777 |                          72 | CO             | 123456     |                               6 | 32123321233212332132      |                            | 171.50.70.174            | Opera/8.46.(Windows NT 6.0; so-DJ) Presto/2.9.165 Version/12.00                                                                          |

De acuerdo al número de ejemplo usado, el servicio responde con las siguientes repuestas.

|\# Ej | fraud_score | fraud_concept |legal_score| legal_concept | reliable |
| :-------------  |:------------- |:------------- | :------------- |:------------- |:------------- |
|         1 |          0    | ['F1', 'F2', 'F3']    |          1    | ['L9', 'L8', 'L7']    | True       |
|         2 |          0.25 | ['F4', 'F5', 'F6']    |          0.75 | ['L6', 'L5', 'L4']    | True       |
|         3 |          0.45 | ['F7', 'F8', 'F9']    |          0.55 | ['L3', 'L24', 'L23']  | True       |
|         4 |          0.5  | ['F10', 'F11', 'F12'] |          0.5  | ['L22', 'L21', 'L20'] | True       |
|         5 |          0.75 | ['F13', 'F14', 'F15'] |          0.25 | ['L2', 'L19', 'L18']  | True       |
|         6 |          0.9  | ['F16', 'F17', 'F18'] |          0.1  | ['L17', 'L16', 'L15'] | True       |
|         7 |          0.95 | ['F19', 'F20', 'F21'] |          0.05 | ['L14', 'L13', 'L12'] | True       |
|         8 |          0.99 | ['F22', 'F23', 'F24'] |          0.01 | ['L11', 'L10', 'L1']  | True       |
|         9 |          1    | ['F1', 'F2', 'F3']    |          0    | ['L9', 'L8', 'L7']    | True       |
|        10 |          0    | ['F4', 'F5', 'F6']    |          1    | ['L6', 'L5', 'L4']    | True       |
|        11 |          0.25 | ['F7', 'F8', 'F9']    |          0.75 | ['L3', 'L24', 'L23']  | True       |
|        12 |          0.45 | ['F10', 'F11', 'F12'] |          0.55 | ['L22', 'L21', 'L20'] | True       |
|        13 |          0.5  | ['F13', 'F14', 'F15'] |          0.5  | ['L2', 'L19', 'L18']  | True       |
|        14 |          0.75 | ['F16', 'F17', 'F18'] |          0.25 | ['L17', 'L16', 'L15'] | True       |
|        15 |          0.9  | ['F19', 'F20', 'F21'] |          0.1  | ['L14', 'L13', 'L12'] | True       |
|        16 |          0.95 | ['F22', 'F23', 'F24'] |          0.05 | ['L11', 'L10', 'L1']  | True       |
|        17 |          0.99 | ['F1', 'F2', 'F3']    |          0.01 | ['L9', 'L8', 'L7']    | True       |
|        18 |          1    | ['F4', 'F5', 'F6']    |          0    | ['L6', 'L5', 'L4']    | True       |
|        19 |          0    | ['F7', 'F8', 'F9']    |          1    | ['L3', 'L24', 'L23']  | True       |
|        20 |          0.25 | ['F1', 'F2', 'F3']    |          0.75 | ['L9', 'L8', 'L7']    | False      |

Se usan los parametros del Ejemplo \# 1, para mostrar el contenido esperado en formato `json` para el request y el response del servicio.

**request**:

**Headers**:
```json
  {
    "Authorization":"Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6Ijg1ODI4YzU5Mjg0YTY5YjU0YjI3NDgzZTQ4N2MzYmQ0NmNkMmEyYjMiLCJ0eXAiOiJKV1QifQ.eyJhdWQiOiJodHRwczovL3VzLWNlbnRyYWwxLWVzY3Vkby1yZWRjb21wLmNsb3VkZnVuY3Rpb25zLm5ldC9kZXZfcHJlZGljdG9yX3NlcnZpY2UiLCJhenAiOiJzYS1ldmVydGVjQGVzY3Vkby1yZWRjb21wLmlhbS5nc2VydmljZWFjY291bnQuY29tIiwiZW1haWwiOiJzYS1ldmVydGVjQGVzY3Vkby1yZWRjb21wLmlhbS5nc2VydmljZWFjY291bnQuY29tIiwiZW1haWxfdmVyaWZpZWQiOnRydWUsImV4cCI6MTYzNTg2OTAzMiwiaWF0IjoxNjM1ODY1NDMyLCJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20iLCJzdWIiOiIxMTYzNDA0ODk3NDU3MTQyNzMwMzgifQ.A_21MDk5rL41s2Wq0I20IN5si9Bv4Be5ZQc68BGIe9rUXOPuTRr-cdmyIHdS54G0FSc7ZJ1OBkTVb7JpjHyi41xEwzG5c7YP3Y51-KE6lVEvI1lBFDAOo6ymcQbRR4bNXNxSjYGMOFT7ZXj5FP-HF9jxvH-NyOASbVGhRGO9uznESVUwgD_vU5xaiJ0Y0Y2_gpPQmwBc7cvuYL6dG7B8FRSupxwRZNBD1I01wkPTw2RlwwHU1LcHq_2jzz4KWnpQKggSqj63RaQPlWr9ky2qQYj9YN3Xe2G9hSS1MTJaNr78YnJLr2QaT_QnGi1acZs19i-YgorZFDr99YStzF8_7g"
  }
```

**Body**:
```json
  {
    "idsesion": "cc733c92685345f68e49bec741188ebb",
    "transaction_processing_date": "2021-12-30 16:34:47",
    "transaction_processing_amount": 10000.0,
    "transaction_card_id": "004C93004C93004C93004C93004C9304C9304C93",
    "transaction_retail_code": "1111111111",
    "transaction_payer_id": "43434343434343434343434333",
    "transaction_payer_email": "0123ABC0123ABC0123ABC0123ABC0123ABC0123A",
    "ip_location_country": "CO",
    "ip_location_city": "Bogota", 
    "merchant_ciiu": 8888,
    "merchant_isic_division_id": 66,
    "card_country": "CO",
    "card_bin": "123456",
    "transaction_card_installments": 1,
    "transaction_payer_phone": "",
    "transaction_payer_mobile": "45678987456547898745",
    "transaction_ip_address": "59.208.38.243",
    "user_agent": "Mozilla/5.0 (iPhone; CPU iPhone OS 14_2 like Mac OS X) AppleWebKit/536.1 (KHTML, like Gecko) CriOS/50.0.894.0 Mobile/00Q341 Safari/536.1"
  }
```

**Response**:

```json
{
    "fraud_score": 0.0, 
    "fraud_concept": ["F1", "F2", "F3"], 
    "legal_score": 1.0, 
    "legal_concept": ["L9", "L8", "L7"],
    "reliable": true
}
```
