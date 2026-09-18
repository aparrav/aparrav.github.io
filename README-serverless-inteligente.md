<!--
  README del repositorio "serverless-inteligente".
  Este es el CASO DE ESTUDIO que pide el proyecto integrador del modulo 10:
  cubre los ocho puntos del enunciado, en este orden.

  Los textos entre <angulares> son los que debes reemplazar.
  Los marcadores [captura: ...] indican donde conviene insertar una imagen.
-->

# Serverless Inteligente

> API REST de comercio electrónico sobre arquitectura sin servidores, con
> control de acceso en dos capas independientes y observabilidad completa.

`AWS Lambda` · `API Gateway` · `DynamoDB` · `Amazon S3` · `CloudWatch` · `Python`

---

## El problema

Construir una API de comercio electrónico —usuarios, pedidos y catálogo— que
cumpla tres condiciones simultáneas:

- **Sin servidores que administrar.** Ni instancias ni contenedores: el costo
  debe seguir al uso y caer a cero cuando no hay tráfico.
- **Con acceso controlado.** Ningún recurso expuesto sin autenticación, y con
  límites de consumo por cliente.
- **Observable.** Que ante una falla sea posible saber qué ocurrió, cuándo y
  en qué componente.

[captura: diagrama de arquitectura]

---

## El desafío principal

El obstáculo real no fue construir los servicios sino **hacer que los controles
de seguridad convivieran con el navegador**.

Una API protegida funciona sin problemas desde la línea de comandos. Desde un
sitio web, en cambio, el navegador introduce una capa intermedia —el
intercambio de recursos entre orígenes— que reacciona de forma opaca ante
cualquier configuración incorrecta: todo error, sea cual sea su causa, llega al
código como el mismo mensaje genérico de fallo de red.

Tres problemas distintos produjeron exactamente el mismo síntoma:

| Causa real | Lo que veía el cliente |
|---|---|
| El método de verificación previa exigía credenciales | Fallo de red |
| Las respuestas de error de la plataforma no llevaban encabezados CORS | Fallo de red |
| El alcance de la política de autorización no cubría todos los métodos | Fallo de red |

Diagnosticar eso obliga a razonar por eliminación, no por el mensaje de error.

---

## La solución

### Control de acceso en dos capas independientes

| Capa | Qué controla | Respuesta ante fallo |
|---|---|---|
| **Autorizador Lambda** | Quién solicita | `401` sin credenciales · `403` con credencial inválida |
| **Clave de API y plan de uso** | Cuánto puede solicitar | `403` con clave inválida · `429` al superar el límite |

Son independientes a propósito: la primera responde *quién eres*, la segunda
*cuánto puedes consumir*. Comprometer una no anula la otra.

### Decisiones de diseño

**El método de verificación previa queda sin autorización.** El navegador lo
emite sin credenciales por definición del estándar: todavía no sabe si el
servidor las aceptará. Exigirlas ahí impide toda comunicación desde el
navegador, aunque la API funcione perfectamente desde otros clientes.

**La política de autorización se emite con alcance de API y etapa.** El
resultado del autorizador se conserva en caché durante cinco minutos. Una
política limitada al método que originó la primera petición se reutilizaría
para las siguientes, denegando cualquiera dirigida a otro método.

**El importe de un pedido se calcula en el servidor.** Aceptar un total enviado
por el cliente permitiría a cualquiera fijar el precio de su propia compra.

**Las consultas de pedidos por cliente usan un índice secundario.** DynamoDB
factura por los datos que *lee*, no por los que devuelve: un recorrido completo
con filtro lee la tabla entera y descarta después.

```python
if usuario_id:
    # Query sobre el indice: lee solo la particion de ese usuario
    items = paginar(tabla.query,
                    IndexName=INDICE,
                    KeyConditionExpression=Key("usuarioId").eq(usuario_id))
else:
    # Sin filtro corresponde un scan: se piden todos los pedidos
    items = paginar(tabla.scan)
```

---

## Herramientas utilizadas

| Componente | Servicio | Función |
|---|---|---|
| Interfaz | Amazon S3 | Sitio estático servido por HTTPS |
| Exposición | API Gateway | API REST, autorizador, clave y plan de uso |
| Lógica | AWS Lambda | Tres funciones de dominio y una de autorización |
| Datos | DynamoDB | Tres tablas, con índice secundario sobre pedidos |
| Observabilidad | CloudWatch y SNS | Registros, métricas, alarma y notificación |

---

## Métricas de impacto

Todas medidas sobre el sistema en funcionamiento.

### Costo del arranque en frío

| Invocación | Duración | Facturado |
|---|---|---|
| Primera, con inicialización | 372 ms | **859 ms** |
| Siguiente, entorno reutilizado | 39 ms | **39 ms** |

