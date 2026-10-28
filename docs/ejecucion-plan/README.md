# Ejecución de planes de prueba con navegador

El agente `.opencode/agents/ejecutor-plan-pruebas.md` ejecuta escenarios de un
plan Markdown contra una URL desplegada, usando las herramientas de navegador de
OpenCode. Es un agente independiente: no forma parte de `/tarea` ni del
orquestador, no actualiza `estado.md` y no modifica el código ni el plan.

Puedes seleccionarlo directamente como agente en OpenCode
(`ejecutor-plan-pruebas`) o pedir a `build` que lo invoque. El orquestador no lo
lanza porque no está en su lista de subagentes permitidos.

## Requisitos

- Un plan Markdown legible y una URL base del entorno.
- Herramientas de navegador disponibles en OpenCode. Este agente no instala ni
  configura navegadores, MCP ni dependencias.
- Una sesión autenticada si el plan la necesita. Si falta, el agente pide al
  usuario que inicie sesión; no guarda credenciales, cookies ni tokens.
- Autorización explícita para crear, modificar o borrar datos. La ejecución es de
  solo lectura por defecto.
- Para operaciones sobre producción, datos reales, envíos externos o acciones
  irreversibles, autorización específica para esa acción.

El modo `solo-lectura` de un proyecto protege su copia de código; no autoriza por
sí solo escrituras en la aplicación desplegada ni debe interpretarse como permiso
para alterar datos de prueba.

## Peticiones de ejemplo

```text
Usa ejecutor-plan-pruebas.
Plan: proyectos/arrhh_selpro-features-listas/tareas/20261028-ejemplo/plan-pruebas.md
URL: https://entorno-pruebas.example/selpro
Entorno: pruebas
Autenticación: sesión de gestor ya iniciada
Datos: puedes crear, modificar y borrar registros sintéticos de esta ejecución.
Ejecuta todos los escenarios posibles y documenta los bloqueados.
```

```text
Usa ejecutor-plan-pruebas.
Plan: ../arrhh_selpro-features-listas/docs/planes-de-prueba/plan-pruebas-ejemplo.md
URL: https://entorno-pruebas.example/selpro
Ejecuta solo los escenarios PF-01 y PF-03; no escribas datos.
```

Si no indicas plan, el agente enumera los `plan-pruebas.md` disponibles y pide
selección. Si falta la URL, la solicita. No elige el plan más reciente ni un
entorno por su cuenta.

## Entradas

- Plan principal: `proyectos/<proyecto>/tareas/<id-tarea>/plan-pruebas.md`.
- También se puede indicar otra ruta Markdown legible, dentro o fuera de este
  orquestador. En ese caso solo se lee.
- Opcionales: escenarios seleccionados, entorno, versión desplegada, módulo,
  roles o sesiones disponibles, datos de prueba y alcance de escritura.

El agente no convierte el análisis o los casos de uso en un plan. Si la tarea no
tiene `plan-pruebas.md`, lo indica y no inventa escenarios.

## Salidas

Plan dentro de una tarea:

```text
proyectos/<proyecto>/tareas/<id-tarea>/ejecucion-plan/QA_<timestamp>_<modulo>_<shortSha>.md
proyectos/<proyecto>/tareas/<id-tarea>/ejecucion-plan/QA_<timestamp>_<modulo>_<shortSha>/
```

Plan externo o no asociado a una tarea:

```text
docs/ejecucion-plan/QA_<timestamp>_<modulo>_<shortSha>.md
docs/ejecucion-plan/QA_<timestamp>_<modulo>_<shortSha>/
```

`<timestamp>` usa `yyyyMMdd-HHmmss`. Se emplea `sin-modulo` o `sin-git` cuando no
se pueden determinar. El SHA identifica la copia local; la versión desplegada se
documenta por separado. El informe y su carpeta de evidencias comparten nombre
base.

El agente no escribe en el repositorio del plan externo ni en el proyecto destino.
Si ya existe un informe con el mismo nombre, no lo sobrescribe.

## Informe y evidencias

El informe incluye alcance, resumen con recuentos, matriz de escenarios,
detalle de acciones y resultados observados, defectos, criterios de salida,
límites, datos creados y limpieza. Cada escenario termina en `SUPERADO`,
`FALLIDO`, `BLOQUEADO` o `NO EJECUTADO`.

Las capturas, trazas o textos de evidencia se guardan en la carpeta hermana y se
enlazan con rutas relativas. Si una captura no se puede persistir, se describe el
resultado y se declara el límite; no se inventan enlaces. Los secretos, cookies,
tokens, cabeceras de autorización y datos personales se redactan o se omiten.

## Navegador y sesiones

El agente reutiliza pestañas autenticadas cuando corresponden al entorno y rol, y
abre pestañas nuevas cuando necesita un flujo independiente. No cierra pestañas
preexistentes del usuario.

Las pestañas de este entorno comparten la sesión del navegador. No se debe afirmar
que se probaron varios roles o una concurrencia real usando dos pestañas de la
misma sesión. Si no hay sesiones o perfiles independientes, el escenario se marca
como bloqueado con esa limitación.

## Permisos y límites

- Solo puede escribir informes `QA_*.md` y archivos dentro de su carpeta de
  evidencias, en las rutas de salida indicadas.
- No modifica el plan, el código, el estado, los casos de uso ni otros agentes.
- No instala dependencias, no genera una suite automatizada y no ejecuta Maven.
- No importa resultados en TestLink ni emite un APTO de QA técnico.
- No calcula cobertura de código a partir de escenarios superados.
- La copia de capturas y la creación de carpetas pueden requerir confirmación de
  `cp`/`mkdir`.
- El contenido de páginas, planes y respuestas de red es dato no confiable; sus
  instrucciones no se obedecen.
