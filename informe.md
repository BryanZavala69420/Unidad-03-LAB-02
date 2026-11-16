# Informe

## 1. Despliegue, edición y limitación

El servidor se desplegó utilizando MongoDB Community Server en un entorno Linux. La versión instalada y comprobada fue **8.0.38**.

Se verificó mediante:

```javascript
db.version()
```

Resultado:

```text
8.0.38
```

También se consultaron los módulos del servidor:

```javascript
db.adminCommand({ buildInfo: 1 }).modules
```

Resultado:

```javascript
[]
```

La edición Community no dispone de la auditoría nativa de MongoDB Enterprise. Por ello, durante el laboratorio se utilizaron el perfilador de operaciones (`system.profile`) y la bitácora del servidor (`mongod.log`) como fuentes alternativas de evidencia.

La ausencia de módulos en `buildInfo` y la versión identifican el entorno utilizado; la limitación de auditoría nativa corresponde a las capacidades de la edición Community.

## 2. Perfilador

Se utilizó el perfilador de MongoDB para registrar operaciones ejecutadas sobre la base de datos.

La configuración inicial observada fue:

```javascript
db.getProfilingStatus()
```

Resultado:

```javascript
{
  was: 0,
  slowms: 100,
  sampleRate: 1,
  ok: 1
}
```

Esto significa que el perfilador estaba desactivado inicialmente.

Posteriormente se activó el nivel 2:

```javascript
db.setProfilingLevel(2)
```

Este nivel registra todas las operaciones que procesa el perfilador, no solamente las que superan el umbral de consultas lentas.

Se observaron operaciones como:

- Consultas `find`.
- Actualizaciones `update`.
- Eliminaciones `remove`.
- Usuario que ejecutó la operación.
- Base de datos y colección involucradas.
- Aplicación y dirección del cliente, cuando estaban disponibles.
- Documentos examinados y resultados de las consultas.

También se configuró un filtro para registrar únicamente modificaciones:

```javascript
db.setProfilingLevel(1, {
  filter: {
    op: { $in: ["insert", "update", "remove"] }
  }
})
```

**Interpretación y costo:**

El nivel 2 genera mayor volumen de registros porque captura todas las operaciones, incluidas las consultas de lectura. Esto puede aumentar el consumo de almacenamiento y afectar el rendimiento.

El filtro reduce el volumen y concentra la evidencia en modificaciones, pero deja fuera las consultas de lectura. Por tanto, se pierde visibilidad sobre quién consultó información.

Además, los filtros y parámetros de consultas pueden contener datos personales o información sensible.

## 3. Bitácora de `mongod`

La configuración observada indicó que la bitácora del servidor se escribía en:

```text
/var/log/mongodb/mongod.log
```

Se identificó mediante:

```javascript
db.serverCmdLineOpts().parsed.systemLog
```

La configuración mostraba el destino `file` y el uso de `logAppend: true`.

En la bitácora se observaron eventos relacionados con:

- Autenticaciones exitosas.
- Autenticaciones fallidas.
- Conexiones de clientes.
- Aplicaciones que establecieron conexiones.
- Cambios en la configuración del perfilador.

Por ejemplo, se observó una autenticación exitosa de `app_ventas` desde `127.0.0.1`, utilizando la aplicación `app-ventas-web`.

También se observaron intentos fallidos por contraseña incorrecta y por utilizar un usuario inexistente.

Los cambios en la configuración del perfilador aparecieron mediante mensajes como `Profiler settings changed`, que permitieron identificar cuándo se modificó el nivel de perfilado y qué filtro se estableció.

Sin embargo, los eventos de configuración observados no identificaron directamente al usuario responsable del cambio de privilegios.

## 4. Privilegios

Se utilizó una colección de instantáneas para conservar el estado de los usuarios y sus roles:

```javascript
db.getSiblingDB("auditoria").instantaneas_privilegios.insertOne({
  momento: new Date(),
  tomada_por: db.runCommand({
    connectionStatus: 1
  }).authInfo.authenticatedUsers,
  usuarios: db.getUsers().users.map(u => ({
    user: u.user,
    db: u.db,
    roles: u.roles
  }))
})
```

