
# Actividad 1 — Lo que no existe en esta edición

## 1. ¿Qué edición de MongoDB estás usando? ¿Qué evidencia lo demuestra?

Estoy utilizando **MongoDB Community Server 8.0.38**.

La evidencia es el resultado del siguiente comando:

```javascript
db.version()
```

Resultado:

```text
8.0.38
```

Además, el comando:

```javascript
db.adminCommand({ buildInfo: 1 }).modules
```

Devuelve un arreglo vacío (`[]`), lo que indica que no aparecen módulos adicionales de Enterprise.

## 2. ¿A dónde escribe tu servidor su bitácora? ¿Cómo lo sabes?

El servidor escribe su bitácora en:

`/var/log/mongodb/mongod.log`

Esto se comprueba mediante el resultado del comando:

```javascript
db.serverCmdLineOpts().parsed
```

En la sección `systemLog` aparecen los siguientes valores:

- `destination: 'file'`
- `logAppend: true`
- `path: '/var/log/mongodb/mongod.log'`

Por lo tanto, MongoDB guarda sus registros en un archivo y agrega nuevas entradas sin reemplazar las anteriores.

## 3. Según REF-U3-04, ¿qué cuatro categorías de eventos registraría la auditoría nativa?

La auditoría nativa de MongoDB Enterprise y Atlas contempla cuatro categorías principales:

1. **Cambios de esquema (DDL):** creación, modificación y eliminación de estructuras de bases de datos y colecciones.
2. **Eventos de replicación y clústeres fragmentados:** cambios relacionados con conjuntos de réplicas y clústeres *sharded*.
3. **Autenticación y autorización:** accesos de usuarios, intentos de autenticación y comprobaciones de permisos.
4. **Operaciones CRUD:** creación, lectura, actualización y eliminación de documentos. Para registrar estas operaciones se requiere habilitar `auditAuthorizationSuccess: true`.

Esta funcionalidad no está disponible en MongoDB Community Edition.

## 4. ¿Qué tendría que evaluar una organización antes de licenciar Enterprise o migrar a Atlas solo para obtener auditoría?

Una organización debería evaluar los siguientes aspectos:

- **Costos:** comparar el precio de la licencia Enterprise con los costos de utilizar Atlas, incluyendo almacenamiento y recursos adicionales.
- **Necesidades de seguridad:** determinar si realmente necesita auditoría nativa para cumplir requisitos legales, normativos o internos.
- **Rendimiento:** considerar que habilitar la auditoría, especialmente para registrar operaciones CRUD exitosas, puede incrementar el consumo de recursos.
- **Infraestructura:** evaluar si conviene mantener un servidor propio con Enterprise o migrar a un servicio administrado en la nube.
- **Migración:** considerar el tiempo, los riesgos, la compatibilidad y los cambios necesarios para trasladar las bases de datos a Atlas.
- **Alternativas:** analizar si los mecanismos disponibles en Community, como el perfilador y las bitácoras del servidor, son suficientes para sus necesidades.

En conclusión, la organización debe justificar el costo y el esfuerzo de implementar una edición superior según sus necesidades reales de auditoría, seguridad y cumplimiento, en lugar de migrar únicamente por disponer de esta característica.
