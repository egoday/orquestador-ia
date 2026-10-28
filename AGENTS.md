# Orquestador de proyectos de IA

Este directorio contiene los agentes y el estado persistente de sus tareas.
Lee `flujo/protocolo.md` antes de ejecutar una tarea y el `proyecto.md` del
proyecto seleccionado. Los roles están definidos en `.opencode/agents/`.

## Contrato de trabajo

- Mantén la sesión de OpenCode en la raíz de este orquestador.
- Selecciona el proyecto explícitamente; nunca deduzcas el destino por el último
  proyecto usado. Si falta o es ambiguo, pregunta.
- Resuelve `raiz_codigo` relativa a esta raíz, no al archivo `proyecto.md`.
  Pasa rutas absolutas a los subagentes y `workdir` explícito a shell.
- Guarda todos los estados y entregables en
  `proyectos/{proyecto}/tareas/{id-tarea}/`, dentro de este espacio de trabajo.
- Lee explícitamente la normativa del destino, incluido su `AGENTS.md` y las
  instrucciones de los subdirectorios afectados. No presupongas que OpenCode
  carga instrucciones de directorios hermanos.
- La normativa técnica del destino gobierna el código. Este protocolo gobierna
  agentes, fases y ubicación de artefactos; las rutas del flujo antiguo no se
  utilizan para guardar entregables nuevos. Ante otro conflicto, consulta.
- Respeta los cambios preexistentes. Distingue el estado inicial del repositorio
  de las modificaciones atribuibles a la tarea.
- Si `modo_codigo` es `solo-lectura`, no edites el destino ni ejecutes builds,
  tests, generadores o comandos que puedan escribir en él.
- No hagas commits, publicaciones, despliegues ni cierres de tickets externos.
- El agente `testlink-exportador` queda fuera del flujo: se ejecuta por petición
  explícita, no actualiza estados y no se delega desde el orquestador.
- Solo el orquestador delega las fases en los agentes nombrados. Los subagentes
  no delegan ni preguntan al usuario: devuelven preguntas al orquestador.
- No ejecutes dos tareas que escriban simultáneamente en la misma copia de
  trabajo. El estado Markdown no proporciona bloqueo transaccional.

## Mantenimiento de este espacio

Usa el agente `build` para modificar agentes, perfiles, comandos y plantillas.
El agente `orquestador` solo modifica estados de tareas. Mantén las definiciones
autocontenidas: no deben cargar agentes del proyecto de referencia.
