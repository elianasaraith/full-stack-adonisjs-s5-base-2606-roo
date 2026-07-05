# Auditoría de documentación existente — Sesión 5

## Contexto

Esta auditoría se realiza sobre mi copia del template `full-stack-adonisjs-s5-base`, utilizado como repositorio base para la Sesión 5 del máster AI4Devs. El proyecto corresponde a un monorepo con backend en AdonisJS 7 y frontend en React 19, tomando como contexto funcional el producto FlowSync descrito en `docs/PRD.md`.

El objetivo de este ejercicio es revisar, por una parte, el backlog generado en la Sesión 4 a partir del PRD de FlowSync y, por otra, auditar el estado real de la documentación existente en el repositorio. La intención no es perfeccionar la documentación, sino identificar qué existe, qué falta, qué está incompleto y qué debería priorizarse en una mejora posterior.

---

## Parte A — Revisión del backlog de historias de usuario S4

### 1. ¿Siguen teniendo sentido las historias generadas?

En general, sí. Las historias generadas en la Sesión 4 siguen teniendo sentido porque se mantienen alineadas con los módulos principales del PRD: autenticación, gestión de tareas, organización y filtrado, exportación y sincronización con Google Calendar. La agrupación por épicas ayudó a cubrir el alcance completo del MVP y a separar funcionalidades base de la funcionalidad diferenciadora del producto.

Sin embargo, al revisarlas después del ejercicio de *poke-holes* y con una mirada más cercana a refinamiento, veo que algunas historias estaban demasiado enfocadas en el flujo feliz. Funcionan como backlog inicial, pero no todas estarían listas para desarrollo sin una revisión adicional de supuestos, casos límite y criterios verificables.

### 2. ¿El alcance sigue ceñido al MVP o se coló algo fuera de scope?

El alcance quedó bastante ceñido al MVP. No se generaron historias sobre equipos, subtareas, calendarios distintos de Google, aplicación móvil nativa, notificaciones push/email ni proyectos o etiquetas, que el PRD declara explícitamente fuera de alcance.

El punto que requiere más cuidado es la sincronización con Google Calendar. El PRD indica que la sincronización inversa completa requiere un spike técnico antes de comprometerla, por lo que cualquier historia que sugiera cambios desde Google Calendar hacia FlowSync debería quedar fuera del backlog implementable o tratarse como investigación, no como funcionalidad cerrada del MVP.

### 3. ¿Hay criterios de aceptación incompletos o poco verificables?

Sí. Al revisar el backlog con más detalle, veo que varias historias tienen criterios válidos para el flujo principal, pero incompletos para un refinamiento real.

En **US-1.2 — Inicio de sesión**, los criterios cubren credenciales correctas, credenciales incorrectas y sesión con token válido. Sin embargo, no especifican qué ocurre con un token expirado, revocado o malformado. Para QA sería difícil validar completamente el comportamiento de sesión solo con esos criterios.

En **US-2.2 — Ver el listado de tareas**, el criterio “las más relevantes para hoy aparecen primero” es poco verificable porque el PRD deja abierto qué significa “relevante”. No se define si se prioriza por fecha límite, tareas vencidas, tareas del día, estado `pending` o alguna combinación. Antes de desarrollo habría que convertirlo en una regla concreta o tratarlo como decisión pendiente de refinamiento.

En **US-2.5 — Cambiar el estado de una tarea**, el criterio de “reabrir” una tarea completada a `pending` aparece marcado como asumido. El PRD indica que el usuario puede cambiar el estado, pero no detalla si todos los cambios entre estados son válidos ni si existen restricciones. Esto puede generar interpretaciones distintas entre frontend, backend y QA.

En **US-4.1 — Exportar tareas a CSV**, los criterios cubren la generación del archivo, pero dejan asumido si el CSV debe incluir todas las tareas o solo las visibles según filtros. Tampoco se especifica formato de fecha, orden de columnas, codificación o comportamiento ante caracteres especiales. Para una exportación, esos detalles son relevantes porque afectan el uso real del archivo fuera de FlowSync.

En **US-5.1 — Conectar la cuenta de Google**, los criterios cubren el flujo feliz de OAuth y el rechazo/cancelación, pero no contemplan escenarios como token revocado después de conectar, permisos OAuth parciales o expiración de autorización. La historia podría pasar QA inicialmente y aun así fallar en producción cuando la conexión quede marcada como activa pero ya no funcione.