Veintidós veces de diferencia. La inicialización se factura, de modo que el
arranque en frío no es solo un problema de latencia.

### Qué determina el costo de inicializar

| Función | Inicialización | Motivo |
|---|---|---|
| Autorizador | **83 – 89 ms** | Importa solo módulos de la biblioteca estándar |
| Funciones de dominio | **499 – 560 ms** | Importan el SDK y construyen el cliente de base de datos |

Seis veces de diferencia entre funciones del mismo proyecto. El costo depende
de **lo que la función carga**, no de lo que hace. Esto confirma la decisión de
declarar los clientes fuera del controlador: situados dentro, esos 450 ms
adicionales se pagarían en cada invocación y no solo en la primera.

### Efectividad del límite de consumo

| Configuración | Peticiones concurrentes | Resultado |
|---|---|---|
| 10 por segundo, ráfaga 20 | 40 | 40 aceptadas |
| 1 por segundo, ráfaga 1 | 60 | **48 rechazadas** con código 429 |

El algoritmo de cubo de fichas repone durante la propia ráfaga, por lo que el
límite efectivo es mayor que el nominal.

### Consulta dirigida frente a recorrido completo

La misma ruta declara en su respuesta qué operación empleó, lo que permite
verificar desde el cliente que el índice se está utilizando:

```json
{ "total": 1, "operacion": "query sobre usuarioId-index" }
{ "total": 3, "operacion": "scan completo" }
```

[captura: la aplicación mostrando ambas operaciones]

---

## Aprendizajes

**El mismo síntoma puede tener causas opuestas.** Una función invocada
directamente falla si añade encabezados CORS por su cuenta, porque la
plataforma ya los añade. La misma función tras una API Gateway falla si *no* los
añade, porque ahí la plataforma no lo hace. Idéntico mensaje de error,
soluciones contrarias.

**Endurecer sin comprender el protocolo degrada la disponibilidad.** Proteger
todos los métodos por igual dejó el sitio inutilizable, mientras la API seguía
respondiendo correctamente desde otros clientes. Un control mal aplicado hace
tanto daño como su ausencia.

**Un caché tiene consecuencias de seguridad.** Conservar la decisión de
autorización cinco minutos ahorra invocaciones, y a la vez implica que una
credencial revocada sigue siendo aceptada durante ese intervalo. Es un
intercambio legítimo, siempre que sea deliberado.

**En una base NoSQL el diseño se deriva de las consultas.** El índice
secundario no responde a una norma de modelado sino a un patrón de acceso
concreto. Y la ausencia de claves foráneas traslada la integridad referencial
al código, donde debe implementarse de forma explícita.

**En serverless el perímetro es la identidad, no la red.** La arquitectura no
tiene red virtual ni subredes, y no le hacen falta: ningún servicio empleado
vive dentro de una red del cliente. El control recae íntegramente sobre IAM.

---

## Habilidades técnicas aplicadas

`Diseño de APIs REST` · `Arquitectura serverless` · `Modelado NoSQL e índices secundarios`
`Autorización basada en políticas IAM` · `Limitación de consumo` · `CORS y seguridad en navegador`
`Observabilidad y alarmas` · `Python y boto3` · `Análisis de logs`

---

## Por qué elegí este proyecto

Porque es el único donde los problemas no venían escritos en el enunciado.

Los servicios individuales están bien documentados y funcionan como se espera.
Lo difícil apareció en las **interacciones entre ellos**: el caché del
autorizador con el alcance de la política, el navegador con los métodos
protegidos, la plataforma con los encabezados que añade o deja de añadir según
el tipo de integración. Nada de eso es evidente leyendo la documentación de cada
servicio por separado.

Resolverlo exigió diagnosticar por eliminación y entender el *porqué* de cada
comportamiento, no solo aplicar una solución encontrada. Es el trabajo que más
se parece al de arquitectura real, donde los sistemas rara vez fallan en sus
componentes y casi siempre en las costuras entre ellos.

---

## Ejecutar el proyecto

```
lambdas/
├── usuarios.py         GET · POST /usuarios
├── pedidos.py          GET · POST /pedidos  (consulta sobre índice)
├── productos.py        GET · POST · DELETE /productos
└── autorizador.py      valida el token y emite la política

sitio/
├── index.html          tres pestañas y panel de respuesta
├── styles.css
└── app.js
```

Las instrucciones completas de despliegue están en [`docs/despliegue.md`](<enlace>).

> Las credenciales no están incrustadas en el código: la aplicación las solicita
> al usuario y no las persiste. Incrustarlas las haría visibles para cualquiera
> que abriera el código fuente de la página.
