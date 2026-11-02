# ITS5A Desktop — JavaFX, SSH y SQLite

 Permite crear, consultar, actualizar y eliminar registros de esa copia, incluso sin conexión.

**Integrantes:** victor manuel pech brito 58808 
                cinthia guadalupe trejo moo 58807

## Ejecutar

 Ejecuta:

```bash
mvn clean test
mvn javafx:run
```


## Preparar la conexión real


- Host y puerto SSH, usuario y contraseña SSH.
- SSH con autenticación por contraseña y reenvío TCP habilitados.
- MariaDB accesible desde el servidor SSH como `127.0.0.1:3306`, o cualquier puerto.
- Base `its5a` existente y usuario SQL con permiso `SELECT` sobre sus tablas.
- Usuario y contraseña SQL: pueden ser distintos a los de SSH.

## Usar la aplicación

1. En el login escribe host, usuario y contraseña SSH. Revisa ambos puertos, credenciales MariaDB y ruta `known_hosts`.
2. Pulsa **Conectar por SSH**. Se abre un puerto local efímero en `127.0.0.1`, se crea el túnel y JDBC conecta a `its5a` a través de él.
3. Selecciona una tabla de la lista y pulsa **Descargar tabla seleccionada**. La selección identifica la tabla; el botón inicia su descarga completa.
4. Se muestra la copia local. **Nuevo**, **Editar**, **Eliminar** y **Actualizar** operan exclusivamente sobre SQLite. Editar y eliminar requieren seleccionar una fila. Eliminar pide confirmación.
5. Cada campo tiene una casilla **NULL**: se puede desenmarcar para escribir. Texto vacío y NULL son diferentes. Los campos BLOB se representan y editan en Base64. Los campos INTEGER/REAL validan el número antes de guardar.
6. Usar **Anterior/Siguiente** para consultar páginas de 100 registros. La descarga incluye todos los registros, no solamente los visibles.
7. Para continuar otro día, pulsa **Abrir copia SQLite** desde el login. No requiere SSH. SE puede elegir un archivo generado por esta aplicación.

### Ubicación de los archivos

Cada descarga produce `snapshot-<UUID>.sqlite` en la carpeta `.its5a-local` de tu usuario. La ruta completa se muestra al abrir la copia. Suele ser `C:\Users\TU_USUARIO\.its5a-local`. Puedes copiar esos archivos entre equipos. 
La importación usa una transacción y un archivo temporal `.part`; el archivo `.sqlite` se publica solamente al completar la descarga. Una nueva descarga crea otra copia y conserva las ediciones anteriores. Las copias incompletas no se abren desde el selector.

## MVC: responsabilidades

| Capa | Archivos | Responsabilidad |
|---|---|---|
| Modelo | `model/LoginConfig.java` | Configuración de acceso y validación básica |
| Modelo | `model/RemoteService.java` | Sesión SSH, túnel, JDBC remoto y listado de tablas |
| Modelo | `model/LocalStore.java` | Importación transaccional, conversiones y CRUD SQLite |
| Vista | `view/MainView.java`, `style.css` | Controles JavaFX, tabla, formulario y mensajes |
| Controlador | `controller/AppController.java` | Eventos, validación del flujo y coordinación del modelo con la vista |
| Arranque | `App.java` | Crea la ventana y conecta vista/controlador |

La vista no ejecuta SQL ni abre sesiones SSH. El modelo no importa JavaFX. El controlador ejecuta el trabajo de red y base de datos en un `Task` con un ejecutor de un solo hilo, evitando bloquear la interfaz y evitando operaciones simultáneas sobre la misma sesión. Los valores CRUD se envían con `PreparedStatement`; los identificadores se escapan y la tabla remota se valida contra el listado de `its5a`.

## Cómo se adapta el esquema a SQLite

Es una **copia editable de datos** Cada archivo contiene:

- `records`: columnas descargadas y una clave interna INTEGER AUTOINCREMENT para identificar cada fila local, aunque la tabla de origen carezca de clave primaria o contenga duplicados.
- `snapshot_info`: nombre original de la tabla y nombre de la clave local.

Si existe una columna llamada `_local_id`, se elige otro nombre para la clave interna sin reemplazar los datos originales. Los nombres se entrecomillan para admitir espacios, palabras reservadas y comillas.

| Tipo JDBC remoto | Tipo local |
|---|---|
| TINYINT, SMALLINT, INTEGER, BOOLEAN | INTEGER |
| FLOAT, REAL, DOUBLE | REAL |
| BINARY, VARBINARY, LONGVARBINARY, BLOB | BLOB |
| BIGINT, DECIMAL/NUMERIC, fechas, texto y demás | TEXT |

BIGINT y DECIMAL se almacenan como texto para evitar redondear valores exactos o desbordar enteros SQLite. En esos campos se permite texto libre; no se reproducen las validaciones de tipos del servidor. No se copian claves foráneas, índices, triggers, valores predeterminados ni restricciones NOT NULL/UNIQUE; los campos originales pueden editarse libremente en la copia. No hay sincronización de vuelta al servidor. Se listan tablas base, no vistas. El selector abre el formato de SQLite propio del proyecto, no cualquier base arbitraria.

## Pruebas

`mvn test` ejecuta pruebas del DAO sobre bases temporales: descarga vacía, CRUD, reapertura persistente, NULL, BLOB, apóstrofos, colisión con `_local_id`, copias independientes y validación de tipos.

Para comprobar la conexión real, usa una tabla de ensayo autorizada: conecta al host, verifica el listado, descarga, crea/edita/elimina una fila, cierra la app y reabre su SQLite sin conexión. Comprueba también que la tabla remota no cambió. Prueba credenciales incorrectas y un host cuya clave no esté registrada.

## Reparto de temas

| Integrante | Implementación y defensa |
|---|---|
| 1 | Login, JavaFX, controlador, listado, hilo de trabajo y validación SSH |
| 2 | DAO SQLite, importación, formularios CRUD, pruebas y documentación |

## Referencias técnicas

- https://openjfx.io/openjfx-docs/ — JavaFX con Maven.
- https://github.com/mwiede/jsch — cliente SSH mantenido.
- https://mariadb.com/docs/connectors/mariadb-connector-j — JDBC MariaDB.
- https://github.com/xerial/sqlite-jdbc — JDBC SQLite.
