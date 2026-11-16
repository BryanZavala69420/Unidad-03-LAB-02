# Actividad 9 — Comparar los tres escenarios

---
| Evento auditable | PostgreSQL (bitácora + disparador) | MongoDB Community (perfilador + bitácora) | MongoDB Enterprise/Atlas (auditoría nativa) |
|---|---|---|---|
| Acceso exitoso | **Sí.** Se registraron conexiones exitosas con usuario y origen. | **Sí.** Se observó la autenticación exitosa de `app_ventas` desde `127.0.0.1`. | Documentado, no practicado. |
| Acceso fallido | **Sí.** Se registraron intentos fallidos de autenticación y sus errores. | **Sí.** Se observaron fallos por contraseña incorrecta y usuario inexistente. | Documentado, no practicado. |
| Cambio de privilegios | **Sí.** La bitácora permite registrar sentencias de administración de roles y privilegios. | **Parcial.** Las instantáneas demostraron que se agregó `dbAdmin`, pero no identificaron quién lo hizo ni la hora exacta. | Documentado, no practicado. |
| Cambio de estructura | **Sí.** La bitácora registró operaciones DDL y cambios relacionados con la estructura. | **No.** No se obtuvo evidencia de un cambio de estructura en las pruebas realizadas. | Documentado, no practicado. |
| Modificación de datos | **Sí.** El disparador registró operaciones `INSERT`, `UPDATE` y `DELETE`. | **Sí.** El perfilador registró operaciones de actualización y eliminación sobre documentos. | Documentado, no practicado. |
| Valor anterior del dato | **Sí.** La tabla de auditoría del disparador permitió conservar valores anteriores y nuevos en los cambios registrados. | **No.** El perfilador mostró la operación y los valores nuevos, pero no conservó el valor anterior del documento. | Documentado, no practicado. |
| Lectura de datos sensibles | **No.** La lectura sensible de `soporte_ana` no produjo una entrada identificable en la bitácora ni en la tabla de auditoría. | **Parcial.** El perfilador registró consultas `find`, pero no se demostró una lectura específica de datos sensibles. | Documentado, no practicado. |
| Cambio de configuración del registro | **Sí.** Se observaron mensajes del servidor indicando cambios en parámetros de registro. | **Sí.** La bitácora registró eventos de cambio en la configuración del perfilador. | Documentado, no practicado. |