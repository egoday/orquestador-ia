# Orquestador de agentes de IA

Espacio de trabajo OpenCode V2 para desarrollar tareas en proyectos externos y
conservar aquí los agentes, el flujo, los estados y los entregables Markdown.

## Primer uso

Abre OpenCode en este directorio y selecciona `orquestador` (predeterminado en
sesiones nuevas). En una sesión existente, selecciónalo explícitamente.

```text
/tarea-estado
/analisis arrhh_selpro-features-listas Descripción concreta de una tarea
/tarea arrhh_selpro-features-listas 20261028-ejemplo
```

El último identificador es ilustrativo: usa el creado por el análisis.
El primer argumento identifica el proyecto, incluso si solo hay uno registrado.

**Situación inicial:** ARRHH está en `solo-lectura`, para comparar esta propuesta
con el flujo original. Puedes generar análisis y casos de uso aquí. Implementación
y validación ejecutable quedarán bloqueadas hasta habilitar el destino por petición
expresa. No se han iniciado tareas ni ejecutado agentes sobre el código destino.

## Organización

```text
.opencode/agents/                         Agentes del flujo e independientes
.opencode/commands/                       Comandos del flujo
.opencode/opencode.jsonc                  Configuración y accesos locales
AGENTS.md                                Contrato general del espacio
flujo/protocolo.md                        Fases, evidencias y reanudación
flujo/plantillas/estado.md                 Plantilla de cada tarea
flujo/plantillas/proyecto.md               Plantilla para registrar proyectos
proyectos/<proyecto>/proyecto.md           Ruta, modo y normativa del destino
proyectos/<proyecto>/contexto/             Particularidades técnicas
proyectos/<proyecto>/tareas/<id-tarea>/    Estado y entregables por tarea
docs/testlink/README.md                    Contrato del exportador independiente
docs/ejecucion-plan/README.md              Contrato del ejecutor independiente
docs/comparacion-flujo-original.md         Correspondencias con la referencia
```

Los archivos nuevos de coordinación quedan fuera de la copia SVN del destino.
La normativa técnica existente se consulta allí por referencia; las definiciones
de roles y del flujo nuevo son independientes y están completas en este espacio.

## Comandos

| Comando | Argumentos | Alcance |
|---|---|---|
| `/proyecto` | id ruta [vcs] [modo] | Alta de proyecto externo |
| `/tarea` | proyecto descripción-o-id | Flujo completo o reanudación |
| `/tarea-estado` | [proyecto] [id] | Consulta sin modificaciones |
| `/analisis` | proyecto descripción-o-id [feedback] | Análisis y aprobación |
| `/casos-de-uso` | proyecto id | Casos de uso y plan funcional |
| `/implementar` | proyecto id | Implementación o corrección |
| `/qa` | proyecto id | Validación independiente |
| `/documentar` | proyecto id | Descripción técnica y aprobación de cierre |

El flujo mantiene aprobación humana del análisis y del cierre, así como un máximo
inicial de tres implementaciones con QA. Los comandos individuales comprueban los
mismos prerrequisitos y guardan el siguiente paso para reanudar con `/tarea`.
`/proyecto` es un alta de mantenimiento: crea el perfil y el contexto inicial, pero
no crea tareas ni ejecuta fases.

## Agentes independientes

### TestLink

`testlink-exportador` convierte un `plan-pruebas.md` en suites y casos XML para
TestLink. No pertenece al flujo: no se lanza desde `/tarea`, no actualiza el estado
y no ejecuta pruebas ni importa el XML.

Selecciónalo directamente como agente en OpenCode o pide a `build` que lo invoque.
El contrato, las peticiones de ejemplo y la importación se documentan en
`docs/testlink/README.md`. Las salidas van a la carpeta `testlink/` de la tarea o,
si el plan es externo, a `docs/testlink/`; nunca al proyecto destino.

### Ejecución de planes con navegador

`ejecutor-plan-pruebas` ejecuta escenarios de un plan contra una URL desplegada
con las herramientas de navegador. Tampoco pertenece al flujo: no se lanza desde
`/tarea`, no actualiza el estado y no modifica código ni planes.

Selecciónalo directamente o pide a `build` que lo invoque. Requiere URL, plan y
autorización explícita para escribir datos; por defecto solo consulta. El informe
y las evidencias se guardan en `ejecucion-plan/` dentro de la tarea o en
`docs/ejecucion-plan/` si el plan es externo. El contrato y los límites se
documentan en `docs/ejecucion-plan/README.md`.

## Dos raíces

- `raiz_orquestador`: configuración, perfiles y entregables.
- `raiz_codigo`: código, pruebas y comandos del repositorio destino.

