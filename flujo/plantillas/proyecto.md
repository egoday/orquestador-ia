---
id: nombre-proyecto
raiz_codigo: ../nombre-proyecto
vcs: svn
modo_codigo: solo-lectura
---

# Nombre del proyecto

Plantilla: sustituir los valores antes de registrar el perfil.
`raiz_codigo` se resuelve desde la raíz del orquestador. `vcs` puede ser `svn`,
`git` o `ninguno`; para este último, documentar cómo identificar la base y los
cambios sin inventar una revisión de VCS.

## Normativa técnica

- Rutas relativas al código destino y orden de precedencia.
- AGENTS.md e instrucciones aplicables en subdirectorios, si existen.

## Contexto

- Arquitectura, stack y versiones compatibles.
- Módulos y restricciones específicas.
- Documentación complementaria en `contexto/`.

## Validación prevista

- Comandos exactos y directorio de trabajo.
- Requisitos de entorno y dependencias.
- Criterios de salida y métricas obligatorias, si existen.

En solo lectura no ejecutar comandos que escriban, incluidos builds y tests.
Los permisos de herramientas se configuran aparte; este perfil no los concede.
