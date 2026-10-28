# Contexto técnico inicial

Fuente: documentación y agentes del proyecto de referencia, consultados el
2026-10-28. Este resumen no sustituye a las convenciones vigentes.

- Compatibilidad estricta Java 8; arquitectura DDD Lite / pragmática.
- Cambios pequeños y acotados, mejora incremental del legacy tocado.
- No añadir dependencias, capas o patrones sin necesidad justificada.
- Terminología de dominio según `GLOSSARY.md`.
- Tests según `TESTING.md` y el módulo: verificar JUnit, Mockito/JMockit y
  fixtures existentes antes de elegir herramientas.

| Módulo | Responsabilidad |
|---|---|
| `arrhh_selpro-war` | Spring MVC, servicios, vistas y persistencia JPA |
| `arrhh_selpro-integration` | Cliente ARPAD, CXF y contratos WSDL |
| `arrhh_selpro-config` | Configuración de entornos |
| `arrhh_selpro-reports` | Informes |
| `arrhh_selpro-scripts-ddl` | Scripts y versionado SQL |
| `arrhh_selpro-ear` | Empaquetado EAR |
| `arrhh_selpro-entrega` | Distribución |
