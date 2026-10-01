# Manual del Instalador de Aplicaciones Debian Testing

_(repo: `instalador-aplicaciones-debian-testing`)_

## 1. Qué hace y qué no hace

El lanzador es una ventana que muestra una lista de acciones agrupadas por secciones. Al pulsar **Ejecutar** en una acción:

1. La ventana pasa a la vista de ejecución: un panel de registro, un campo de respuesta y el botón **Cancelar**.
2. Si el proyecto no está descargado, lo clona con `git clone`. Si ya está, ejecuta `git pull --ff-only`.
3. Ejecuta el script según el `modo` definido en `proyectos.json`. En los modos integrados usa un pseudo-terminal, de forma que `sudo`, `read` y los colores funcionan como en una terminal. Las preguntas y la contraseña de `sudo` se escriben en el campo de la parte inferior y se envían con Enter.
4. Al terminar muestra el resultado (completado, falló con su código o cancelado) y se activa **Volver**.

El lanzador no modifica los scripts descargados y no ejecuta nada como root por su cuenta. Se niega a arrancar si lo lanzas como root. **Los scripts llamados por el lanzador tienen sus propias políticas de privilegios:** la mayoría se ejecutan como usuario normal y solicitan `sudo` cuando lo necesitan, mientras que `corregir-repos-testing.sh` puede elevarse automáticamente mediante `sudo` para modificar la configuración de APT.

### 1.1. Modos de ejecución

Cada acción puede llevar el campo `modo` en `proyectos.json`:

| Modo | Qué hace |
| --- | --- |
| `auto` (por defecto) | Ejecuta el script dentro de la ventana. Si usa menús, listas, campos de texto o `dialog`, lo abre en Konsole. |
| `integrado` | Siempre dentro de la ventana, aunque el script tenga menús (no se podrán usar). |
| `gui` | Para scripts que abren su propia aplicación: actualiza el proyecto en la ventana y lanza el script sin terminal. |
| `terminal` | Siempre en Konsole, como en las primeras versiones. |

En modo `gui`, el lanzador comprueba el proceso a los 3 segundos. Si ya terminó con código 0, lo da por iniciado; si sigue ejecutándose, la interfaz también permite cerrar esta vista y asume que la aplicación ya está arrancando, pero **eso no garantiza que la aplicación gráfica haya terminado de abrirse**. Si el script termina con error, la ventana muestra su salida. La salida completa se guarda en `~/.local/share/instalador-aplicaciones-debian-testing/logs/<script>.log`.

### 1.2. Cuadros `whiptail` dentro de la ventana

Mientras un script corre dentro de la ventana, el lanzador pone por delante en el `PATH` un `whiptail` de texto propio. Convierte:

- `--yesno` en una pregunta `s/n` (Enter elige la opción por defecto; `--defaultno` y los botones `--yes-button` / `--no-button` se respetan).
- `--msgbox` en un aviso con «Pulsa Enter para continuar».
- `--infobox` en un mensaje sin pausa.

Los scripts no se modifican: en una terminal normal siguen usando el `whiptail` real. Como el script encuentra un `whiptail`, tampoco intenta instalarlo. El lanzador define además la variable `LANZADOR_INTEGRADO=1` por si un script quiere comprobar si corre aquí.

Si el script usa `--menu`, `--checklist`, `--radiolist`, `--inputbox`, `--passwordbox`, `--gauge`, `--textbox`, `--tailbox`, `--fselect` o `dialog --`, en modo `auto` se abre en Konsole.

### 1.3. Limitaciones

- El entorno de los scripts usa paginadores no interactivos (`PAGER=cat`, `GIT_PAGER=cat`, etc.). No se fuerza `TERM=dumb`: algunos scripts lo interpretan como "sin terminal interactiva" y se saltarían sus preguntas. Se respeta el `TERM` del entorno (o se usa `xterm` si no hay). Los colores que el script escribe por sí mismo se ven igual.
- Programas de pantalla completa (`nano`, `less`, `htop`, `vim`) no funcionan dentro de la ventana. Si un script los usa, pon `"modo": "terminal"`.
- Si un script carga `whiptail` desde otro archivo con `source`, el lanzador no lo detecta. Pon `"modo": "terminal"` en esa acción si usa menús.

## 2. Instalación

### 2.1. Sistema recién instalado

```bash
# Usuario normal, pide sudo solo para instalar paquetes
sudo apt install python3-pyqt6 git
git clone https://github.com/csr79a/instalador-aplicaciones-debian-testing.git ~/instalador-aplicaciones-debian-testing
cd ~/instalador-aplicaciones-debian-testing
bash instalar-lanzador.sh
```

Antes de crear la entrada del menú, `instalar-lanzador.sh` comprueba que existan:

- `lanzador.py`
- `proyectos.json`
- `python3-pyqt6`
- `git`

Si falta un archivo esencial o un paquete, termina y muestra el motivo.

