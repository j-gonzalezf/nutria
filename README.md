# nutria

Registro personal de alimentación: contador de calorías, macros y (más adelante)
micronutrientes, con búsqueda inversa de alimentos por nutriente.

Uso personal, un solo usuario, coste cero.

## Por qué

Que un contador me diga que voy corto de hierro está bien. Que además me diga
qué puedo comer esta tarde para arreglarlo, está mejor. Ese es el objetivo.

## Estado

En construcción. Nada usable todavía.

## Stack

| Capa | Elección | Motivo |
|---|---|---|
| Base de datos | Supabase (Postgres) | Gratis, Postgres de verdad, auth y RLS incluidos |
| Lógica | Funciones SQL vía RPC | Las agregaciones se resuelven en la BD, no en el navegador |
| Front | Astro + islas | Estático: sin SSR, sin servidor que mantener |
| Hosting | Vercel | Capa gratuita; `main` a producción, `develop` a preview |

El front es un bundle estático que habla directamente con Supabase desde el
navegador. No hay backend propio.

## Datos nutricionales

Todas las fuentes son gratuitas y de acceso abierto. Se importan a nuestra
propia base de datos en lugar de consultarlas en caliente: así la búsqueda por
nutriente puede indexarse, no dependemos de límites de terceros y funciona
sin conexión.

| Fuente | Uso | Notas |
|---|---|---|
| [BEDCA](https://www.bedca.net/) (AESAN) | Principal | Oficial española, ~1.000 alimentos, 39 nutrientes, en español |
| [USDA FoodData Central](https://fdc.nal.usda.gov/) | Relleno | El panel de micronutrientes más completo que hay gratis |
| [CIQUAL](https://ciqual.anses.fr/) (ANSES) | Relleno | Alimentos europeos, buena cobertura de micros |
| [Open Food Facts](https://world.openfoodfacts.org/) | Fase 2 | Productos envasados por código de barras |

### Limitación conocida

Los micronutrientes solo son fiables en alimentos genéricos. La etiqueta legal
de un producto envasado solo obliga a declarar energía, grasas, saturadas,
hidratos, azúcares, proteínas y sal — por eso menos del 20% de las entradas de
Open Food Facts tiene algún micronutriente más allá del sodio.

nutria muestra "sin datos" en lugar de estimar. Un contador que se inventa
números es peor que no tener contador.

## Decisiones de diseño

- **Esquema agnóstico a los nutrientes.** `nutrients` / `foods` /
  `food_nutrients` en lugar de columnas fijas. Empezar por macros es un filtro
  en la interfaz, no una limitación del modelo: añadir micronutrientes será
  cargar filas, sin migraciones.
- **Todo se almacena en gramos.** Las porciones ("1 vaso = 200 ml") son una
  tabla de equivalencias aparte, azúcar de entrada. Un registro nunca guarda
  "1 vaso": guarda 200 g. Cambiar una equivalencia no corrompe el histórico.
- **RLS en todas las tablas, sin excepción.** La clave `anon` viaja en el
  bundle y es pública por diseño. Una tabla sin RLS es una tabla abierta a
  internet.

## Alcance

**Fase 1** — Registro por peso con buscador · Objetivos calculados
(Mifflin-St Jeor, editables) · Registro de peso corporal · Gráficas de
evolución.

**Fase 2** — Recetas y platos compuestos · Panel completo de micronutrientes y
búsqueda inversa · Escáner de códigos de barras.

## Ramas

- `main` — producción.
- `develop` — desarrollo, se despliega solo en una URL de preview.

```bash
# día a día
git checkout develop && git push

# publicar
git checkout main && git merge develop && git push && git checkout develop
```
