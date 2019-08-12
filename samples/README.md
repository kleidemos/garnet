# Samples | Примеры

## Flocking | _Стайность?_

[[Code](https://github.com/bcarruthers/garnet/blob/master/samples/Garnet.Samples.Common/Flocking.fs)]

> This sample demonstrates classic boids-style flocking or steering, with behaviors tuned so agents form clusters. 
> Neighbor forces are calculated in brute-force fashion without use of spatial structures. 
> The code is organized to minimize direct dependency on ECS and MonoGame.

Данный пример демонстрирует классический Boids алгоритм поведения стай, при котором агенты стремятся образовать кластер.
Поведение групп вычисляется методом перебора без использования пространственных структур.
Код организован таким образом, чтобы минимизировать зависимости от ECS и MonoGame.

## Strategy | Стратегия

[[Code](https://github.com/bcarruthers/garnet/blob/master/samples/Garnet.Samples.Common/Strategy.fs)]

> Strategy games like Civilization have a world map consisting of grid cells. 
> This sample demonstrates use of 2D location as a custom entity key for storing components in a world grid.

Стратегические игры, такие как цивилизации имеют карту мира состоящую из клеток.
В этом примере демонстрируется использование 2D-координат локации в качестве ключа сущности для хранения компонентов в мировой сетке.

> Of course, you could also forego ECS entirely and simply use an array of cells for your map, implementing your own logic for staged changes or sparse storage if needed. 
> The benefit of using ECS is having those features available in a uniform way, especially for larger maps with sparse components.

Конечно, вы можете вообще отказаться от ECS и просто использовать массив ячеек для своей карты, реализуя свою собственную логику поэтапных изменений, разряженного хранилища и т.п., если это необходимо.
Основным профитом от использования ECS является его единообразность, особенно в случае больших карт с небольшим числом компонентов.

### World grid storage | Хранилище мировой сетки

> First, we define a 2D location type and use it to identify both cells and segments. 
> The underlying component storage is based on 64-element segments (which can also be called pages or chunks), so we can define a mapping from cells to 8x8 cell segments. 
> Note that unlike entity IDs, locations don't need to be created or recycled.

Во-первых, мы определяем тип двухмерных координат и используем его для идентификации как ячеек, так и сегментов _(чтобы последнее не значило)_.
Базовое хранилище компонентов основано на 64-элементных сегментах (которые также могут называться страницами или чанками), поэтому мы можем определить сопоставление из ячеек в 8*8 ячейки сегментов.
Обратите внимание, что в отличие от идентификаторов, локации не требуют создания или переиспользования.

```fsharp
[<Struct>]
type Loc = { x : int; y : int }
    with 
        override c.ToString() = 
            sprintf "(%d, %d)" c.x c.y

module Loc =
    // > Components are stored in size 64 segments, so
    // > we need to define a mapping from component keys
    // > to tuple of (segment key, index within segment)
    // Компоненты хранятся в сегментах размером 64,
    // поэтому нам надо определить маппинг из ключей компонентов
    // в кортеж из (ключа сегмента, индекса в сегменте)
    // _если правильно понял, то весь сыр-бор из-за экономии_
    // _и детально частично описывается в EntityId и Partitioning_
    let toKeyPair p = 
        struct(
            { x = p.x >>> 3; y = p.y >>> 3 }, 
            ((p.y &&& 7) <<< 3) ||| (p.x &&& 7))

type WorldGrid() = 
    let store = ComponentStore<Loc, Loc>(Loc.toKeyPair)
```

> You can iterate over world cells similarly to entities. 
> However, to use location you would need to explicitly store it as a component or define join operations that implicitly calculate it on the fly.

Вы можете проитерировать клетки мира аналогично сущностям.
Однако для использования координат вам может понадобиться явно хранить их как компонент или определить операции объединения, которые неявно будут вычислять его на лету.

```fsharp
// > increase the temperature of volcano cells by
// > joining on cell components in the map
// увеличиваем температуру от вулканических клеток
// через присоединение клетки компонента на карте
fun param struct(cl : Climate, p : Loc, _ : Volcano) ->
    { cl with temperature = cl.temperature + 1 }
|> Join.update3
|> Join.over map)
```

### Units | Юниты

> You can store units as entities:

Вы можете хранить юниты как сущности:

```fsharp
let entity =
    c.Create()
        .With(Archer)
        .With({ x = 10; y = 20 })
        .With({ unitSize = 5 })                
```

> However, for efficient world map queries, you may want to maintain a reference to units in the world map grid, along with any other components or markers useful for querying:

Однако для эффективных запросов к карте мира может быть необходимо сохранять ссылки на юниты на мировой карте, а также любые другие компоненты или маркеры полезные для запросов:

```fsharp
map.Get(loc).Add { unitEid = entity.id }
```

### Inspecting | Проверка / просмотр ?

> Like entities, you can also print the content of grid cells:

Как и сущнсоти, вы можете также вывести содержимое каждой клетки:

```fsharp
printfn "%s" <| 
    c.GetInstance<WorldGrid>()
        .Get({ x = 10; y = 15 })
        .ToString()
```
``` 
Entity (10, 15): 12 bytes
Climate {tempurature = 79;
 humidity = 60;}
Terrain Grassland
```

> Printing entire container:

Вывод всего контейнера:

```fsharp
printfn "%s" <| c.ToString()
```
```
Types
  Channels
    [None]
      PrintMap: 1H 0P 0E 1T 0SE
      ResetMap: 1H 0P 0E 1T 0SE
      StepSim: 1H 0P 0E 1T 0SE
    Garnet.Ecs
      Commit: 1H 0P 0E 0T 1SE
  Coroutines
    Stack: 0 pending, 0 active, frames: 
    Timed: 0 total, due: 0
  SegmentStore<Int32>
    Eid: 50/1C 
      C0 1 xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx..............
    Loc: 50/1C 
      C0 1 xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx..............
    UnitType: 50/1C 
      C0 1 xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx..............
    UnitSize: 50/1C 
      C0 1 xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx..............
  Pools
    0: 50C 0T 0P 0R
  SegmentStore<Loc>
    Climate: 400/9C 
      C0 (0, 0) xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
      C1 (0, 1) xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    Ore: 43/8C 
      C0 (0, 0) ..x.....x.....................xx......x..x........x.....x...x...
      C1 (0, 1) ......x.....x................x....x....x.....x.....x.x...x......
    Terrain: 400/9C 
      C0 (0, 0) xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
      C1 (0, 1) xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    City: 16/6C 
      C0 (0, 0) .........x......................x.....x.........................
      C1 (0, 1) .x................x.................x.x.........................
    Occupant: 48/9C 
      C0 (0, 0) ..x....................x...x..........x.....xx.........x........
      C1 (0, 1) x......x.........x....x.x..........x......x.........xx......x...
```

