# Protocolo multiagente — versión 1

## 1. Identidad y rutas

El alta de un proyecto se realiza con `/proyecto`; no crea una tarea ni un estado.
La inicialización no autoriza desarrollo ni escrituras en el destino.

La clave de una tarea es `(proyecto, id-tarea)`. El identificador es
`yyyyMMdd-slug-descriptivo`, con sufijo numérico si hay colisión. No contiene
separadores ni `..`. La fecha y el identificador no cambian al reanudar.
Una referencia externa repetida no identifica por sí sola una tarea.

Cada delegación debe incluir:

| Dato | Contenido |
|---|---|
| `raiz_orquestador` | Ruta absoluta de este espacio |
| `proyecto` / `perfil` | Identificador y ruta absoluta de `proyecto.md` |
| `raiz_codigo` | Ruta absoluta y canónica del código destino |
| `directorio_tarea` | Ruta absoluta bajo `proyectos/{proyecto}/tareas/` |
| Identidad | Id, título, fecha y referencia externa o `pendiente` |
| Operación | Fase, revisión de análisis, iteración de implementación |
| Entradas | Rutas exactas de estado, normativa y artefactos vigentes |
| Resultado esperado | Archivos exactos y criterios de terminación |
| Restricciones | Modo del código, alcance aprobado, feedback y bloqueos |

El subagente lee esas fuentes; no depende del contexto del chat padre.
Verifica que la raíz existe, corresponde al perfil y es distinta del directorio
de entregables. No cambies la Location de OpenCode al destino.

## 2. Artefactos

Todos se guardan bajo `directorio_tarea`. Los enlaces entre entregables son
relativos a ese directorio; las rutas de código se identifican como relativas a
`raiz_codigo`. Usa `flujo/plantillas/estado.md` para iniciar el estado.

| Responsable | Archivo |
|---|---|
| Orquestador | `estado.md` |
| Analista | `analisis.md` |
| Casos de uso | `casos-de-uso.md`, `plan-pruebas.md` |
| Desarrollador | `implementacion-{n}.md` |
| QA | `qa-{n}-r{revision}.md` |
| Documentador | `descripcion-tecnica.md` |

Los entregables mínimos de cada rol se definen en su agente. Todos son Markdown.
Textile es una extensión posterior, no una fase implícita. La exportación TestLink
(`testlink-exportador`) y la ejecución funcional con navegador
(`ejecutor-plan-pruebas`) existen como agentes independientes, fuera de las fases
y sin actualizar `estado.md`.

## 3. Estados y transiciones

```text
NUEVA → ANALISIS → PENDIENTE_APROBACION_ANALISIS
  rechazo → ANALISIS (revisión + feedback)
  aprobación → [CASOS_DE_USO opcional] → IMPLEMENTACION(n) → QA(n)
    NO APTO y n < límite → IMPLEMENTACION(n+1)
    NO APTO y n = límite → BLOQUEADA
    APTO → DOCUMENTACION → PENDIENTE_CIERRE → CERRADA
```

Cualquier fase puede quedar `BLOQUEADA`. Guarda motivo, fase de retorno y acción
necesaria. Solo reanuda tras resolver la causa; nunca conviertas un bloqueo de
infraestructura automáticamente en un defecto del código. El límite inicial
es 3 implementaciones; una ampliación requiere decisión humana registrada.

`CERRADA` significa aprobación humana de la entrega, no commit ni ticket cerrado.
El rechazo del cierre vuelve a la fase pertinente con el feedback registrado.
Un cambio de alcance vuelve a análisis y requiere nueva aprobación.

### Aprobaciones y versiones

- Incrementa `Revision analisis` al modificar `analisis.md`; conserva un historial
  de revisiones con el feedback. Registra qué revisión aprobó el usuario.
- Una revisión nueva invalida la aprobación anterior y los entregables derivados
  como evidencia vigente. Conserva los archivos y anota la invalidación.
- `n` empieza en 1 al lanzar la primera implementación. No lo incrementes por
  consultas de estado ni por interrupciones antes de completar la fase.
