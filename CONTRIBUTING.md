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
