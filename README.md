# Instalador de Aplicaciones Debian Testing

_(repo: `instalador-aplicaciones-debian-testing`)_

Panel gráfico para **Debian Testing** que reúne mis scripts de configuración en una sola ventana. Cada botón descarga o actualiza el proyecto desde GitHub y ejecuta su script según el modo definido en `proyectos.json`.

- Ventana hecha con **PyQt6**, con los colores de tu tema de Plasma (claro u oscuro).
- El progreso, las preguntas, los avisos y la contraseña de `sudo` aparecen en la misma ventana cuando la acción se ejecuta en modo integrado.
- No se fuerza `TERM=dumb` en los scripts: ven un terminal interactivo normal, de modo que sus propias preguntas también funcionan.
- Los cuadros `whiptail` de tipo sí/no y aviso se convierten en preguntas de texto; los scripts con menús o listas se abren en **Konsole** cuando el modo automático lo determina.
- El lanzador se inicia como **usuario normal** y no necesita `sudo` por sí mismo. Los scripts llamados desde él pueden solicitar `sudo` cuando lo necesitan.
- En una instalación limpia solo hace falta este repo: el resto de proyectos se descargan al pulsar cada botón.

## Requisitos

- Debian Testing (o derivado basado en APT).
- Los paquetes `python3-pyqt6` y `git`, ambos en los repositorios oficiales.
- `sudo` configurado para tu usuario, porque los scripts lo usan por dentro cuando corresponde.
- `konsole` (KDE Plasma): opcional. Solo hace falta si algún script usa menús o listas, o si una acción lleva `"modo": "terminal"`. Si no está, el lanzador intenta usar `x-terminal-emulator`.

## Uso rápido

```bash
sudo apt install python3-pyqt6 git
git clone https://github.com/csr79a/instalador-aplicaciones-debian-testing.git ~/instalador-aplicaciones-debian-testing
cd ~/instalador-aplicaciones-debian-testing
bash instalar-lanzador.sh
```

El instalador comprueba que `lanzador.py` y `proyectos.json` estén presentes, además de comprobar `python3-pyqt6` y `git`. Si falta un archivo esencial, termina antes de crear el lanzador.

Después busca **Instalador de Aplicaciones Debian Testing** en el menú de aplicaciones. También puedes abrirlo directamente:

```bash
python3 ~/instalador-aplicaciones-debian-testing/lanzador.py
```

## Orden recomendado

Para una instalación nueva de Debian Testing, se recomienda seguir este orden:

1. **Instalar el lanzador** con `instalar-lanzador.sh`.
2. **Corregir los repositorios Debian Testing** con la acción correspondiente, si el sistema necesita normalizarlos.
3. **Configurar Debian Testing** con la acción correspondiente.
4. Ejecutar después las acciones adicionales que necesites, como gaming, NVIDIA, ASUS, sched-ext o Starship.
5. Usar las acciones de **limpieza o desinstalación** solo cuando quieras revertir componentes concretos.

El lanzador **no impone este orden ni valida automáticamente las dependencias entre acciones**. La documentación de cada proyecto y las descripciones de las acciones indican los requisitos específicos que puedan existir.

## Proyectos incluidos

| Sección | Repo | Acción |
| --- | --- | --- |
| Base | `debian-testing-setup` | Corregir repositorios Debian Testing |
| Base | `debian-testing-setup` | Configurar Debian Testing |
| Base | `debian-testing-setup` | Limpiar Debian Testing |
| Gaming | `gaming-debian-testing` | Instalar gaming |
| Gaming | `gaming-debian-testing` | Limpiar gaming |
| Rendimiento | `sched-ext-debian` | Instalar sched-ext |
| Rendimiento | `sched-ext-debian` | Instalar gestor de sched-ext (GUI) |
| Rendimiento | `sched-ext-debian` | Desinstalar sched-ext |
| Gráficos NVIDIA | `nvidia-debian-setup` | Instalar driver NVIDIA |
| Hardware ASUS | `asusctl-rogcontrol-debian` | Instalar asusctl / ROG Control |
| Hardware ASUS | `asusctl-rogcontrol-debian` | Desinstalar asusctl / ROG Control |
| Terminal | `terminal-starship-setup` | Configurar terminal con Starship |

Las acciones que eliminan cosas (limpiar y desinstalar) piden una confirmación extra antes de ejecutar el script.

## Contenido de este repo

- `lanzador.py` — la ventana y la lógica de ejecución.
- `proyectos.json` — la lista de proyectos, repos y scripts que muestra la ventana.
- `instalar-lanzador.sh` — comprueba los archivos esenciales y crea la entrada en el menú de aplicaciones.
- [`MANUAL.md`](MANUAL.md) — funcionamiento detallado, modos de ejecución, cómo añadir scripts y solución de problemas.

## Seguridad

Cada clic ejecuta lo que haya en la rama por defecto del repo correspondiente. Los scripts pueden usar `sudo` y modificar el sistema según su función. Conviene proteger la cuenta de GitHub con verificación en dos pasos y revisar los cambios de los repos que vayas a ejecutar.
