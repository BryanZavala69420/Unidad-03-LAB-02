#  Actividad 7 — Cambios de privilegios: el límite más importante




### 1. ¿Por qué un cambio de privilegios es uno de los eventos auditables más importantes?

Porque puede modificar los permisos de acceso a la base de datos. Un usuario con privilegios adicionales podría consultar, modificar o administrar información que antes no tenía permitido. Esto representa un riesgo para la seguridad y la integridad de los datos.

### 2. ¿Con qué frecuencia tomarías instantáneas? ¿Qué factores influyen?

Tomaría instantáneas diariamente y también después de cada cambio importante en usuarios o roles. La frecuencia dependería de la criticidad de la información, la cantidad de cambios administrativos, los requisitos de auditoría y el espacio de almacenamiento disponible.

### 3. ¿Quién debería poder escribir en `auditoria.instantaneas_privilegios`?

Únicamente una cuenta de auditoría o un proceso automatizado autorizado, con permisos exclusivos para insertar registros. Los administradores y usuarios de aplicación no deberían poder modificar ni eliminar las instantáneas, para conservar su integridad.

### 4. Redacta en tres líneas cómo comunicarías esta limitación a Dirección.

Las instantáneas permiten identificar cambios en los privilegios y comparar el estado de los usuarios entre dos momentos.  
Sin embargo, no registran directamente quién realizó el cambio ni la hora exacta en que ocurrió.  
Para obtener esa información se necesita una fuente de auditoría que registre los cambios de privilegios.
