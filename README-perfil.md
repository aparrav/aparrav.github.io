<!--
  README del PERFIL de GitHub.
  Para que aparezca como portada, el repositorio debe llamarse EXACTAMENTE
  igual que tu usuario. Ejemplo: si eres github.com/aparra, el repo es "aparra".

  Los textos entre <angulares> son los que debes reemplazar.
-->

# Hola, soy Arnoldo Parra

**Ingeniero de Datos · Azure y AWS · Arquitecturas cloud y seguridad**

Trabajo con datos en Azure y me expandí hacia arquitectura de soluciones en AWS.
Me interesa el punto donde ambas cosas se cruzan: cómo se diseña una plataforma
de datos que además sea segura, observable y sostenible en costo.

Lo que encontrarás aquí son proyectos construidos de principio a fin, con su
documentación, sus decisiones de diseño explicadas y, cuando corresponde,
los números que las respaldan.

---

## En qué trabajo

- **Ingeniería de datos** — pipelines, modelado y orquestación sobre Azure
- **Arquitectura cloud** — serverless, contenedores y diseño de APIs en AWS
- **Seguridad y cumplimiento** — control de accesos, auditoría y observabilidad

---

## Tecnologías

**Nube**
`AWS` · `Azure` · `Lambda` · `API Gateway` · `DynamoDB` · `S3` · `ECS / Fargate`
`CloudWatch` · `CloudTrail` · `AWS Config` · `Azure Data Factory` · `Databricks`

**Lenguajes y datos**
`Python` · `SQL` · `JavaScript` · `Spark` · `Delta Lake`

**Prácticas**
`Infraestructura como código` · `Mínimo privilegio` · `Observabilidad` · `Metodologías ágiles`

---

## Proyectos

| Proyecto | De qué trata | Stack |
|---|---|---|
| **[Serverless Inteligente](<enlace>)** | API REST de comercio electrónico con autorización en dos capas, sobre arquitectura sin servidores. Incluye caso de estudio con métricas. | Lambda · API Gateway · DynamoDB · CloudWatch |
| **[Cloud Secure](<enlace>)** | Controles de seguridad, auditoría y cumplimiento automatizado sobre una cuenta de AWS. | CloudTrail · Config · CloudWatch · SNS · S3 |
| **[MicroPay en contenedores](<enlace>)** | Microservicios sobre ECS con Fargate, y optimización de costos del clúster. | ECS · Fargate · ALB · CloudWatch |
| **[Azure ↔ AWS](<enlace>)** | Equivalencias entre ambas nubes para plataformas de datos, razonadas desde la práctica. | — |

---

## Un par de cosas que aprendí construyendo esto

- El arranque en frío de una función no depende de lo que hace, sino de **lo que
  carga**. Una función que solo importa la biblioteca estándar inicializa en 85 ms;
  la misma función importando el SDK de AWS tarda 500 ms.
- En una base NoSQL se paga por lo que se **lee**, no por lo que se devuelve. Un
  índice secundario no es una optimización opcional: cambia el costo de la consulta.
- Un servicio de AWS no puede usar un rol por tener los permisos adecuados. El rol
  debe además **declarar que confía en ese servicio**. Son dos condiciones distintas.

---

## Contacto

- **LinkedIn** — <enlace a tu perfil>
- **Correo** — <tu correo>

<!--
  Sugerencias opcionales, si quieres enriquecerlo despues:
  - Insignias de estadisticas de GitHub
  - Una seccion "Escribiendo" si publicas articulos
  - Certificaciones, cuando las obtengas
-->
