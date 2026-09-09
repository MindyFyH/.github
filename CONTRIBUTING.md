# Cómo se trabaja acá

Convenciones de los repositorios de Mindy FyH. Aplican por defecto a todos; si
un repositorio necesita algo distinto, lo escribe en su propio
`CONTRIBUTING.md`, que reemplaza a este.

Estos proyectos se construyen con ayuda de IA y no todos los que trabajan acá
programan. Aun así las reglas son más estrictas que en el resto de la
organización, y el motivo es simple: acá los errores se cuentan en plata. Un
redondeo mal hecho o un permiso mal escrito no se ven al probar a mano — se
descubren meses después, en un descuadre o en datos de un cliente visibles para
otro.

## Ramas

La rama principal se llama **`main`**, siempre, y está protegida: no se puede
pushear directo.

Para trabajar, sal de `main` y crea una rama con un nombre que diga qué estás
haciendo:

```
feat/agregar-informe-mensual
fix/corregir-calculo-iva
chore/subir-dependencias
```

Después abres un pull request hacia `main`. Es el único camino. Las ramas se
borran solas al mergear, y el merge es siempre por squash: un commit por PR.

## Mensajes de commit

Empieza el mensaje con una de estas palabras y dos puntos: `feat:`, `fix:`,
`docs:`, `style:`, `refactor:`, `chore:`. De ahí sale el registro de cambios.

```
feat: agregar informe de honorarios por mes
fix: el IVA no se calculaba en las notas de crédito
```

## Tests

Acá los tests **sí bloquean**. El umbral es 85% de cobertura **sobre las líneas
que agregues o modifiques**, no sobre el proyecto entero: la deuda vieja no
molesta, pero lo que escribes entra cubierto.

Si le pides tests a un agente, revisa que verifiquen algo de verdad. Un test que
no comprueba nada es peor que no tener test: se ve verde y da confianza falsa.

Y si arreglas un bug, el test va **primero** y tiene que fallar antes del
arreglo. Un test escrito después casi nunca prueba lo que cree probar.

### Los tres tests que más pagan en estos proyectos

1. **Cuadratura**: dado un conjunto de asientos, que los débitos sumen igual que
   los créditos. Una sola comprobación que atrapa toda una familia de errores.
2. **Casos borde de montos**: cero, negativos, el máximo esperado, y los montos
   que caen justo en la mitad de un redondeo.
3. **Permisos con dos usuarios**: autenticarse como el cliente A y verificar que
   no ve nada del cliente B. Es el error más caro que se puede cometer acá, y el
   más fácil de introducir sin darse cuenta al agregar una tabla o una vista.

## Dinero

**Nunca uses punto flotante para montos.** Enteros en pesos, o un tipo decimal
exacto. En JavaScript `0.1 + 0.2` no da `0.3`, y eso termina siendo un descuadre.

```js
// NO
const iva = monto * 0.19;

// SÍ — pesos enteros, redondeo explícito y elegido a propósito
const iva = Math.round(monto * 19 / 100);
```

**El redondeo se elige, no se hereda.** El aritmético y el bancario dan
resultados distintos en los montos que caen en la mitad. Decide cuál usa el
proyecto, escríbelo en el `README.md`, y pruébalo con esos casos.

**El IVA va en una sola constante**, en un solo lugar, con su test.

Un cálculo que alimenta un informe contable o una declaración no se cambia sin
un test de referencia: el resultado esperado de un conjunto de entrada fijo,
versionado junto al código. Si el cambio es correcto, ese resultado esperado se
actualiza en el mismo PR, y su diff es la evidencia de qué cambió en la práctica.

## Datos reales: nunca en el repositorio

**Cero datos reales de clientes, ni en los tests.** Los datos de prueba son
inventados: RUT que no existen, nombres que no existen, montos redondos, todo
dentro de `tests/fixtures/`.

Esto es más grave de lo que parece, porque el escaneo automático **no lo
atrapa**. Las herramientas de seguridad buscan patrones de contraseñas y llaves;
una planilla de contabilidad real es una tabla con nombres, RUT y montos, no
coincide con ningún patrón, y pasa el control sin una sola alerta. Por eso estos
repositorios tienen un chequeo aparte que rechaza `.csv`, `.xlsx` y respaldos de
base de datos fuera de `tests/fixtures/`.

Si necesitas reproducir un problema con datos reales, hazlo en tu computador y
no subas el archivo. Si ya lo subiste, avísalo: hay que limpiar el historial, y
borrarlo del último commit no alcanza.

## Secretos

**Nunca subas** archivos `.env`, llaves, certificados, tokens ni contraseñas.
Cuando un proyecto necesita variables de entorno, se versiona un `.env.example`
con los **nombres** de las variables y sin ningún valor.

Si un secreto llegó a un commit, **lo primero es rotar la credencial** — generar
una nueva y desactivar la vieja. Borrarlo del historial no lo invalida:
cualquiera que haya clonado el repositorio antes ya lo tiene.

Instala el hook que revisa esto antes de cada commit, una vez por proyecto:

```
git config core.hooksPath .githooks
```