- Cada implementación completada es inmutable. Una nueva ejecución que cambia
  código usa la siguiente iteración; no sobrescribe su informe anterior.
- Cada repetición QA de la misma implementación incrementa `revision` y conserva
  informes anteriores. El estado apunta al informe vigente.
- Cambios de código posteriores al QA invalidan su veredicto para el cierre.

## 4. Reanudación y fases individuales

Antes de actuar, lee estado, perfil y artefactos vigentes. Confirma la identidad
del destino y compara el estado actual del código con la evidencia registrada.
Si cambió la ruta o el código de forma incompatible, registra un bloqueo y pide
resolverlo antes de continuar.

Registra la fase como en curso ANTES de delegar. Al finalizar, abre los archivos,
comprueba su contenido, identidad, enlaces y resultado; después actualiza estado,
historial, artefactos vigentes y siguiente paso. Una respuesta sin archivo no
completa una fase. Si el subagente se interrumpe, conserva lo parcial y comprueba
qué terminó antes de relanzarlo; no marques completado por mera existencia.

Los comandos de fase individual se detienen tras esa fase (análisis incluye su
aprobación; documentación incluye propuesta de cierre). Registran el siguiente
paso pendiente para que `/tarea` continúe sin repetir la fase completada.
`/tarea-estado` solo lee: no corrige ni reconstruye archivos automáticamente.
Ante incoherencia entre estado y entregables, informa antes de reparar.

## 5. Prerrequisitos

- Análisis: solicitud y perfil suficientes; puede ejecutarse en solo lectura.
- Casos de uso: análisis aprobado de la revisión vigente.
- Implementación: análisis aprobado, casos de uso si se solicitaron, perfil en
  `desarrollo` y permisos de escritura configurados. Si no, `BLOQUEADA`.
- QA: implementación completa vigente y ejecución de validaciones permitida.
- Documentación: QA vigente APTO y coherencia entre código y entregables.
- Cierre: documentación completa y aprobación humana registrada.

No saltes requisitos mediante comandos individuales ni invocaciones directas.
En modo `solo-lectura` puedes completar análisis y casos de uso; detente antes de
implementación. No ejecutes Maven ni tests: también escriben en el destino.

## 6. Evidencias y cambios preexistentes

Al iniciar guarda revisión base, salida de status y diff relevante de cambios
preexistentes en `estado.md` (sin secretos). Para SVN usa `svnversion`,
`svn status`, `svn diff`; para Git, `git rev-parse HEAD`, `git status --short`,
`git diff` y `git diff --cached`. Lee también archivos nuevos: los diffs de
versionado pueden omitirlos. Los comandos se ejecutan con `workdir=raiz_codigo`.

Cada informe de implementación registra las rutas afectadas y el diff relevante
o una descripción verificable de los archivos nuevos. QA contrasta con esa base;
no atribuye todos los cambios de la copia de trabajo a la tarea.

Una prueba ejecutada exige comando exacto, directorio, código de salida y extracto
literal del resultado. Distingue **Verificado**, **Declarado** y **No ejecutado**
(motivo). No inventes métricas de cobertura ni herramientas disponibles.
Una comprobación obligatoria pendiente impide QA APTO.

## 7. Resultado de subagentes

Solo escriben los entregables de su rol y tarea; no modifican `estado.md`.
Devuelven dudas bloqueantes en `## Preguntas abiertas`, junto al informe parcial
si es posible. Terminan con:

```text
RESULTADO: OK | BLOQUEADO | APTO | NO APTO
ARTEFACTOS: rutas absolutas de los archivos realmente guardados
RESUMEN: resultado, evidencias y siguiente paso en 3–5 líneas
```

## 8. Alcance de los controles

OpenCode aplica los permisos de herramientas. El orden de fases, la selección de
la tarea activa y las aprobaciones documentales dependen de las instrucciones del
orquestador. Los patrones de permisos de archivos cubren todos los proyectos
registrados, no se interpolan dinámicamente desde `proyecto.md`.
No hay scheduler, bloqueo entre sesiones ni máquina de estados ejecutable.
