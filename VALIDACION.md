# Validación del proyecto

Validación realizada el 29 de octubre de 2026:

- Compilación de todos los archivos Java (principal, pruebas y capturas): correcta con Eclipse ECJ 3.38.0, objetivo Java 17 y las dependencias fijadas en pom.xml.
- Ejecución JUnit Platform 1.11.4: **5 pruebas encontradas, 5 exitosas, 0 fallidas**.
- Interfaz JavaFX ejecutada con Monocle en modo sin pantalla: login, tabla SQLite con alta real de un registro sintético y formulario de edición.
- Tres capturas PNG generadas desde los controles JavaFX en ejecución y revisadas visualmente.

No se ejecutó Maven en el entorno de preparación (no estaba instalado). El proyecto incluye pom.xml para compilar y ejecutar normalmente con Maven en tu equipo. No se probó SSH ni MariaDB contra el servidor de la clase: faltan host, credenciales y clave verificada. Tampoco se creó o publicó un repositorio GitHub.

## Reproducir capturas de demostración

En un equipo con escritorio, desde la carpeta del proyecto:

```bash
mvn test-compile org.codehaus.mojo:exec-maven-plugin:3.5.0:java -Dexec.mainClass=mx.its5a.DemoCapture -Dexec.classpathScope=test
```

`DemoCapture` utiliza datos sintéticos en una base temporal. Captura la vista real y elimina su base temporal al terminar. No simula una sesión SSH ni debe presentarse como evidencia del host remoto. Las pruebas del controlador completo contra el servidor y las capturas definitivas deben realizarse antes de entregar.