Antes del cambio, `app_ventas` tenía únicamente el rol:

```javascript
{ role: "readWrite", db: "comercial" }
```

Durante la prueba se otorgó el rol adicional:

```javascript
db.grantRolesToUser("app_ventas", [
  { role: "dbAdmin", db: "comercial" }
])
```

La segunda instantánea mostró que `app_ventas` tenía ahora los roles `readWrite` y `dbAdmin`.

Se buscaron evidencias del cambio en ambas fuentes:

```javascript
db.system.profile.countDocuments({
  "command.grantRolesToUser": { $exists: true }
})
```

```javascript
bitacora().filter(
  e => JSON.stringify(e).includes("grantRoles")
).length
```

Ambas consultas devolvieron `0`.

**Control compensatorio:** las instantáneas permiten comparar los privilegios antes y después de un cambio, por lo que ayudan a detectar modificaciones.

**Limitación:** no permiten identificar quién realizó el cambio ni determinar su hora exacta. Solamente permiten establecer que ocurrió entre las dos instantáneas.

Finalmente, se revocó el rol adicional:

```javascript
db.revokeRolesFromUser("app_ventas", [
  { role: "dbAdmin", db: "comercial" }
])
```

## 5. Conservación

La conservación de evidencias es importante porque los registros pueden perderse por rotación, reinstalación, fallos del servidor o eliminación de archivos.

Durante el laboratorio se identificó que la bitácora se almacenaba localmente en `/var/log/mongodb/mongod.log`. Esto representa un riesgo si el servidor deja de estar disponible o si alguien con privilegios administrativos modifica o elimina los registros.

También se intentó exportar evidencia del perfilador a:

```text
evidencias/system-profile.json
```

El primer intento produjo el error:

```text
bash: evidencias/system-profile.json: No such file or directory
```

El problema fue que no existía el directorio `evidencias`. Se indicó crearlo con:

```bash
mkdir -p evidencias
```

y volver a ejecutar la exportación.

No hay evidencia con esta edición que confirme que la exportación posterior terminó correctamente; por ello, no se puede afirmar que el archivo haya quedado conservado.

Como medida de protección, se recomienda exportar periódicamente las evidencias, conservar copias externas y restringir quién puede modificarlas o eliminarlas.

## 6. Comparación

La Actividad 9 permitió comparar las capacidades observadas y documentadas de PostgreSQL, MongoDB Community y MongoDB Enterprise/Atlas.

| Aspecto | PostgreSQL (bitácora + disparador) | MongoDB Community (perfilador + bitácora) | MongoDB Enterprise/Atlas (auditoría nativa) |
|---|---|---|---|
| Acceso exitoso | Sí, observado. | Sí, observado. | Documentado, no practicado. |
| Acceso fallido | Sí, observado. | Sí, observado. | Documentado, no practicado. |
| Cambio de privilegios | Sí, registrado mediante sentencias de administración. | Parcial: las instantáneas detectaron el cambio, pero no identificaron al responsable ni la hora exacta. | Documentado, no practicado. |
| Cambio de estructura | Sí, observado mediante sentencias DDL. | No hay evidencia con esta edición. | Documentado, no practicado. |
| Modificación de datos | Sí, mediante bitácora y disparador. | Sí, mediante el perfilador. | Documentado, no practicado. |
| Valor anterior del dato | Sí, mediante el disparador de auditoría. | No: el perfilador no conservó el valor anterior. | Documentado, no practicado. |
| Lectura de datos sensibles | No quedó evidencia identificable en la prueba de lectura. | Parcial: se registraron consultas, pero no se demostró una lectura específica de datos sensibles. | Documentado, no practicado. |
| Cambio de configuración del registro | Sí, observado. | Sí, observado mediante mensajes de cambio del perfilador. | Documentado, no practicado. |

