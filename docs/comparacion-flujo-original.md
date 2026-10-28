# Comparación con el flujo de referencia

Referencia consultada el 2026-10-28:
`../arrhh_selpro-features-listas/.opencode/` y `.github/agents/`.
Esta es una adaptación, no una copia literal ni una migración del proyecto origen.

## Correspondencias

| Original | Este espacio |
|---|---|
| Orquestador específico ARRHH | Orquestador común + perfil por proyecto |
| Envoltorios que leen `.github/agents/` | Roles autocontenidos en `.opencode/agents/` |
| `.opencode/flujo/protocolo.md` | `flujo/protocolo.md` |
| `docs/tareas/<id>/estado.md` | `proyectos/<proyecto>/tareas/<id>/estado.md` |
| `docs/analisis-tecnico/...md` | `<directorio_tarea>/analisis.md` |
| `docs/casos-de-uso/` y `docs/planes-de-prueba/` | Dos archivos dentro de la tarea |
| `docs/tareas/<id>/implementacion-n.md` | `<directorio_tarea>/implementacion-n.md` |
| `docs/qa-reports/QA_...-itN.md` | `<directorio_tarea>/qa-N-rR.md` |
| `docs/descripcion-tecnica/...md` | `<directorio_tarea>/descripcion-tecnica.md` |
| Contexto Java 8/Maven/SVN en roles | Perfil ARRHH y normativa del código |

## Comportamientos conservados

- Análisis antes de implementar, con criterios de aceptación y estrategia de tests.
- Aprobación humana del análisis y opción de casos de uso/plan funcional.
- Desarrollo acotado al alcance aprobado; informes de implementación por iteración.
- QA independiente: diff real, pruebas reejecutadas y seis dimensiones de revisión.
- Límite inicial de tres implementaciones; bloqueo y decisión humana al agotarlo.
- Documentación basada en evidencias y aprobación del cierre.
- Prohibición de commits, despliegues y cierres externos automáticos.
- Reanudación basada en archivos y ejecución individual de fases.

## Ajustes explícitos

- Selección de proyecto obligatoria y contrato con dos raíces absolutas.
- Estado incluye solicitud original, siguiente paso, cambios preexistentes,
  revisión de análisis aprobada y vigencia de entregables derivados.
- Revisar el análisis invalida la aprobación anterior; repetir QA conserva el
  historial con número de revisión propio.
- QA puede devolver BLOQUEADO si faltan condiciones para verificar; no se deduce
  APTO por ausencia de hallazgos cuando faltan pruebas obligatorias.
- Los umbrales y comandos de calidad se toman del proyecto, no se generalizan
  porcentajes de cobertura de ARRHH a otros stacks.
- ARRHH empieza en solo lectura y los roles de implementación/QA están definidos
  pero pendientes de habilitar permisos de escritura y ejecución.

## Extensiones fuera del flujo inicial

El original también contiene exportación TestLink y un ejecutor de planes con
navegador, además de salidas Textile de análisis y documentación. Esta versión
inicial genera entregables Markdown del ciclo de desarrollo. Incorporar esas
extensiones requerirá adaptar sus contratos y rutas; no se cargan los agentes
originales de forma implícita.

## Cómo contrastar

Compara cada agente de aquí con el envoltorio del mismo rol en el original y su
definición correspondiente en `.github/agents/`. El desarrollador corresponde
a `fullstack-developer`, QA a `QA`. El documentador y el orquestador se definen
directamente en `.opencode/agents/` en ambos espacios.
No se han trasladado tareas históricas ni retirado archivos del proyecto origen.
