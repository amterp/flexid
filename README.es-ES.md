

# Go FlexID: Generación de IDs Flexibles

[![Project status](https://img.shields.io/github/tag/amterp/flexid.svg?style=flat-square)](https://github.com/amterp/flexid/tags/latest)
[![go.dev reference](https://img.shields.io/badge/go.dev-reference-007d9c?logo=go&logoColor=white&style=flat-square)](https://pkg.go.dev/github.com/amterp/flexid)
[![Go Report Card](https://goreportcard.com/badge/amterp/flexid?cache=0)](https://goreportcard.com/report/amterp/flexid)
[![MIT Licensed](https://img.shields.io/badge/license-MIT-blue.svg)](./LICENSE)

Una biblioteca de Go para generar IDs de cadena cortos y configurables con un componente de tiempo y uno aleatorio.
También conocidos como "FIDs": **F**lexible **ID**s (IDs Flexibles).

Útil cuando quieres *garantizar* 0 colisiones entre dos puntos en el tiempo, mientras minimizas colisiones para
IDs generados *dentro* de ese intervalo.

**Por ejemplo:**

> Generar IDs en base-62 con un tamaño de tick de 100 milisegundos, y con 5 caracteres aleatorios extra al final.

Puede generar `J3GzHY02O6S`.

## Características ✨

- **Altamente Configurable:**
  - Establece tu propia **época** (fecha/hora de inicio).
  - Ajusta el **tamaño del tick** (milisegundos, segundos, minutos, etc.).
  - Elige diferentes **alfabetos** (Base62, Base16 (hex), Base64URL, Crockford Base32, o personalizado).
  - Controla la **longitud** de la parte aleatoria. Redúcela para IDs más cortos, aumentala para una mayor resistencia a colisiones.
- **Corto:** Genera IDs compactos usando conjuntos de caracteres configurables (alfabetos).
- **Resistente a Colisiones:** El sufijo aleatorio criptográficamente seguro minimiza la probabilidad de colisiones.
- **Fácil de Usar:** Comienza con valores predeterminados sensatos o crea generadores ajustados finamente.
- **Ligero**: Cero dependencias.

## Instalación 🚀

```sh
go get github.com/amterp/flexid
```

## Uso 🔨

### Básico

Puedes usar el generador predeterminado, que utiliza algunos ajustes predeterminados sensatos.

```go
import fid "github.com/amterp/flexid"

id := fid.MustGenerate()
anotherId, err := fid.Generate()
```

### Avanzado (Ajustes Personalizados)

Puedes crear tu propio `Generator` pasando tu propio objeto `Config`.

```go
import fid "github.com/amterp/flexid"

// NewConfig creates with defaults.
// You can then chain With methods to customize settings.
config := fid.NewConfig().
	WithTickSize(fid.Second).
	WithNumRandomChars(6).
	WithAlphabet(fid.Base16LowerAlphabet)

// Create the generator with the config.
generator, err := fid.NewGenerator(config)
if err != nil {
    panic(err)
}

// Use it to generate ids.
id := generator.MustGenerate()
```

## ¿Cómo funciona? 🤔

¡Es simple!

TLDR: Dado un alfabeto, codifica una marca de tiempo de la época y anexa caracteres aleatorios del alfabeto.

### Componente de Tiempo

Cada ID comienza con una marca de tiempo de la época codificada. Esta marca de tiempo se mide en *ticks*. El tamaño del tick es configurable
y por defecto es un milisegundo.

Por ejemplo, si especificas un tamaño de tick de 100 ms, el componente de tiempo será el número de ticks de 100 ms desde la época.
En cada nuevo tick, el componente de tiempo cambia, garantizando la unicidad a través de cada incremento de tu tamaño de tick (100 ms en este ejemplo).
Cuanto mayor sea tu tamaño de tick, más corto será el componente de tiempo al codificarlo (ya que el número de ticks desde la época disminuirá).

Algunos ejemplos de cómo **solo este segmento inicial** varía dependiendo del tamaño del tick:

| Tamaño del Tick          | Ejemplo   |
|--------------------|-----------|
| Milisegundo        | `Ui8NksP` |
| Décima de segundo (100 ms) | `J2YxUE`  |
| Segundo             | `1u3UkZ`  |
| Hora               | `223a`    |
| Día                | `5Fe`     |

Por defecto, la hora de inicio de la época es la época UNIX tradicional de 1970-01-01. Sin embargo, puedes anular esto para reducir
el tamaño del componente de tiempo.

La siguiente tabla compara IDs generados a partir de la época UNIX vs. una época de 2025-01-01, generados en abril de 2025.

| Granularidad        | Ejemplo Época UNIX (1970) | Ejemplo Época 2025-01 | Reducción de caracteres |
|--------------------|---------------------------|-----------------------|---------------------|
| Milisegundo        | `Ui8NksP`                 | `9YGDBT`              | -1                  |
| Décima de segundo (100 ms) | `J2YxUE`                  | `5vCer`               | -1                  |
| Segundo             | `1u3UkZ`                  | `aifP`                | -2                  |
| Hora               | `223a`                    | `dC`                  | -2                  |
| Día                | `5Fe`                     | `1d`                  | -1                  |

Ten en cuenta que, al comenzar con el componente de tiempo, los IDs generados con el mismo tamaño de tick son ordenables cronológicamente.

### Componente Aleatorio

El componente de tiempo, por sí solo, garantiza la unicidad entre ticks. Sin embargo, para evitar colisiones
*dentro* del mismo tick, añadimos un "componente aleatorio". Simplemente, seleccionamos caracteres aleatorios de un alfabeto dado.

Usando la base-62 predeterminada como ejemplo, cada carácter anexado reduce la probabilidad de colisión en un factor de 62.
Si usamos 5 caracteres aleatorios, eso son `62^5 = 916,132,832` posibilidades únicas. Por lo tanto, para cualquier par de IDs generados en el mismo tick de granularidad, la probabilidad de que
colisionen es 1 en 916,132,832.

Dicho esto, ten en cuenta el [Problema del Cumpleaños](https://en.wikipedia.org/wiki/Birthday_problem) y
el principio del [Palomar](https://en.wikipedia.org/wiki/Pigeonhole_principle).

## Ejemplos de IDs 📗

A continuación se muestran algunos ejemplos de IDs generados con diferentes configuraciones.

| Configuración                                                   | Ejemplo             |
|------------------------------------------------------------|---------------------|
| Base-62, época UNIX, milisegundo, 5 caracteres aleatorios (predeterminado) | `Ui8TX1zB2Avb`      |
| Base-36, época UNIX, milisegundo, 5 caracteres aleatorios           | `m9dw96b1y9gtx`     |
| Base-62, época UNIX, décima de segundo, 5 caracteres aleatorios            | `J2Z31IPSxtl`       |
| Base-62, época 2025-01, décima de segundo, 5 caracteres aleatorios         | `5vHWk2ayCn`        |
| Base-64, época UNIX, hora, 5 caracteres aleatorios                  | `B2TXxM3k1`         |
| Base-16, época UNIX, milisegundo, 6 caracteres aleatorios           | `19628e9e59adc559d` |

## ¿Por qué FlexIDs?

Existen alternativas como UUIDs, NanoIDs, ULIDs, etc., ¿entonces qué ofrecen los FIDs sobre estas?

- **Configurabilidad:** Ajusta finamente la época, el tamaño del tick, el alfabeto y la longitud del sufijo aleatorio para equilibrar con precisión la longitud del ID, la capacidad de ordenamiento y la resistencia a colisiones para tus necesidades específicas.
  - ¿Necesitas IDs cortos para un sistema con una vida útil limitada conocida? Ajusta la época y el tamaño del tick.
  - ¿Necesitas una mayor resistencia a colisiones dentro de un tick? Aumenta la longitud aleatoria.
- **Brevedad:** Al configurar adecuadamente la época y el tamaño del tick, FlexID puede generar IDs significativamente más cortos que alternativas como ULID o UUID, manteniendo la capacidad de ordenamiento cronológico.
- **Simplicidad:** El concepto subyacente (prefijo de tiempo + sufijo aleatorio) es directo y fácil de entender.

## Rendimiento

¡Generar FIDs es muy rápido! No hay estado ni bloqueo: se generarán tan rápido como tu CPU pueda procesar.

En las [pruebas de rendimiento](./benchmarks) en un Apple M2 Pro, obtengo ~240 nanosegundos / operación, o alrededor de 4 a 5 millones de IDs por segundo.

## Contribución 🙏

¡Las contribuciones son bienvenidas! No dudes en abrir un issue o enviar una pull request.

## Licencia 📜

Esta biblioteca está licenciada bajo la [licencia MIT](./LICENSE).

---

Nota: Esta biblioteca anteriormente se conocía como `stid`. Si la usaste, consulta [MIGRATING.md](./docs/MIGRATING.md) sobre cómo migrar a `flexid`.