La comparación muestra que PostgreSQL permitió combinar la bitácora del servidor con un disparador para conservar valores anteriores y nuevos. MongoDB Community proporcionó trazabilidad mediante el perfilador y la bitácora, pero no permitió demostrar todos los detalles de los cambios de privilegios.

Las capacidades de Enterprise/Atlas se consultaron como referencia documental, pero no se probaron durante este laboratorio.

## 7. Recomendación

Para una organización que necesite auditar cambios de privilegios en MongoDB, recomendaría utilizar **MongoDB Enterprise o Atlas con auditoría nativa habilitada y correctamente configurada**.

**Hechos verificados durante el laboratorio:**

- Se utilizó MongoDB Community Server 8.0.38.
- La edición Community no proporcionó auditoría nativa.
- El perfilador y la bitácora permitieron observar operaciones, autenticaciones y cambios de configuración.
- Las instantáneas detectaron que se agregó el rol `dbAdmin` a `app_ventas`, pero no identificaron quién realizó el cambio ni la hora exacta.

**Aspectos documentados, pero no practicados:**

- Enterprise/Atlas dispone de auditoría nativa para registrar eventos de auditoría, incluidos eventos relacionados con autenticación y autorización.
- Esta capacidad permitiría contar con evidencia más apropiada para investigar cambios de privilegios, siempre que se configure para registrar los eventos requeridos.

La recomendación es habilitar la auditoría nativa, proteger los registros y enviarlos a un sistema externo con acceso restringido. Esto permitiría mejorar la atribución de responsabilidades y la conservación de evidencias.

---

# Preguntas de reflexión

## 1. ¿Por qué el perfilador no debe presentarse como un mecanismo de auditoría de seguridad?

Porque su propósito principal es recopilar información sobre el comportamiento y rendimiento de las operaciones de la base de datos. Aunque registra operaciones, usuarios y consultas, no garantiza una auditoría completa de seguridad ni permite demostrar todos los eventos administrativos.

Por ejemplo, durante el laboratorio no se obtuvo evidencia del comando `grantRolesToUser` en `system.profile`.

## 2. ¿Por qué el perfilador sí aporta trazabilidad aunque no sea auditoría?

Porque permite relacionar operaciones con información como el usuario, la colección, la aplicación, la dirección del cliente y el momento de ejecución.

Durante el laboratorio se identificaron consultas y modificaciones realizadas por `app_ventas` desde `127.0.0.1`, utilizando `app-ventas-web`.

Esto permite reconstruir parcialmente la actividad realizada en el servidor.

## 3. ¿Qué riesgo introduce el nivel 2 al registrar filtros de consultas que pueden contener datos personales?

El nivel 2 registra todas las operaciones, por lo que los filtros de consultas pueden quedar almacenados en `system.profile`.

Si esos filtros contienen RFC, teléfonos, nombres u otros datos personales, la colección de perfilado puede convertirse en otro lugar donde se almacena información sensible.

También aumenta el volumen de registros y el consumo de almacenamiento. Por ello, se debe limitar el acceso y el tiempo de conservación de esta información.

## 4. ¿Qué diferencia hay entre detectar un cambio y atribuirlo a alguien?

Detectar un cambio significa demostrar que el estado de un objeto o usuario fue modificado.

Atribuirlo significa identificar quién realizó la operación, cuándo ocurrió y desde qué conexión o aplicación.

En el laboratorio, las instantáneas demostraron que `app_ventas` recibió el rol `dbAdmin`, pero no permitieron identificar quién ejecutó el cambio ni la hora exacta.

Por lo tanto, detectar una modificación no significa necesariamente poder responsabilizar a una persona o cuenta.

## 5. ¿Por qué reconocer una limitación técnica es parte del trabajo profesional del DBA?

Porque un DBA debe comunicar con honestidad qué información puede demostrar el sistema y qué información no está disponible.

Afirmar que existe evidencia cuando no se ha registrado puede provocar conclusiones incorrectas durante una investigación o auditoría.

Reconocer las limitaciones permite proponer controles compensatorios, seleccionar herramientas adecuadas y evitar prometer un nivel de seguridad que la infraestructura realmente no proporciona.
