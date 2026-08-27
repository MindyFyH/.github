# .github

Archivos de salud comunitaria por defecto de la organización **Mindy FyH**.

GitHub hereda automáticamente el contenido de este repositorio en cualquier repo
de la organización que no tenga su propia versión de cada archivo. Editar acá es
editar el estándar de todos los repos a la vez, sin copiar nada.

## Qué se hereda desde acá

| Archivo | Qué es |
| -- | -- |
| `CONTRIBUTING.md` | Ramas, commits, tests, reglas de dinero y de datos |
| `SECURITY.md` | Cómo reportar una vulnerabilidad |
| `.github/PULL_REQUEST_TEMPLATE.md` | Plantilla de pull request |
| `.github/ISSUE_TEMPLATE/` | Formularios de bug y de solicitud |

## Qué NO se hereda

`CODEOWNERS`, `LICENSE`, `.gitignore`, `dependabot.yml`, `AGENTS.md`,
`.mindy.yml` y los workflows **no se heredan**: cada repositorio necesita los
suyos.

Esto importa especialmente acá, porque esta organización recibe repositorios
transferidos desde otras. **Un repositorio que llega por transferencia deja de
heredar los archivos de su organización anterior y empieza a heredar estos**,
así que conviene revisar que no traiga copias propias que pisen lo de acá.

La herencia es todo-o-nada por archivo: un repo con su propio
`.github/ISSUE_TEMPLATE/` ignora por completo el de acá.

## Este repositorio es público

GitHub exige que el repositorio de archivos por defecto sea público; con uno
privado la herencia simplemente no ocurre. Todo lo que se agregue acá queda
legible por cualquiera en internet.

En consecuencia, nada de lo que se escriba en este repositorio puede contener
nombres de clientes, RUT, montos, nombres de hosts, rutas internas ni detalles
de arquitectura. Las convenciones son genéricas por diseño. Si algo necesita ese
nivel de detalle, va en el repositorio privado que corresponda.

El workflow de seguridad que usan estos repos no vive acá: se invoca desde
`MindyNetworks/.github`, que también es público, así que un arreglo de reglas o
de pins llega a las tres organizaciones a la vez.
