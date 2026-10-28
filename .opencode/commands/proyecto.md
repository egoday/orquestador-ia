---
description: Inicializa un proyecto externo — id ruta [vcs] [modo]
agent: build
---

Inicializa el registro de un proyecto externo en este orquestador: $ARGUMENTS

Formato: `<id> <ruta> [vcs] [modo]`.

- `id`: identificador del proyecto, estable y seguro para carpeta.
- `ruta`: raíz de un proyecto **existente**, relativa a este orquestador o absoluta.
- `vcs`: `svn`, `git` o `ninguno`; si se omite, detéctalo.
- `modo`: `solo-lectura` (predeterminado) o `desarrollo`.

No crees, clones ni modifiques el proyecto destino. Este comando solo registra el
perfil y los archivos de trabajo dentro de `orquestador-ia`. No inicies una tarea,
no crees `estado.md`, no ejecutes builds, tests ni generadores y no hagas commits.

## 1. Validar la petición

1. Si faltan `id` o `ruta`, pregunta solo por los datos necesarios. No deduzcas el
   destino por la última tarea ni por el nombre de una carpeta cercana.
2. Normaliza `id` a minúsculas y al patrón `[a-z0-9][a-z0-9_-]*`. Rechaza `/`,
   `..`, espacios, puntos y separadores de ruta. Si el identificador ya existe en
   `proyectos/`, no lo reutilices ni lo sobrescribas: informa y detente.
3. Si `modo` no se indica, usa `solo-lectura`. Si se pide `desarrollo`, confirma
   expresamente que el usuario quiere ese modo y advierte de que las reglas de
   edición y shell de desarrollador/QA se habilitan aparte.
4. Resuelve `ruta` desde la raíz del orquestador. Comprueba que existe, es un
   directorio y no es la raíz del orquestador ni está dentro de `proyectos/`.
   Canonicaliza para validar, pero guarda en el perfil una ruta relativa a la raíz
   del orquestador, con separadores `/` y sin barra final. Si no puedes expresarla
   relativa, guarda la absoluta y explícalo en el resumen.
5. Si `vcs` no se indica, detecta `.git` o `.svn` en la raíz. Si hay ambos o
   ninguno, pregunta. No ejecutes comandos que escriban en el destino.

## 2. Inspeccionar el destino en solo lectura

Lee la raíz y, si existen, los documentos de normativa y manifiestos relevantes.
No ejecutes builds, tests, generadores ni scripts del proyecto.

- Detecta `.git`, `.svn`, `pom.xml`, `build.gradle`, `package.json`,
  `requirements.txt`, `pyproject.toml`, `go.mod`, `composer.json` u otros
  manifiestos evidentes.
- Busca `AGENTS.md`, `CONVENTIONS.md`, `GLOSSARY.md`, `DECISIONS.md`, `TESTING.md`,
  `STACK.md` y `.github/commit-instructions.md`.
- Registra solo lo observado. No inventes stack, módulos, comandos, versiones ni
  convenciones. Lo que no puedas confirmar queda como pendiente para el análisis.

## 3. Crear el perfil

Crea `proyectos/<id>/proyecto.md` con este contenido, adaptando los valores
detectados. Usa la fecha local (`date +%Y-%m-%d`), no una fecha inventada:

```markdown
---
id: <id>
raiz_codigo: <ruta-relativa>
vcs: <vcs>
modo_codigo: <modo>
---

# <título>

`raiz_codigo` es relativa a la raíz de `orquestador-ia`.
Este perfil se interpreta por los agentes; no configura permisos de OpenCode.

## Estado operativo

Proyecto registrado el <fecha>. Modo inicial `<modo>`.

<Texto de modo: en solo-lectura, no editar, mover ni borrar archivos del destino,
no modificar metadatos del VCS y no ejecutar builds, tests ni generadores. En
desarrollo, indicar que las reglas de edición y shell de desarrollador/QA deben
habilitarse aparte y que hasta entonces la implementación queda bloqueada.>

Los entregables se guardan en `tareas/`. La ejecución funcional con navegador, si
se autoriza, se realiza con el agente independiente `ejecutor-plan-pruebas` y no
modifica la copia.

## Normativa técnica (rutas relativas al código)

<Una lista de los documentos detectados, o una frase que indique que no se
detectaron documentos estándar y que el análisis debe reconstruir el contexto a
partir de README, manifiestos, código, tests, configuración y CI.>

La normativa técnica se consulta en su ubicación original; no se duplica aquí.
Las referencias del destino a agentes o artefactos de su flujo antiguo no cambian
este protocolo.

## Contexto

Ver `contexto/tecnico.md`. Es un resumen inicial; comprobar detalles en la
normativa y el código al realizar cada tarea.

## Validación prevista

<Pendiente de completar con comandos reales, directorio de trabajo, requisitos de
entorno y criterios de salida. En solo lectura no se ejecutan builds ni tests.>
```

