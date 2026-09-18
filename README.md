# aparrav.github.io

Portafolio personal de **Arnoldo Parra Villagrán** — ingeniería de datos y
arquitectura cloud.

**→ [aparrav.github.io](https://aparrav.github.io)**

---

## Qué contiene

El sitio es una sola página sin dependencias externas: no usa frameworks,
librerías ni CDN. Todo el CSS y el JavaScript viven en el propio `index.html`,
así que carga en una sola petición y funciona sin conexión una vez abierto.

| Sección | Contenido |
|---|---|
| Perfil | Recorrido desde la integración de sistemas hacia su diseño |
| Experiencia | Línea de tiempo de los cargos ocupados |
| Proyectos | Serverless Inteligente, Cloud Secure y MicroPay |
| Caso de estudio | Serverless Inteligente desarrollado en profundidad |
| Formación | Estudios, certificaciones e idiomas |

## El caso de estudio

La sección más extensa documenta una API de comercio electrónico sobre
arquitectura sin servidores: navegador → API Gateway → Lambda → DynamoDB,
con registros, métricas y una alarma que notifica por correo.

Está escrita con datos medidos, no con descripciones. Entre ellos:

- **859 ms → 39 ms** de tiempo facturado, según el entorno esté frío o reutilizado.
- **85 ms frente a 500 ms** de inicialización, según lo que importa cada función.
- **48 de 60** peticiones cortadas al superar el límite del plan de uso, antes de
  invocar cómputo.

El caso incluye el desafío, la solución, las herramientas, los aprendizajes y
una revisión honesta de un fallo de seguridad encontrado en el propio proyecto.

## Estructura del repositorio

```
index.html                        Sitio completo: estructura, estilos y scripts
README-perfil.md                  Borrador del README del perfil de GitHub
README-serverless-inteligente.md  Documentación técnica del proyecto
```

## Publicación

El repositorio se llama `aparrav.github.io`, así que GitHub Pages lo sirve
automáticamente desde la raíz de `main`. Cada `git push` actualiza el sitio en
uno o dos minutos; no hay proceso de compilación.

## Contacto

[arnoldo.parra@gmail.com](mailto:arnoldo.parra@gmail.com) ·
[LinkedIn](https://linkedin.com/in/arnoldo-parra-villagran-5723a034)