`raiz_codigo` se resuelve respecto a la raíz de este orquestador. Las delegaciones
usan rutas absolutas y shell recibe `workdir` explícito. No hace falta copiar
agentes, crear enlaces simbólicos ni instalar configuración en los otros proyectos.

## Registrar otro proyecto

La vía recomendada es `/proyecto <id> <ruta> [vcs] [modo]`. Crea el perfil, el
contexto inicial y `tareas/README.md`, valida que la ruta exista y propone las
reglas de acceso. No crea ni modifica el proyecto destino. El modo predeterminado
es `solo-lectura`.

Si prefieres revisar cada paso manualmente, usa `build` para mantener este espacio:
Este agente puede solicitar ejecutar Python para validar los archivos locales; los
agentes del flujo conservan shell limitado a consultas.

1. Copia `flujo/plantillas/proyecto.md` a `proyectos/<id>/proyecto.md`.
2. Define ruta, VCS, normativa, contexto y comandos reales de validación.
3. Crea `contexto/` y `tareas/` cuando tengan contenido. Empieza en solo lectura.
4. Añade acceso `external_directory` a su raíz canónica en la configuración.
   Puedes escribirla relativa al home (`~/...` o `$HOME/...`): OpenCode expande
   ambas formas. No se interpola desde el perfil.
5. Para mantenerlo en consulta, añade las reglas de denegación de edición como
   las del perfil ARRHH. Verifica que la ruta resuelta corresponde al destino.
6. Prueba primero un análisis y la consulta de su estado.

Los roles no incluyen Java, Maven ni módulos de ARRHH: esas decisiones pertenecen
al perfil y su normativa. No importes tareas de otro proyecto por coincidencia
de identificador o ticket.

## Habilitar desarrollo posteriormente

Este es un cambio de configuración deliberado, no un efecto de aprobar un análisis.
Cuando el usuario autorice trabajar en un destino:

1. Cambia su `modo_codigo` a `desarrollo`.
2. Retira únicamente las denegaciones de edición de esa raíz.
3. En `.opencode/agents/desarrollador.md`, después del `edit: * → deny`, añade
   reglas `edit → allow` para los directorios de código autorizados, usando rutas
   `~/...` o absolutas. Añade después exclusiones de generados y metadatos del VCS.
4. Añade permisos shell específicos a desarrollador y QA para los comandos de
   compilación/tests del perfil. Actualmente solo se permiten consultas de VCS y
   fecha. Mantén bloqueados commits, publicaciones y despliegues. QA conserva
   denegada la edición directa de código; ejecutar tests sí puede generar archivos.
5. Valida la configuración y realiza una tarea pequeña, comprobando aprobación,
   implementación, QA, documentación y reanudación antes de ampliar el uso.

Los permisos de ARRHH usan `~/code/rrhh/arrhh_selpro-features-listas/*`, relativa
al home. Sigue suponiendo esa disposición de carpetas: si mueves el proyecto,
actualiza esas reglas además de la ruta del perfil.

## Límites actuales

- La coordinación y las transiciones las ejecuta un LLM mediante instrucciones;
  no hay motor determinista, scheduler ni bloqueo de tareas entre sesiones.
- Los permisos de escritura de artefactos son por rol sobre los proyectos, no
  por tarea activa; el protocolo exige respetar la identidad recibida.
- Los permisos shell no son un sandbox. La configuración inicial restringe shell
  a consultas; habilitar comandos de build implica permitir sus efectos.
- El estado es Markdown y requiere conservar sus campos al editarlo manualmente.
- La exportación TestLink y la ejecución de planes con navegador existen como
  agentes independientes, fuera del flujo. Textile sigue sin implementarse.
- Las pestañas del navegador comparten sesión; el ejecutor no puede acreditar
  concurrencia entre roles distintos sin sesiones independientes.

## Validación

Configuración escrita con la sintaxis documentada de OpenCode V2. La comprobación
estática de archivos y referencias no sustituye a una tarea real con agentes.
Validación inicial realizada: parseo JSON y YAML con Python/PyYAML, seis agentes
del flujo y dos agentes independientes (TestLink y ejecución de planes), ocho
comandos con agente existente, perfil con ruta válida y regla de protección de
edición correspondiente. Resultado: correcto. No se ha ejecutado el ciclo
funcional, ni el agente de navegador contra un entorno real, ni se ha validado un
build del proyecto destino.
La primera ejecución funcional recomendada es `/analisis` sobre una solicitud
concreta, seguida de `/tarea-estado` en otra sesión para comprobar reanudación.
