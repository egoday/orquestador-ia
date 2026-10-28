---
id: arrhh_selpro-features-listas
raiz_codigo: ../arrhh_selpro-features-listas
vcs: svn
modo_codigo: solo-lectura
---

# ARRHH SELPRO — features listas

`raiz_codigo` es relativa a la raíz de `orquestador-ia`.
Este perfil se interpreta por los agentes; no configura permisos de OpenCode.

## Estado operativo

Registrado para consulta y comparación con el flujo original. Por petición del
usuario, el destino se mantiene intacto: no editar, mover ni borrar sus archivos,
no modificar propiedades SVN y no ejecutar builds, tests ni generadores.
Los análisis y otros entregables autorizados se guardan aquí, en `tareas/`.
El paso a `desarrollo` requiere una petición posterior explícita y ajustar los
permisos descritos en `README.md` del orquestador.

## Normativa técnica (rutas relativas al código)

Leer `CONVENTIONS.md`, `AGENTS.md`, `GLOSSARY.md`, `DECISIONS.md`, `TESTING.md`
y `STACK.md`. `CONVENTIONS.md` prevalece en reglas técnicas del proyecto.
Consultar `.github/commit-instructions.md` si existe al proponer texto de commit.
No cargar las definiciones antiguas de agentes para ejecutar este flujo.

La normativa técnica sigue siendo consultada en su ubicación original; no se
duplica aquí. Sus referencias a agentes o artefactos en `docs/` pertenecen al
flujo original; los artefactos nuevos siguen `flujo/protocolo.md`.

## Contexto

Ver `contexto/tecnico.md`. Es un resumen inicial; comprobar detalles en la
normativa y el código al realizar cada tarea.

## Validación prevista (solo al habilitar desarrollo)

Con directorio de trabajo `raiz_codigo` y JDK 8 confirmado en el entorno:

- Cambio acotado: `mvn -pl <modulo> test`.
- Impacto de integración: `mvn -pl <modulo> verify`.
- WSDL: `mvn -pl arrhh_selpro-integration generate-sources test`.
- JaCoCo y Checkstyle solo si están configurados para el módulo.

No fijar una ruta personal a JAVA_HOME sin comprobarla. Usar el stack y los
criterios de testing documentados en el destino; no asumir métricas disponibles.
