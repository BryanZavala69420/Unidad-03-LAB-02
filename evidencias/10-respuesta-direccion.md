# Actividad 10 — Responder a Dirección
---
## 1. ¿Quién cambió el RFC de María García en la Actividad 2? ¿Y en la Actividad 3?

- **Actividad 2:** No hay evidencia con esta edición. Esta actividad únicamente activa el colector de registros mediante `logging_collector`; no contiene un cambio del RFC de María García.
- **Actividad 3:** No hay evidencia con esta edición que permita demostrar de manera independiente quién realizó el cambio. Se ejecutó un `UPDATE` mediante una conexión configurada con el usuario `app_ventas`, pero la consulta a `dba.bitacora` devolvió `(0 rows)`. Por lo tanto, no existe un registro de auditoría que confirme la identidad del responsable.

## 2. ¿Cuándo y desde qué aplicación?

- **Fecha y hora:** No hay evidencia con esta edición. La Actividad 3 no produjo una entrada de bitácora con marca de tiempo.
- **Aplicación:** El comando de la Actividad 3 especificó `application_name=app-ventas-web`. Sin embargo, al no existir un registro del evento, no hay evidencia independiente que confirme la aplicación desde la que se ejecutó.

## 3. ¿Qué valor tenía antes?

No hay evidencia con esta edición. La Actividad 3 únicamente mostró que el comando terminó con `UPDATE 1`, lo que confirma que se modificó una fila, pero no se configuró una auditoría que conservara el valor anterior del RFC.

## 4. ¿Cambió alguien los privilegios de `app_ventas`?

Sí. En la Actividad 6 se ejecutó la siguiente sentencia:

`GRANT UPDATE ON clientes TO app_ventas;`

La bitácora registró el evento:

- **Usuario:** `postgres`.
- **Fecha y hora:** `2026-10-25 22:50:09.367 UTC`.
- **Aplicación:** `psql`.
- **Origen:** `[local]`.
- **Cambio:** Se otorgó el privilegio `UPDATE` sobre la tabla `clientes` a `app_ventas`.

Esto demuestra que los privilegios de `app_ventas` sí fueron modificados durante el laboratorio.