No copies documentación del destino. No escribas rutas absolutas en el perfil si
puedes expresarlas relativas al orquestador.

## 4. Crear el contexto inicial y la carpeta de tareas

Crea `proyectos/<id>/contexto/tecnico.md`:

```markdown
# Contexto técnico inicial

Proyecto: <id>
Registrado: <fecha>
Fuente: inspección inicial de <ruta-relativa>; pendiente de completar.

- Stack: <detectado o pendiente de confirmar>.
- Módulos: pendiente de confirmar.
- Normativa: <documentos detectados o «no se detectaron documentos estándar»>.
- Validación: pendiente; no ejecutar en solo lectura.

Este resumen no sustituye a la normativa del destino. El análisis de la primera
tarea debe confirmar arquitectura, comandos y restricciones.
```

Crea `proyectos/<id>/tareas/README.md`:

```markdown
# Tareas

Todavía no se ha iniciado ninguna tarea con este orquestador.
Cada tarea tendrá su propia carpeta `yyyyMMdd-slug/` y un `estado.md`.
Los documentos del flujo original no se importan automáticamente.

Si se exporta el plan con `testlink-exportador`, la tarea puede contener
`testlink/`. Si se ejecuta con `ejecutor-plan-pruebas`, puede contener
`ejecucion-plan/`. Ninguna de esas acciones modifica el estado ni forma parte del
flujo de fases.
```

No crees una tarea, un estado ni una carpeta `yyyyMMdd-*` en este paso.

## 5. Permisos de OpenCode

Comprueba si `.opencode/opencode.jsonc` ya contiene reglas equivalentes para la
raíz. Si faltan, propone añadirlas y pide confirmación antes de editar la
configuración global del espacio:

```jsonc
{ "action": "external_directory", "resource": "<recurso>/*", "effect": "allow" }
```

En `solo-lectura`, añade además:

```jsonc
{ "action": "edit", "resource": "<recurso>/*", "effect": "deny" }
```

Calcula `<recurso>` de forma canónica. Si la raíz está bajo el home, exprésala como
`~/...`; si no, usa la ruta absoluta. Puedes usar `python3` con la biblioteca
estándar para calcularla o validar JSON/YAML, sin instalar dependencias ni escribir
en el destino. Añade la regla `external_directory` después de la regla genérica
`* ask` y antes de las reglas de shell; añade la regla `edit` junto a las demás
reglas de edición. No dupliques reglas existentes y conserva comentarios, orden y
formato del archivo. En `desarrollo`, no añadas la denegación de edición; advierte
de que las reglas de `desarrollador` y `qa` siguen pendientes.

Si el usuario rechaza el cambio de configuración, no lo hagas. Muestra las líneas
exactas que debería añadir y explica que el acceso externo puede solicitar
confirmación y que la protección de edición no quedará aplicada.

## 6. Validar y resumir

1. Reabre `proyecto.md` y comprueba frontmatter, `id`, `raiz_codigo`, `vcs` y
   `modo_codigo`; resuelve la raíz desde el orquestador y verifica que coincide con
   el destino canonicalizado.
2. Comprueba que existen `contexto/tecnico.md` y `tareas/README.md`.
3. Si modificaste la configuración, reabre el JSONC y verifica que las reglas están
   en el lugar correcto y sin duplicados. Si no puedes validar el JSONC con un
   parser local, dilo y no afirmes que la configuración es válida.
4. No ejecutes builds, tests ni comandos del destino. No hagas commit.
5. Resume:
   - proyecto e identificador;
   - ruta relativa registrada, VCS y modo;
   - documentos detectados y datos pendientes;
   - archivos creados;
   - reglas de permisos añadidas o pendientes;
   - siguiente paso: `/analisis <id> <descripción de la primera tarea>`.

Si algo falla, no dejes un perfil a medias ni inventes valores. Informa del error y
de lo que falta.
