
# Actividad 3 — Activar el perfilador y registrar operaciones

## Evidencia obtenida

Se consultaron las operaciones registradas en `system.profile` mediante el siguiente comando:

```javascript
db.system.profile.find(
  { ns: "comercial.clientes" },
  {
    op: 1,
    ns: 1,
    command: 1,
    user: 1,
    client: 1,
    appName: 1,
    ts: 1,
    millis: 1,
    nreturned: 1,
    docsExamined: 1,
    nModified: 1,
    ndeleted: 1
  }
).sort({ ts: 1 })
```

Se encontraron tres tipos de operaciones:

| # | Operación | Usuario | Cliente | Aplicación | Momento |
|---|---|---|---|---|---|
| 1 | `query` | `app_ventas@comercial` | `127.0.0.1` | `app-ventas-web` | 16/11/2026 22:05:30.933 UTC |
| 2 | `update` | `app_ventas@comercial` | `127.0.0.1` | `app-ventas-web` | 16/11/2026 22:05:30.936 UTC |
| 3 | `remove` | `app_ventas@comercial` | `127.0.0.1` | `app-ventas-web` | 16/11/2026 22:05:30.938 UTC |

También aparecen las mismas tres operaciones a las 22:06:03 UTC.

## Preguntas

### 1. ¿Puedes saber cuál era el RFC antes del cambio con la información de `system.profile`?

No. La información de `system.profile` permite identificar la operación de actualización y el nuevo valor enviado:

```javascript
u: {
  '$set': {
    rfc: 'GAMA780912XYZ'
  }
}
```

Sin embargo, no registra automáticamente el valor anterior del documento. Por lo tanto, no es posible conocer cuál era el RFC antes del cambio únicamente con esta evidencia.

### 2. ¿Puedes saber qué datos tenía el documento eliminado?

No. El registro de eliminación muestra:

```javascript
command: {
  q: { _id: 2 },
  limit: 1
}
```

Esto permite identificar que se intentó eliminar el documento cuyo `_id` era `2`, pero no muestra su contenido ni los valores de sus campos.

Además, `ndeleted: 0` indica que no se eliminó ningún documento en esa operación.

### 3. A diferencia de `log_statement = 'mod'` en PostgreSQL, el nivel 2 sí registró la consulta. ¿Qué ventaja y qué riesgo tiene eso?

**Ventaja:** permite obtener mayor trazabilidad de las operaciones realizadas, incluyendo consultas de lectura (`query`), actualizaciones (`update`) y eliminaciones (`remove`). También permite identificar al usuario, la dirección IP, la aplicación y el momento de ejecución.

**Riesgo:** registrar más operaciones puede incrementar el volumen de información almacenada y provocar que datos sensibles aparezcan en las bitácoras. Esto representa un riesgo de privacidad y seguridad si los registros no están protegidos adecuadamente.

### 4. ¿Por qué el campo `user` incluye `@comercial`?

Porque MongoDB identifica al usuario junto con la base de datos de autenticación en la que está definido.

En este caso:

- `app_ventas`: nombre del usuario.
- `comercial`: base de datos donde se autentica.

Por lo tanto, `app_ventas@comercial` indica que el usuario `app_ventas` se autenticó utilizando la base de datos `comercial`.
 
## Conclusión

El perfilador de MongoDB permitió identificar las operaciones realizadas por la aplicación `app-ventas-web`, así como el usuario, cliente y momento de ejecución.

Sin embargo, `system.profile` no conserva automáticamente los valores anteriores de los documentos ni el contenido completo de los documentos eliminados. Para obtener esa información sería necesario implementar un mecanismo adicional de auditoría que conserve los valores anteriores y posteriores de los documentos.