En **US-5.2 — Reflejar tareas con fecha límite como eventos en Google Calendar**, el criterio indica que se crea un evento al crear una tarea con fecha límite, pero no define reglas de zona horaria, calendario destino, hora del evento, ni qué ocurre con tareas ya existentes antes de conectar Google. Esos vacíos afectan directamente la experiencia principal del MVP.

### 4. ¿Hay historias que cambiaron de naturaleza?

Sí. La épica de sincronización con Google Calendar fue la que más cambió de naturaleza al revisarla. En el backlog inicial aparecía como un conjunto de historias funcionales implementables, pero después del análisis la veo como un bloque de alto riesgo que necesita separar funcionalidad, reglas de negocio y validaciones técnicas.

Por ejemplo, **US-5.1 — Conectar la cuenta de Google** no debería quedarse solo como “conectar OAuth”. También requiere definir cómo se valida que la conexión siga siendo usable en el tiempo y qué ocurre si los permisos se revocan o quedan incompletos.

La **US-5.2 — Reflejar tareas con fecha límite como eventos en Google Calendar** también cambió de naturaleza. Inicialmente parecía una historia directa de creación de eventos, pero ahora veo que depende de decisiones previas sobre zona horaria, calendario destino, tareas preexistentes y representación de una fecha límite como evento. Sin esas definiciones, el equipo podría implementar comportamientos distintos y todos podrían parecer “correctos”.

También la **US-2.2 — Ver el listado de tareas** necesita refinamiento. El criterio de orden por relevancia para “hoy” no debería tratarse como una historia cerrada hasta definir la regla exacta de ordenamiento. En su estado actual, es más una intención de producto que un criterio verificable.

### 5. ¿Hay historias nuevas que deberían aparecer?

Sí. No necesariamente como funcionalidades nuevas fuera del MVP, sino como historias o spikes derivados de ambigüedades del PRD:

- **Spike — Definir regla de ordenamiento por relevancia para hoy.**  
  Deriva de **US-2.2**. Antes de implementar el listado, habría que definir si la prioridad depende de tareas vencidas, tareas con fecha límite hoy, estado `pending`, fecha de creación u otro criterio.

- **Spike — Definir reglas de fecha, hora y zona horaria para eventos de Google Calendar.**  
  Deriva de **US-5.2**. El PRD menciona zonas horarias como riesgo explícito, pero el backlog no define cómo convertir una fecha límite en un evento correcto.

- **Historia o criterio adicional — Sincronizar tareas existentes al conectar Google Calendar.**  
  Deriva de **US-5.1** y **US-5.2**. El backlog no define si las tareas con fecha límite creadas antes de conectar Google deben generar eventos retroactivamente.

- **Historia o criterio adicional — Detectar conexión OAuth no funcional después de una conexión exitosa.**  
  Deriva de **US-5.1** y **US-5.5**. El backlog cubre fallos puntuales de API, pero no el caso en que la cuenta siga marcada como conectada aunque el token haya sido revocado, expirado o no tenga permisos suficientes.

- **Criterios adicionales — Definir alcance de exportación CSV.**  
  Deriva de **US-4.1**. Se debería aclarar si exporta todas las tareas, solo las filtradas, cómo se representan fechas vacías, caracteres especiales y estados.

### 6. Comparación con el backlog trabajado en el directo de S4

Mi backlog priorizó una descomposición funcional por módulos del PRD. Eso ayudó a cubrir el alcance completo del MVP, pero probablemente dejó la sincronización con Google Calendar demasiado agrupada como si fuera una funcionalidad lineal.

Comparándolo con el enfoque trabajado en el directo de S4, hoy priorizaría antes los riesgos y dependencias críticas: OAuth, permisos, zonas horarias, confiabilidad de sincronización y reglas pendientes. Creo que mi backlog acertó en cobertura del MVP, pero necesitaba más separación entre historias implementables, spikes técnicos y decisiones de producto aún no resueltas.

### 7. Ajustes concretos al backlog

#### Ajuste 1