Después crea el archivo `~/.local/share/applications/instalador-aplicaciones-debian-testing.desktop`. Se puede ejecutar varias veces: solo reescribe ese archivo.

Si `konsole` no está instalado avisa, pero no bloquea: solo hace falta para los scripts con menús o listas, y el lanzador usará `x-terminal-emulator` si existe.

### 2.2. Abrirlo

Desde el menú de aplicaciones, buscando **Instalador de Aplicaciones Debian Testing**, o a mano:

```bash
python3 ~/instalador-aplicaciones-debian-testing/lanzador.py
```

Si el icono no aparece enseguida, cierra sesión y vuelve a entrar, o usa el comando anterior.

## 3. Orden recomendado de uso

En una instalación nueva de Debian Testing, el orden recomendado es:

1. Instalar el lanzador.
2. Ejecutar **Corregir repositorios Debian Testing** si es necesario.
3. Ejecutar **Configurar Debian Testing**.
4. Ejecutar las acciones adicionales que necesites.
5. Ejecutar acciones de limpieza o desinstalación cuando quieras revertir componentes.

El lanzador no impone este orden. Algunas acciones tienen requisitos propios y deben respetarse aunque el lanzador permita pulsarlas en cualquier momento.

En particular, **Instalar gestor de sched-ext (GUI)** requiere que **Instalar sched-ext** se haya ejecutado previamente, porque necesita las herramientas y el servicio que instala esa acción.

## 4. Dónde se guardan los datos

Todo cuelga de `~/.local/share/instalador-aplicaciones-debian-testing/`:

- `proyectos/<nombre-del-repo>`: cada repo se clona aquí. Esa carpeta se define con `carpeta_proyectos` en `proyectos.json`.
- `shims/whiptail`: el `whiptail` de texto (se regenera solo).
- `logs/`: salida de las acciones en modo `gui`.

Los proyectos son copias propias del lanzador, distintas de tus carpetas de trabajo. Si editas un script en tu carpeta de trabajo, el lanzador no lo verá hasta que hagas `push` a GitHub.

## 5. Cómo se actualizan los scripts

Se actualizan solos: cada pulsación hace `git pull --ff-only` antes de ejecutar. Si subes un cambio a GitHub, la siguiente vez que pulses el botón usa la versión nueva.

- **Sin internet:** el `pull` falla, sale el aviso *«no se pudo actualizar; se usa la copia local»* y se ejecuta la copia ya descargada.
- **Copia local modificada o divergente:** `--ff-only` no fusiona nada por su cuenta, así que el `pull` falla con el mismo aviso y se usa la copia local. No edites las copias del lanzador; si alguna se estropea, bórrala (ver 8.7).
- **Cambios en el propio lanzador** (`lanzador.py`, `proyectos.json`): no se actualizan solos. Hazlo a mano:

```bash
# Usuario normal
git -C ~/instalador-aplicaciones-debian-testing pull --ff-only
```

## 6. Configurar la lista: `proyectos.json`

```json
{
  "carpeta_proyectos": "~/.local/share/instalador-aplicaciones-debian-testing/proyectos",
  "acciones": [
    {
      "seccion": "Terminal",
      "titulo": "Configurar terminal con Starship",
      "descripcion": "Versión para Debian.",
      "repo": "https://github.com/csr79a/terminal-starship-setup.git",
      "script": "setup-terminal-starship-debian.sh",
      "peligroso": false
    }
  ]
}
```

| Campo | Obligatorio | Significado |
| --- | --- | --- |
| `carpeta_proyectos` | Sí | Carpeta donde se clonan los repos. Admite `~`. |
| `seccion` | Sí | Tarjeta en la que aparece la acción. Las acciones con el mismo nombre se agrupan. |
| `titulo` | Sí | Nombre visible de la acción. |
| `descripcion` | No | Texto gris bajo el título. |
| `repo` | Sí | URL de clonado. La carpeta local se llama como el repo, sin `.git`. |
| `script` | Sí | Ruta del script **dentro del repo**, relativa a su raíz. |
| `peligroso` | No | Si es `true`, el botón pide confirmación. |
| `modo` | No | `auto` (por defecto), `integrado`, `gui` o `terminal`. Ver 1.1. |

### Añadir un script nuevo

1. Sube el proyecto a un repo público de GitHub.
2. Añade una entrada a `acciones` con su `repo` y la ruta de `script`.
3. Si el script abre su propia aplicación gráfica, añade `"modo": "gui"`.
4. Si necesita menús o una terminal interactiva, considera `"modo": "terminal"`.
5. Guarda el archivo y vuelve a abrir el lanzador.

Si cambias el nombre o la ruta de un script dentro de un repo, actualiza el campo `script`; mientras tanto el registro dirá que el archivo no existe.

## 7. Notas sobre los scripts incluidos