Y si el escaneo marca algo, **no lo silencies para que pase**: confirma primero
si es real.

## Gestor de paquetes

**pnpm es el estándar** en los proyectos de JavaScript y TypeScript. Los
repositorios nuevos arrancan con pnpm.

**npm queda como legacy.** Lo usan la mayoría de los repositorios y no se migran
por ahora: si el repositorio ya tiene `package-lock.json` y funciona, se queda
así. Lo que no se hace es empezar algo nuevo con npm.

**yarn no se usa.** No hay ningún repositorio con `yarn.lock` en la organización
y no se adopta. Si llega uno transferido desde afuera, se migra antes de
integrarlo:

```bash
rm yarn.lock && pnpm import && pnpm install
```

### Un solo lockfile por repositorio

Nunca conviven dos. Es el error que más cuesta ver, porque no rompe nada de
inmediato: el CI elige uno, instala un árbol de dependencias distinto al que
tienes local, y la diferencia aparece semanas después como un bug que no
reproduce nadie.

Ya pasó: un repositorio de la organización tenía `pnpm-lock.yaml` y
`package-lock.json` al mismo tiempo, y el CI instalaba con el equivocado.

Si migras de npm a pnpm, el `package-lock.json` se borra en el mismo commit:

```bash
rm package-lock.json && pnpm import && pnpm install
```

`pnpm import` lee el lockfile viejo antes de borrarlo, así que las versiones
exactas que tenías se conservan.

### Por qué importa, más allá del gusto

Los pines de seguridad viven en el gestor. En pnpm son `pnpm.overrides`, en npm
`overrides`, en yarn `resolutions`: tres sintaxis para lo mismo. Cuando sale un
CVE y hay que fijar una versión transitiva, con un solo gestor hay un solo lugar
donde buscar en todos los repositorios.

Esto no es teórico. `mindyv2` tiene 17 pines de CVE en `pnpm.overrides`, y el CI
los estaba ignorando porque instalaba con npm: probaba un árbol sin parchar. El
piso de seguridad descartando los parches de seguridad.

Conviene además declarar la versión en el `package.json`, para que el CI y cada
máquina usen la misma:

```json
"packageManager": "pnpm@10.0.0"
```

## Tratamiento de datos personales

Cada repositorio lleva un `docs/tratamiento-de-datos.md`: la ficha de qué datos
de personas toca ese código, para qué, con qué base de licitud, quién más los
ve, si salen del país, cuánto se guardan y cómo se protegen.

Lo exige la **Ley 21.719**, vigente desde diciembre de 2026. El registro formal
es de la compañía, pero esta ficha solo la puede llenar quien escribe el código:
la región de un proyecto Supabase, una integración nueva o una tabla con RUT no
se adivinan desde afuera.

**Se actualiza en el mismo pull request que introduce el cambio**, igual que el
`CHANGELOG.md`. Concretamente, cuando el cambio agrega un dato personal nuevo,
un proveedor externo que lo recibe, o una integración que lo mueve.

Si el proyecto no trata datos de personas, se escribe eso en el archivo, en una
línea, con el motivo. La ausencia declarada vale; la silenciosa no se puede
auditar.

Dos secciones concentran los errores más caros. La de **transferencias
internacionales**, porque un proyecto en Vercel con base en Supabase casi
siempre tiene los datos fuera de Chile sin que nadie lo haya decidido a
propósito. Y la de **datos sensibles** —salud, datos de niños, biométricos—,
porque cambian las reglas: exigen consentimiento reforzado y obligan a avisarle
al titular si hay una brecha.

## Documentación

Toda la documentación del proyecto vive en el directorio `docs/` de la raíz del
repositorio, y se escribe en Markdown (`.md`). No se dejan documentos sueltos en
la raíz ni repartidos por otras carpetas: si es documentación, va en `docs/`.

El `README.md` es la excepción: se queda en la raíz, porque es lo primero que se
ve al abrir el repositorio. Todo lo demás —guías, notas técnicas, decisiones,
instrucciones de setup— va en `docs/`, con nombre en minúsculas y palabras
unidas por guiones (`docs/deploy-a-staging.md`, no `docs/Deploy Staging.md`).

## Registro de cambios

Cada repositorio tiene un `CHANGELOG.md`. Si tu cambio altera cómo se comporta
el proyecto, agrega una línea bajo `## [No publicado]`, en lenguaje de quien usa
el proyecto. Los cambios de configuración o dependencias no la necesitan.

El historial de commits ya dice *qué* se cambió. El changelog es para el *por
qué*, que es exactamente lo que se pierde cuando el código lo escribió un
agente.

## Trabajar con agentes

Cada repositorio tiene un `AGENTS.md` en la raíz con el contexto del proyecto y
las reglas de dinero. Si el agente se equivoca siempre en lo mismo, la
corrección va **en ese archivo**, no en el prompt de cada sesión.

Un cambio escrito por un agente entra igual que cualquier otro: con alguien que
lo leyó y responde por él. "Lo hizo la IA" no es una explicación cuando un
informe sale mal.

Para reportar una vulnerabilidad, ver [SECURITY.md](SECURITY.md).