- **Ajuste:** Separar la épica de Google Calendar en historias más pequeñas y criterios adicionales: conexión OAuth (**US-5.1**), validación de conexión usable en el tiempo, creación de eventos desde tareas con fecha límite (**US-5.2**), actualización de eventos, desconexión (**US-5.4**) y manejo de errores/reintentos (**US-5.5**).
- **Motivo:** En S4 quedó claro que la sincronización concentra el mayor riesgo técnico y funcional. Si se deja como historias demasiado amplias, es más difícil validar alcance, probar casos límite y detectar dependencias como tokens revocados, permisos parciales, zonas horarias y tareas preexistentes.

#### Ajuste 2

- **Ajuste:** Refinar **US-2.2 — Ver el listado de tareas** para definir qué significa “relevante para hoy” antes de implementarla.
- **Motivo:** El criterio actual no es suficientemente verificable. Desarrollo podría ordenar por fecha límite, QA podría esperar tareas vencidas primero y producto podría esperar pendientes del día. Sin una regla explícita, el mismo criterio permite múltiples interpretaciones.

#### Ajuste 3

- **Ajuste:** Refinar **US-4.1 — Exportar tareas a CSV** para definir alcance y formato del archivo: si exporta todas las tareas o solo las filtradas, orden de columnas, formato de fecha, manejo de campos vacíos y caracteres especiales.
- **Motivo:** La exportación parece simple, pero en un entorno laboral estos detalles afectan directamente la utilidad del archivo y las pruebas de QA. El backlog actual deja varias decisiones como asumidas.

#### Ajuste 4

- **Ajuste:** Agregar criterios a **US-5.1** y **US-5.2** para definir qué ocurre con las tareas existentes con fecha límite cuando el usuario conecta Google Calendar por primera vez.
- **Motivo:** El PRD no especifica si deben sincronizarse retroactivamente o solo desde nuevas tareas. Es una regla de negocio importante para la experiencia inicial del usuario y para validar la propuesta de valor del MVP.

---

## Parte B — Auditoría de documentación del proyecto

| Tipo de documentación | Estado | Ubicación | Observación |
|---|---|---|---|
| README de proyecto | Completa | `README.md` | Permite arrancar el proyecto desde cero: describe el monorepo, requisitos, instalación de backend y frontend, configuración de `.env`, generación de `APP_KEY`, ejecución de migraciones y comandos para levantar ambos servicios. También incluye una tabla básica de endpoints. |
| Descripción de arquitectura general | Parcial o pobre | `README.md`, `CLAUDE.md`, `docs/README.md` | Existe una descripción básica del monorepo, stack y separación entre `backend`, `frontend`, `docs` y `openspec`. Sin embargo, no hay un documento de arquitectura que explique componentes, responsabilidades, flujos principales, relación backend/frontend, autenticación, persistencia o límites del sistema. El `docs/README.md` menciona que ADRs y diagramas se añadirán en sesiones posteriores, pero aún no existen. |
| Documentación de API o endpoints | Parcial o pobre | `README.md`, `backend/start/routes.ts` | El README lista los endpoints principales y su autenticación, pero no documenta contratos completos: request body, response body detallado, códigos de error, headers, validaciones ni ejemplos. Para integrarse correctamente todavía sería necesario leer rutas, controllers o probar manualmente. |
| Docstrings y comentarios significativos en código | Parcial o pobre | `backend/app`, `frontend/src` | Existen comentarios breves en controllers, middlewares, validators y transformers. Ayudan a entender la intención básica de endpoints como login, registro, profile y users, pero son descripciones mínimas. No hay TSDoc/JSDoc suficientemente completo para reglas de negocio, contratos, errores esperados o decisiones no evidentes. |
| Decisiones técnicas registradas | Inexistente | No se encontraron archivos ADR ni documentos de decisiones (`find . -iname "*adr*" -o -iname "*decision*" -o -iname "*decisiones*"` no devolvió resultados). | El stack está declarado en README y `CLAUDE.md`, pero no existen ADRs o equivalentes que expliquen por qué se eligieron AdonisJS, SQLite, access tokens, React/Vite u OpenSpec, ni qué alternativas se descartaron. |
| Guía operacional | Parcial o pobre | `README.md` | Hay instrucciones para entorno local, pero no existe guía de despliegue, troubleshooting, runbooks de incidencias, manejo de logs, recuperación de errores comunes, Docker/PM2 o consideraciones para operar el proyecto fuera del entorno de desarrollo. La búsqueda de términos operacionales no arrojó documentación útil más allá de referencias indirectas en dependencias. |
| Convenciones de código del proyecto | Parcial o pobre | `CLAUDE.md`, `backend/.editorconfig`, `backend/eslint.config.js` | `CLAUDE.md` sí define convenciones relevantes: lógica en controllers, validación con VineJS, salida vía `UserTransformer`, rutas bajo `/api/v1`, uso de middleware de auth e imports con subpath imports. También existen configuraciones técnicas en backend. Aun así, falta una guía más completa para humanos sobre naming, estructura por capas, manejo de errores, testing y criterios para agregar nuevas funcionalidades. |
| Especificación OpenSpec y trazabilidad con el código | Parcial o pobre | `openspec/config.yaml`, `openspec/specs/authentication/spec.md`, `openspec/specs/users/spec.md`, `backend/start/routes.ts` | El repo incluye configuración OpenSpec y specs para autenticación y usuarios. Las rutas reales encontradas en `backend/start/routes.ts` coinciden parcialmente con el alcance documentado en README: health, register, login, logout, profile, users y users/:id. Sin embargo, todavía no se evidencia una trazabilidad completa entre specs, cambios aplicados, criterios de aceptación y código actual. Además, `GET /api/v1/users/active` está mencionado como endpoint de demo en README/código, pero no está implementado en las rutas actuales. |
| Documentación adicional detectada: memoria para copilotos de IA | Parcial o pobre | `CLAUDE.md`, `.claude/`, `.cursor/` | La memoria del proyecto ayuda a Claude Code o Cursor a entender contexto, stack, comandos, convenciones y cosas que no debe hacer. Es muy útil para trabajo asistido por IA, pero no reemplaza documentación de arquitectura, API u operación pensada para el equipo humano. |

