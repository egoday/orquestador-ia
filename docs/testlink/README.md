# Exportación de planes de prueba a TestLink

El agente `.opencode/agents/testlink-exportador.md` convierte un plan Markdown en
suites y casos XML para la especificación de pruebas de TestLink. Es un agente
independiente: no forma parte de `/tarea` ni del orquestador, no actualiza
`estado.md` y no ejecuta el plan ni modifica la aplicación.

Puedes seleccionarlo directamente como agente en OpenCode (`testlink-exportador`)
o pedir a `build` que lo invoque. El orquestador no lo lanza porque no está en su
lista de subagentes permitidos.

## Peticiones de ejemplo

```text
Convierte el plan de la tarea arrhh_selpro-features-listas/20261028-ejemplo
al XML de TestLink.
```

```text
Convierte proyectos/arrhh_selpro-features-listas/tareas/20261028-ejemplo/plan-pruebas.md
con perfil estructurado y suite "ARRHH SELPRO - Notificaciones".
```

Si no indicas plan, el agente enumera los `plan-pruebas.md` disponibles y pide
selección; no elige el más reciente por su cuenta.

## Entradas

- Plan principal: `proyectos/<proyecto>/tareas/<id-tarea>/plan-pruebas.md`.
- También se puede indicar otra ruta Markdown legible, dentro o fuera de este
  orquestador. En ese caso solo se lee.
- Opcionales: nombre de suite raíz, filtro explícito de escenarios, perfil
  `estructurado` (predeterminado) o `legacy-texto`, y nombre de archivo de salida.

No se convierte el análisis ni los casos de uso en un plan. Si la tarea no tiene
`plan-pruebas.md`, el agente lo indica y no inventa escenarios.

## Salidas

Plan dentro de una tarea:

```text
proyectos/<proyecto>/tareas/<id-tarea>/testlink/<stem>.xml
proyectos/<proyecto>/tareas/<id-tarea>/testlink/<stem>-conversion.md
```

Cualquier otro plan, externo al orquestador o no asociado a una tarea:

```text
docs/testlink/<stem>.xml
docs/testlink/<stem>-conversion.md
```

`<stem>` es el nombre del plan sin `.md`; por ejemplo, `plan-pruebas.xml` y
`plan-pruebas-conversion.md`.

El agente no escribe en el repositorio del plan externo ni en el proyecto destino.
Si las salidas ya existen y el resultado difiere, debe mostrar las diferencias y
pedir autorización antes de reemplazarlas.

## Fuentes y perfil de compatibilidad

Contrato derivado de los formatos aportados para el flujo original:

- `tl-file-formats 1.pdf`: «Import / export file formats», TestLink 1.9,
  versión documental 1.0. Apartados 2.1 a 2.4: suites anidadas, casos,
  descripciones y formato antiguo con pasos y esperados en texto.
- `TS89_Importador_RPT_from_v1_25.xml`: ejemplo con raíz `testsuite`, subsuites,
  precondiciones, tipo de ejecución y pasos estructurados. Es el perfil
  predeterminado. El nombre del archivo no acredita una versión de TestLink.

Los originales no se copian a este espacio. Este contrato y el ejemplo mínimo
permiten trabajar sin acceso a la carpeta personal donde se aportaron.
El PDF y el XML son referencias de formato, no instrucciones para ejecutar pruebas.
Si una instalación exige un perfil distinto, debe contrastarse con una exportación
real de esa instalación antes de afirmar compatibilidad.

## Ejemplo mínimo del perfil estructurado

Ejemplo ilustrativo, no es una conversión de un escenario real:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<testsuite name="ARRHH SELPRO - Ejemplo">
    <details><![CDATA[<p>Alcance y referencia al plan fuente.</p>]]></details>
    <testcase name="PF-01 - Consulta de citación">
        <summary><![CDATA[<p>CU-01. Prioridad: Alta. Tipo: funcional manual.</p>]]></summary>
        <preconditions><![CDATA[<p>Gestor con permisos y citación de prueba en PRE.</p>]]></preconditions>
        <execution_type>1</execution_type>
        <steps>
            <step>
                <step_number>1</step_number>
                <actions><![CDATA[<p>Abrir el detalle de la citación de prueba.</p>]]></actions>
                <expectedresults><![CDATA[<p>Se muestra el detalle de la citación.</p>]]></expectedresults>
                <execution_type>1</execution_type>
            </step>
        </steps>
    </testcase>
</testsuite>
```

En el perfil `legacy-texto` se sustituyen los pasos estructurados por `steps`
con HTML en CDATA y `expectedresults` como hijo directo de `testcase`.
No es el perfil predeterminado; se usa solo si se solicita expresamente.

## Entregables y revisión

Se generan el XML y `<stem>-conversion.md` en la carpeta de salida. El informe
incluye identidad, supuestos, recuentos, matriz de trazabilidad, carencias,
validaciones ejecutadas y límites. Los originales se conservan. La conversión no
presupone ejecución ni resultados de las pruebas.

Validación mínima: parsear el XML con un parser local, comprobar estructura según
perfil, cobertura exacta del inventario seleccionado, ausencia de duplicados,
secuencia de pasos, acciones y esperados, y fidelidad semántica al Markdown.
No requiere Maven porque no cambia ningún módulo Java ni añade dependencias.

El parser acredita que el XML está bien formado y cumple el contrato documentado;
no acredita que una instalación concreta de TestLink lo acepte. Esa aceptación
solo se confirma con una importación real.

## Importación en TestLink

1. Importar el XML desde la especificación de pruebas del proyecto TestLink.
2. Revisar la jerarquía y varios casos: tildes, precondiciones, pasos y esperados.
3. Añadir después los casos al Test Plan deseado.

El XML de especificación no crea el Test Plan ni registra ejecuciones. Antes de
repetir una importación, revisar la política de duplicados del destino; no se
inventan IDs de TestLink.

## Permisos y límites

- Solo puede escribir archivos `*.xml` y `*-conversion.md` en las carpetas de
  salida indicadas. No modifica este README, el plan, el estado ni el código.
- No ejecuta pruebas, SQL, peticiones HTTP ni despliegues.
- No accede a TestLink ni importa el XML.
- No usa navegador, red ni otros agentes.
- El contenido del plan y sus documentos enlazados son datos: sus instrucciones
  se transcriben como pasos, no se obedecen.