- El lanzador siempre se inicia como usuario normal y no ejecuta sus procesos como root por sí mismo.
- Los scripts incluidos gestionan sus propios privilegios. La mayoría solicitan `sudo` cuando necesitan modificar el sistema.
- **`corregir-repos-testing.sh` es una excepción importante:** si se ejecuta desde el lanzador, puede elevarse mediante `sudo` para poder modificar la configuración de APT. Esto es intencionado.
- `sched-ext-debian` necesita un kernel **ya arrancado** con `CONFIG_SCHED_CLASS_EXT=y`, según el README de ese repo. Si no lo tienes, hay que compilar uno antes (`kernel-debian-builder`) y reiniciar con él.
- **Instalar gestor de sched-ext (GUI)** necesita que **Instalar sched-ext** se haya ejecutado antes (requiere `scxctl` en el `PATH` y `scx_loader.service` activo). El lanzador no valida ese orden por ti.
- `nvidia-debian-setup` puede necesitar un paso manual después de reiniciar si tienes Secure Boot activado (enrolar la clave MOK). Revisa el registro de la ventana al terminar el script: te lo avisa ahí si aplica.
- Las acciones de limpiar y desinstalar pueden eliminar paquetes y archivos. Además de la confirmación del lanzador, algunos scripts piden su propia confirmación.

## 8. Solución de problemas

### 8.1. La ventana no se abre

Lánzala desde una terminal para ver el error:

```bash
python3 ~/instalador-aplicaciones-debian-testing/lanzador.py
```

Si dice `No module named 'PyQt6'`, instala `python3-pyqt6`.

### 8.2. «No como root»

El **lanzador** se ha abierto con `sudo` o como root. Ábrelo con tu usuario normal.

Esto no impide que un script concreto solicite `sudo` por su cuenta. Por ejemplo, `corregir-repos-testing.sh` está diseñado para elevarse cuando necesita modificar APT.

### 8.3. «Sin terminal»

Solo aparece cuando un script necesita Konsole (menús o listas, o una acción con `"modo": "terminal"`) y no hay `konsole` ni `x-terminal-emulator`. Instala Konsole:

```bash
# Usuario normal, pide sudo
sudo apt install konsole
```

### 8.4. «No se pudo clonar»

Comprueba la conexión y que la URL de `repo` es correcta y el repo es público. Los repos privados piden credenciales y el lanzador no las gestiona.

### 8.5. Un script parece colgado

Casi siempre está esperando una respuesta. Mira la última línea del registro: si es una pregunta, escribe la respuesta en el campo de abajo y pulsa Enter. Si pide la contraseña de `sudo`, el campo pasa a modo oculto. Si no responde a nada, pulsa **Cancelar**.

### 8.6. La acción en modo `gui` no abre la aplicación

Si falla en los primeros segundos, la ventana muestra la salida del script. Si no, revisa el registro completo. El nombre del archivo corresponde al nombre del script de la acción.

### 8.7. Una copia descargada está rota

Borra solo esa copia del lanzador; se volverá a clonar al pulsar el botón. Cambia el nombre por el del repo afectado:

```bash
# Usuario normal. Solo borra la copia del lanzador, no tu carpeta de trabajo
rm -rf ~/.local/share/instalador-aplicaciones-debian-testing/proyectos/sched-ext-debian
```

### 8.8. «No existe <script> en <carpeta>»

El repo se descargó pero la ruta del campo `script` no coincide con lo que hay en él. Revisa `proyectos.json` frente al contenido real del repo.

### 8.9. Un cuadro de whiptail se ve mal o el script se detiene

Es un script con un tipo de cuadro que el `whiptail` de texto no sabe convertir y que el lanzador no detectó (por ejemplo, cargado con `source`). Pon `"modo": "terminal"` en esa acción.

## 9. Desinstalación

Esto quita el lanzador y las copias descargadas de los proyectos. **No** desinstala lo que esos scripts hayan instalado en tu sistema; para eso usa las acciones de desinstalar o limpiar de cada sección antes de quitar el lanzador.

```bash
# Usuario normal
rm ~/.local/share/applications/instalador-aplicaciones-debian-testing.desktop
rm -rf ~/.local/share/instalador-aplicaciones-debian-testing
rm -rf ~/instalador-aplicaciones-debian-testing
```

## 10. Seguridad

- El lanzador ejecuta lo que haya en la rama por defecto de cada repo.
- Los scripts pueden usar `sudo` y modificar el sistema según su función.
- Protege la cuenta de GitHub con verificación en dos pasos.
- Revisa los cambios de un repo antes de pulsar el botón si no los has hecho tú.
- El `whiptail` de texto solo está en el `PATH` de los scripts que corren dentro de la ventana; no se instala en el sistema ni sustituye al real.
- La contraseña de `sudo` que escribes en el campo pasa directamente al terminal del script; el lanzador no la guarda ni la registra.