---

## Top 3 carencias que más duelen

1. **Falta documentación de API completa.**  
   Aunque el README lista endpoints, no hay contratos detallados con payloads, respuestas, errores y ejemplos. Esto dificulta que frontend, QA u otro consumidor puedan integrarse sin leer código o hacer pruebas manuales.

2. **No hay documentación de arquitectura general.**  
   El repo explica el stack y la estructura, pero no muestra cómo se relacionan backend, frontend, autenticación, base de datos y OpenSpec. Esto aumenta la curva de entrada para alguien nuevo y limita el uso efectivo de copilotos de IA.

3. **No existen decisiones técnicas registradas como ADRs.**  
   El proyecto declara tecnologías, pero no explica por qué se eligieron ni qué compromisos implican. En un entorno laboral, esto complica mantener coherencia cuando el equipo crece o cuando se quiere cambiar una decisión.

---

## Top 3 cosas que ya están bien

1. **El README permite levantar el proyecto localmente.**  
   Incluye requisitos, comandos de instalación, configuración de `.env`, generación de clave, migraciones y arranque de backend y frontend. Es suficiente para que alguien nuevo pueda comenzar sin demasiada fricción.

2. **La estructura del monorepo es clara.**  
   La separación entre `backend`, `frontend`, `docs`, `openspec`, `.claude` y `.cursor` facilita entender dónde vive cada parte del sistema y cómo se relaciona con el flujo del máster.

3. **Existe memoria de proyecto para copilotos de IA.**  
   `CLAUDE.md` entrega contexto útil sobre stack, comandos, convenciones y restricciones. Esto es una buena base para que herramientas como Claude Code o Cursor generen cambios más alineados con el proyecto.

---

## Exploración rápida de formatos de documentación

### C4

Un diagrama C4 aporta una vista visual por niveles del sistema, desde el contexto general hasta contenedores y componentes, permitiendo entender rápidamente cómo se relacionan las partes sin leer toda la base de código.

### ADR

Un ADR aporta trazabilidad sobre decisiones técnicas importantes: qué se decidió, por qué, qué alternativas se consideraron y cuáles son las consecuencias. Esto evita que el conocimiento quede solo en conversaciones o memoria del equipo.

### OpenAPI

Una especificación OpenAPI aporta un contrato formal y consumible de la API, incluyendo rutas, métodos, payloads, respuestas y errores. Es más precisa que una tabla en Markdown y permite generar documentación, clientes o pruebas automáticamente.
