# Garnet | Garnet

[![Build status](https://ci.appveyor.com/api/projects/status/g82kak7btxp48rnd?svg=true)](https://ci.appveyor.com/project/bcarruthers/garnet)

[NuGet package](https://www.nuget.org/packages/Garnet/)

Garnet is a lightweight game composition library for F# with entity-component-system (ECS) and actor-like messaging features.

_TODO Перевести на русский:_
Garnet - легковесная библиотека для сборки игр на F# с entity-component-system (ECS) и актороподобными возможностями обработки сообщений.

```fsharp
open Garnet.Composition

// > events
// события
[<Struct>] type Update = { dt : float32 }

// > components
// компоненты
[<Struct>] type Position = { x : float32; y : float32 }
[<Struct>] type Velocity = { vx : float32; vy : float32 }

// > create a world
// создаем мир
let world = Container()

// > register a system that updates position
// регистрируем систему обновляющую позицию
let system =
    world.On<Update> (
        fun e struct(p : Position, v : Velocity) -> {
            x = p.x + v.vx * e.dt
            y = p.y + v.vy * e.dt
            }
        |> Join.update2
        |> Join.over world)

// > add an entity to world
// добавляем сущность мира
let entity = 
    world.Create()
        .With({ x = 10.0f; y = 5.0f })
        .With({ vx = 1.0f; vy = 2.0f })

// > run updates and print world state
// запускаем обновления и вывод состояния мира
for i = 1 to 10 do
    world.Run <| { dt = 0.1f }
    printfn "%O\n\n%O\n\n" world entity
```

## Table of contents | Содержание

> * Introduction
>     * [Background](#background)
>     * [Goals](#goals)
>     * [Building](#building)
>     * [Samples](#samples)
> * Guide
>     * [Containers](#containers)
>     * [Entities](#entities)
>     * [Components](#components)
>     * [Systems](#systems)
>     * [Actors](#actors)
>     * [Integration](#integration)
> * [Roadmap](#roadmap)
> * [FAQ](#faq)
> * [License](#license)
> * [Maintainers](#maintainers)

* Введение
	* [Предпосылки](#background)
	* [Цели](#goals)
	* [Сборка](#building)
	* [Примеры](#samples)
* Guide
    * [Контейнеры](#containers)
    * [Сущности](#entities)
    * [Компоненты](#components)
    * [Системы](#systems)
    * [Акторы](#actors)
    * [Интеграция](#integration)
* [Дорожная карта](#roadmap)
* [FAQ](#faq)
* [Лицензия](#license)
* [Maintainers](#maintainers)

## Getting started

> 1. Create either a .NET Framework or .NET Core application.
> 2. Reference the [Garnet NuGet package](https://www.nuget.org/packages/Garnet/). 
> 3. See code samples for library usage.

1. Создайте приложение .NET или .NET Core.
2. Подключите [NuGet-пакет Garnet](https://www.nuget.org/packages/Garnet/). 
3. Ориентируйтесь на примеры использования библиотеки.

> Note that for .NET Core, if you encounter warnings about FSharp.Core package downgrade, you may need to add an explicit reference in your .fsproj file to the newer FSharp.Core version: 

Обратите внимание, в случае .NET Core, если вы получите предупреждение об ~даунгрейде FSharp.Core, вам потребуется добавить ссылку на новую версию FSharp.Core в .fsproj явно.

     <PackageReference Include="FSharp.Core" Version="4.6.2" />

## Background | Предпосылки

> Garnet emerged from [Triverse](http://cragwind.com/blog/posts/grid-projectiles/), a 2D game under development where players build and command drone fleets in a large tri-grid world. 
> The game serves as a performance testbed and ensures the library meets the actual needs of at least one moderately complex game.

Garnet появлися из [Triverse](http://cragwind.com/blog/posts/grid-projectiles/), 2D игры находящейся в стадии разработки, в которой игрки строят и управляют беспилотными флотами в большом _трехмерном?_ мире.
Данная игра также служит полигоном для Garnet, что обеспечивает соотвествие библиотеки реальным потребностям как минимум одной умерено сложной игры.

> ECS is a common architecture for games, often contrasted with OOP inheritance. 
> It focuses on separation of data and behavior and is typically implemented in a data-oriented way to achieve high performance. 
> It's similar to a database, where component tables are related using a common entity ID, allowing systems to query and iterate over entities with specific combinations of components present. 
> EC (entity-component) is a related approach that attaches behavior to components and avoids systems.

ECS - это широко распространенная для игр архитектура, часто контрастирующая с наследованием в ООП.
Она фокусируется на разделении данных и поведения, и как правило реализуется с упором на данных для достижения высокой производительности.
ECS похожа на базу данных, где таблицы компонентов связаны посредством общего идентификатора сущности, что позволяет системам запрашивать и перебирать сущности с определенными комбинациями компонентов.
EC (entity-component) - связанный с ECS подход, который прикрепляет поведение к компонентам и сторонится систем.

> While ECS focuses on managing shared state, the actor model isolates state into separate actors which communicate only through messages. 
> Actors can send and receive messages, change their behavior as a result of messages, and create new actors. 
> This approach offers scaleability and an abstraction layer over message delivery, and games can use it at a high level to model independent processes, worlds, or agents.

В то время как ECS фокусируется на управлении распределенным состоянием, акторная модель напротив изолирует состояние в отдельных акторах, которые взаимодействуют друг с другом посредством сообщений.
Акторы могут отправлять и получать сообшений, изменять их состояния в зависимости от сообщений, а также создавать новые акторы.
Данный подход предлагает масштабируемость и слой асбтракций над передачей сообщений. Игры могут использовать его как высокоуровневый интсрумент моделирования независимых процессов, миров или агентов.

## Goals | Цели

> - **Lightweight**: Garnet is essentially a simplified in-memory database and messaging system suitable for games. 
> No inheritance, attributes, or interface implementations are required in your code. 
> It's more of a library than a framework or engine, and most of your code shouldn't depend on it.

> - **Fast**: Garbage collection spikes can cause dropped frames and inconsistent performance, so Garnet minimizes allocations and helps library users do so too. 
> Component storage is data-oriented for fast iteration.

> - **Minimal**: The core library focuses on events, scheduling, and storage, and anything game-specific like physics, rendering, or update loops should be implemented separately.

> - **Complete**: In addition to traditional ECS, Garnet provides actor-like messaging for scenarios where multiple ECS worlds are beneficial, such as AI agents or networking.

- **Легкость**: Garnet по сути своей является упрощенной in-memory базой данных и системой сообщений подходящими (или даже адаптированными) под игры.
Ваш код не требует какого-либо наследования, атрибутов или реализаций интерфейса.
Это больше чем библиотека, скорее фреймворк или движок, и большая часть вашего кода не должна зависеть от него.

- **Скорость**: Пики в ходе работы сборщика мусора могут быть причиной потери фреймов и изменчивой производительности, поэтому Garnet минимизирует выделение памяти и помогает в этом пользователям библиотеки.
Хранение компонентов ориентированно на данные для ускоренного перебора.

- **Минимальность**: Основная библиотека фокусируется на событиях, планировании и хранении, а все что зависит от конкретных игр, типа физики, рендеринга или цикла обновлений должно быть реализовано отдельно.

- **Достаточность**: В дополнение к традиционному ECS, Garnet предоставляет акторную модель сообщений для сценариев, где может быть полезно множество ECS миров, например ИИ или сеть.

## Building | Сборка

> 1. Install [.NET Core SDK](https://dotnet.microsoft.com/download) (2.1 or later)
> 1. Install [VS or build tools](https://visualstudio.microsoft.com/downloads) (2017 or later)
> 1. Run build.cmd

1. Установите [.NET Core SDK](https://dotnet.microsoft.com/download) (2.1 или выше)
1. Установите [VS или инструменты сборки](https://visualstudio.microsoft.com/downloads) (2017 или выше)
1. Запустите build.cmd

## Samples | Примеры

> Sample documentation [here](https://github.com/bcarruthers/garnet/tree/master/samples).

Документация по примерам [здесь](https://github.com/bcarruthers/garnet/tree/master/samples).

## Containers | Контейнеры

> ECS containers provide a useful bundle of functionality for working with shared game state, including event handling, component storage, entity ID generation, coroutine scheduling, and resource resolution.

Контейнеры ECS обеспечивают полезный набор функциональностей для работы с совместно используемым состояним игры, включая обработку событий, хранение компонентов, генерацию идентификаторов сущностей, планирование корутин (или сопрограмм), и управление жизненным циклом ресурсов.

```fsharp
// > create a container/world
// создание контейнера/мира
let c = Container()
```

### Registry | Регистр / Реестр

> Containers store single instances of types such as component lists, ID pools, settings, and any other arbitrary type. 
> You can access instances by type, with optional lazy resolution.
> This is the service locator (anti-)pattern.

Контейнеры хранят отдельные экземпляры типов, таких как списки компонентов, пулы идентификаторов, настройки и любые другие произвольные типы.
Вы можете получить доступ к этим экземплярам через их тип, с ленивым разрешением при необходимости.
Это сервис локатор, анти-паттерн.

```fsharp
// > option 1: add specific instance
// вариант 1: добавляем  конкретный экземпляр
c.RegisterInstance(defaultWorldSettings)
// > option 2: register a factory
// вариант 2: регистрируем фабрику
c.Register(fun () -> defaultWorldSettings)
// > resolve type
// получаем экземпляр по типу
let settings = c.GetInstance<WorldSettings>()
```

> To access value types you can use a reference cell:

Для доступа к value-типам, вы можете использовать ссылочные ячейки:

```fsharp
c.Register(fun () -> ref { zoomLevel = 0.5f })
let zoom = c.GetInstance<ref<Zoom>>()
```

### Object pooling | Объектный пул ?

> Avoiding GC generally amounts to use of structs, pooling, and avoiding closures. 
> Almost all objects are either pooled within a container or on the stack, so there's little or no GC impact or allocation once maximum load is reached. 
> If needed, warming up or provisioning buffers ahead of time is possible for avoiding GC entirely during gameplay.

Чтобы не попасть под раздачу от GC как правило используют структуры, пулинг и устранение замыканий.
Почти все объекты либо объединены в контейнер, либо находятся на стеке, поэтому влияние GC минимально или вовсе отсутствует даже при достижении максимальных нагрузок.
При необходимости, буферы можно разгореть или подготовить заранее, что позволит избежать влияние GC во время игры.

### Commits | Фиксации

> Certain operations on containers, such as sending events or adding/removing components, are staged until a commit occurs, allowing any running event handlers to observe the original state. 
> Commits occur automatically after all subscribers have completed handling a list of events, so you typically shouldn't need to explicitly commit.

Некоторые операции над контейнерами, такие как отправка событий или добавление/удаление компонентов, выполняются поэтапно, позволяя всем запущенным обработчикам событий наблюдать изначальное состояние.
Фиксация произойдет автоматически после того, как все подписчики завершат обработку списка событий, поэтому вам обычно не требуется явно коммитить.

```fsharp
// > create an entity
// создаем сущность
let e = c.Create().With("test")
// > not yet visible
// здесь еще не видима
c.Commit()
// > now visible
// а здесь видима
```

## Entities | Сущности

> An entity is any identifiable thing in your game which you can attach components to. 
> At minimum, an entity consists only of an entity ID.

Сущность - это любая идентифицируемая вещь в вашей игре, и к ней вы можете прикрепить компоненты.
В вырожденном случае, сущность содержит только ее собственный идентификатор.

### Entity ID | Идентификатор сущности

> Entity IDs are 32 bits and stored in a component list. 
> This means they can be accessed and iterated over like any other component type without special handling. 
> IDs use a special Eid type rather than a raw int32, which offers better type safety but means you need a direct dependency on Garnet if you want to define types with an Eid (or you can manage converting to your own ID type if this is an issue). 

Идентификаторы сущностей представляют из себя 32 бита, которые хранятся в списке компонентов.
Это означает, что они могут быть доступны и подвергнуты перебору как любой другой тип компонентов без специальной обработки.
Идентификаторы используют специальный тип `Eid`, а не нативный `int32`, что обеспечивают лучшую типобезопасность. С другой стороны, это приводит к зависимости ваших типов от Garnet, если вы хотите определить тип с `Eid`. Или вы можете определить преобразование из вашего собственного типа ID и обратно, если это является проблемой

```fsharp
let entity = c.Create()
printfn "%A" entity.id
```

### Generations | Поколения

> A portion of an ID is dedicated to its generation number. 
> The purpose of a generation is to avoid reusing IDs while still allowing buffer slots to be reused, keeping components stored as densely as possible.

Часть идентификатора зависит от номера его поколения.
Концепция поколений позволяет избежать повторного использования идентификаторов, но сохраняет возможность повторного использования буферных слотов, тем самым храня компоненты как можно плотнее.

### Partitioning | Разбиения ?

> Component storage could become inefficient if it grows too sparse (i.e. the average number of occupied elements per segment becomes low). 
> If this is a concern (or you just want to organize your entities), you can optionally use partitions to specify a high bit mask in ID generation. 
> For example, if ship and bullet entities shared the same ID space, they may become mixed over time and the ship components would become sparse. 
> Instead, with separate partitions, both entities would remain dense. 
> Note: this will likely be replaced with groups in the future.

Хранилище компонентов может стать неэффективным, если оно становится слишком разреженным (т.е. средне число занятых элементов в сегменте становится низким).
Если это вызывает беспокойство (или вам просто потребуется организовать свои сущности), вы можете дополнительно указать разбиение (раздел), чтобы определить маску старших битов в поколении идентификаторов.
Например, если сущности кораблей и пуль делят одно и тоже пространство идентификаторов, они могут перемешаться через некоторое время и компоненты кораблей станут очень разреженными.
Вместо этого можно разделить их сегменты и обе сущности сохранят плотность.
Примечание: вероятно, что в будущем этот функционал будет заменен группами.

### Generic storage | Общее хранилище

> Storage should work well for both sequential and sparse data and support generic key types. 
> Entity IDs are typically used as keys, but other types like grid location should be possible as well.

Хранилище должно хорошо работать и с последовательными, и с разреженными данными и поддерживать общие типы ключей.
Обычно в качестве ключей используются идентификаторы сущностей, но и другие типы, такие как позиции в сетке также могут использоваться.

### Inspecting | Проверка ? 

> You can print the components of an entity at any time, which is useful in REPL scenarios as an alternative to using a debugger.

Вы можете вывести компоненты сущностей в любой момент, что может быть полезно в REPL как альтернатива дебаггеру.

```fsharp
printfn "%s" <| c.Get(Eid 64).ToString()
```
```
Entity 0x40: 20 bytes
Eid 0x40
Loc {x = 10;
 y = 2;}
UnitType Archer
UnitSize {unitSize = 5;}
```

## Components | Компоненты

> Components are any arbitrary data type associated with an entity. 
> Combined with systems that operate on them, components provide a way to specify behavior or capabilities of entities.

Компонентом может быть любой тип данных асоциированный с сущностью.
В сочетании с системами, которые ими оперируют, компоненты обеспечивают способ настройки поведения или возможностей сущности.

### Data types | Типы данных

> Components should ideally be pure data rather than classes with behavior and dependencies. 
> They should typically be structs to avoid jumping around in memory or incurring allocations and garbage collection. 
> Structs should almost always be immutable, but mutable structs (with their gotchas) are possible too.

В идеале компоненты должны представлять чистые данные, в отличии от классов снабженны-х поведением и зависимостями.
Как правило они являются структурами, дабы избежать скачков памяти в ходе ее выделения или сборки мусора.
Структуры почти всегда должны быть иммутабельными, но мутабельность структур (со всеми вытекающими) также возможна.

```fsharp
[<Struct>] type Position = { x : float32; y : float32 }
[<Struct>] type Velocity = { vx : float32; vy : float32 }

// > create an entity and add two components to it
// создаем сущность и добавляем к ней два компонента 
let entity = 
    c.Create()
        .With({ x = 10.0f; y = 5.0f })
        .With({ vx = 1.0f; vy = 2.0f })
```

### Storage | Хранилище

> Components are stored in 64-element segments with a mask, ordered by ID. 
> This provides CPU-friendly iteration over densely stored data while retaining some benefits of sparse storage. 
> Some ECS implementations provide a variety of specialized data structures, but Garnet attempts a middle ground that works moderately well for both sequential entity IDs and sparse keys such as grid locations.

Компоненты хранятся в 64-элементных сегментах с маской, упорядоченной по идентификатору.
Что позволяет процессору удобно перемещаться по плотно упакованным данным все еще сохраняя некоторые преимущества разреженного хранилища.
Некоторые реализации ECS предоставляют различные специализированные структуры данных, но Garnet пытается достичь золотой серидины, обеспечивая умеренно хорошую производительность как для последовательных идентификаторов, так и для разреженных ключей, как _сетки позиций_.

> Only a single component of a type is allowed per entity, but there is no hard limit on the total number of different component types used (i.e. there is no fixed-size mask defining which components an entity has).

К каждой сущности может быть прикреплен только один экземпляр компонентов конкретного типа. Но не существует жесткого ограничения на общее количество различных типов компонентов, которые могут быть ассоциированы с конкретной сущностью.

### Iteration | Перебор или итерация

> You can iterate over entities with specific combinations of components using joins. 
> In this way you could define a system that updates all entities with a position and velocity, and iteration would skip over any entities with only a position and not velocity. 
> Currently only a fixed set of predefined joins are provided rather than allowing arbitrary queries.

Вы можете перебирать сущности с определенными комбинациями компонентов, используя соеднинения (джойны).
Таким образом, вы можете определить систему, которая обновляет все объекты, которые имеют позицию и скорость, а в ходе итерации будут проигнорированы все объекты имеющие только позицию без скорости.
В данный момент предоставляется ограниченный набор предопределенных джойнов. Произвольные запросы не поддерживаются.

```fsharp
let runIter =
    // > first define an iteration callback:
    // > (1) param can be any type or just ignored
    // > (2) use struct record for component types
    // сперва определяем колбек итерации:
    // (1) param может быть любого типа или просто игнорируется
    // (2) используем тупл-структуру для сложных типов
    fun param struct(eid : Eid, p : Position, h : Health) ->
        if h.hp <= 0 then 
            // > [start animation at position]
            // > destroy entity
            // [начинаем анимацию в позиции]
            // уничтожаем сущность
            c.Destroy(eid)
    // > iterate over all entities with all components
    // > present (inner join)
    // перебираем все сущности у которых есть все компоненты
    // (inner join)
    |> Join.iter3
    // > iterate over container
    // указываем компонент по которому будет осуществляться перебор/итерация
    |> Join.over c
let healthSub =
    c.On<DestroyZeroHealth> <| fun e ->
        runIter()
```

### Adding | Добавление

> Additions are deferred until a commit occurs, so any code dependent on those operations completing needs to be implemented as a coroutine.

Добавление будет отложено, пока не будет выполнена фиксация, поэтому любой код зависящий от этой операции должен быть реализован как сопрограмма.

```fsharp
let e = c.Get(Eid 100)
e.Add<Position> { x = 1.0f; y = 2.0f }
// > change not yet visible
// изменения еще не видны
```

### Removing | Удаление

> Like additions, removals are also deferred until commit. 
> Note that you can repeatedly add and remove components for the same entity ID before a commit if needed.

Аналогично добавлениею, удаление также откладывается до фиксации.
Обратите внимание, что вы можете при необходимости повторно добовлять и удалять компоненты для одной и той же сущности перед фиксацией.

```fsharp
e.Remove<Velocity>()
// > change not yet visible
// изменения еще не видны
```

### Updating | Обновление 

> Unlike additions and removals, updating/replacing an existing component can be done directly at the risk of affecting subsequent subscribers. 
> This way is convenient if the update operation is commutative or there are no other subscribers writing to the same component type during the same event. 
> You can alternately just use addition if you don't know whether a component is already present.

В отличиии от добавления и удаления, обновление и замена существующего компонента может производиться напрямую с риском воздействия на последующих подписчиков.
Это удобно в случае комутативности операции или отсутствия других подписчиков на тот же тип в рамках одного и того же события.
Вы можете использовать добавление для теъ случаев, когда вам не известно, существует компонент или нет.

```fsharp
let e = c.Get(Eid 100)
e.Set<Position> { x = 1.0f; y = 2.0f }
// > change immediately visible
// изменение видны моментально
```

### Markers | Маркеры

> You can define empty types for use as flags or markers, in which case only 64-bit masks need to be stored per segment. 
> Markers are an efficient way to define static groups for querying.

Вы можете определить пустой тип в качестве флага или маркера, в этом случае в сегменте будут храниться только 64-битные маски.
Маркеры являются эффективным способом определения статический групп для запросов.

```fsharp
type PowerupMarker = struct end
```

_// Примечание переводчика: обратите внимание, что при `struct end` появляется дефолтный конструктор._

## Systems | Системы

> Systems are essentially event subscribers with an optional name. 
> System event handlers often iterate over entities, such as updating position based on velocity, but they can do any other kind of processing too. 
> Giving a system a name allows hot reloading.

Системы по сути своей являются подписчиками событий с опциональным названием.
Обработчики событий систем часто перебирают объекты, например для обновления позиций в соответсвии со скоростью, но они также могут выполнять и другие виды обработки.
Присвоение системе имени позволяет перезагружать ее по горячему (_оставить hot reload?_).

```fsharp
module MovementSystem =     
    // > separate methods as needed
    // выделяем метод при необходимости
    let registerUpdate (c : Container) =
        c.On<UpdatePositions> <| fun e ->
            printfn "%A" e

    // > combine all together
    // собираем все вместе
    let definition =
        // > give a name so we can hot reload
        // устанавливаем имя, чтобы мы могли перезагрузить систему
        Registration.listNamed "Movement" [
            registerUpdate
            ]
```

### Execution | Исполнение

> When any code creates or modifies entities, sends events, or starts coroutines, it's only staging those things. 
> To actually set all of it into motion, you need to run the container, which would typically happen as part of the game loop. 
> Each time you run the container, it commits all changes, publishes events, and advances coroutines, repeating this process until no work remains to do (so you should avoid introducing cycles unless they are part of a timed coroutine).

Когда какой-либо код создает или изменяет сущности, инициирует события или запускает сопрограммы, он в действительности лишь декларирует эти изменения.
Чтобы привести все в движение, необходимо запустить контейнер, что как правило происходит как часть цикла игры.
Каждый раз, когда вы запускаете контейнер, он производит все изменения, публикует события и запускает сопрограммы, повторяя эту операцию, пока вся работа не будет выполнена (поэтому вы должны избегать циклов, кроме тех случаев, где они являются часть синхронизированной программы).

```fsharp
// > run the container
// запускаем контейнер
c.Run()
```

### Events | События

> Like components, you can use any arbitrary type for an event, but structs are generally preferable to avoid GC. 
> When events are published, subscribers receive batches of events with no guaranteed ordering among the subscribers or event types. 
> Any additional events raised during event handling are run after all the original event handlers complete, thereby avoiding any possibility of reentrancy but complicating synchronous behavior. 

Как и в случаае компонентов, вы можете использовать любой тип в качестве события, но структуры обычно предпочтительнее для работы с GC.
Когда события публикуются, подписчики получают пачки событий без каких-либо гарантий порядка подписчиков или ивентов.
Любой дополнительный ивент, запущенный в ходе обработки текущего события будет запущен после того, как будет обработано исходное событие. Этим исключается повторный вход, но усложняется синхронное поведение.

```fsharp
[<Struct>] type UpdateTime = { dt : float32 }

// > call sub.Dispose() to unsubscribe
// вызывите sub.Dispose(), чтобы отписаться от события
let sub =
    c.On<UpdateTime> <| fun e ->
        // > [do update here]
        // [производим обновление]
        printfn "%A" e

// > send event
// инициируем событие
c.Send { dt = 0.1f }
```

> Events intentionally decouple publishers and subscribers, and since dispatching events is typically not synchronous within the ECS, it can be difficult to trace the source of events when something goes wrong (no callstack).

События намеренно отрывают от _публикаторов?_ и подписчиков. Так как в ECS отправка событий не синхронна, то могут возникать трудности при попытке отследить источник событий, когда что-то пойдет не так (отсутсвует стек вызовов).

### Coroutines | Корутины / Сопрограммы

> Coroutines allow capturing state and continuing processing for longer than the handling of a single event. 
> They are implemented as sequences and can be used to achieve synchronous behavior despite the asynchronous nature of event handling. 
> This is one of the few parts of the code which incurs allocation.

Сопрограммы позволяют захватывать состояние и продолжать обработку гораздо дольше, чем обраотка одного события.
Они реализованы как последовательности и могут быть использованы для получения синхронного поведения, вопреки асинхронной природе обработки событий.
Это одна из немногих частей кода, которая производит аллокации.

> Coroutines run until they encounter a yield statement, which can tell the coroutine scheduler to either wait for a time duration or to wait until all nested processing has completed. 
> Nested processing refers to any coroutines created as a result of events sent by the current coroutine, allowing a stack-like flow and ordering of events.

Сопрограммы запускаются до тех пор, пока не встретят оператор `yield`, который говорит планировщику либо подождать некоторое время, либо подождать, пока будут выполнены все вложенные обработки.
Вложенными обработчиками являются все сопрограммы созданные как результат событий инициированные текущей сопрограммой. Тем самым создается подобие стека и очередности событий.

```fsharp
let system =
    c.On<Msg> <| fun e ->
        printf "2 "

// > start a coroutine
// запускаем сопрограмму
c.Start <| seq {
    printf "1 "
    // > send message and defer execution until all messages and
    // > coroutines created as a result of this have completed
    // отправляем сообщение и ждем, пока все сообщения
    // и подпрограммы созданные из-за данного сообщения будут выполнены
    c.Send <| Msg()
    yield Wait.defer
    printf "3 "
    }

// > run until completion
// > output: 1 2 3
// запускаем и ждем завершения
// вывод: 1 2 3
c.Run()
```

> Time-based coroutines are useful for animations or delayed effects. 
> You can use any unit of time as long as it's consistent.

Основанные на времени сопрограммы полезны для анимации или отложенных эффектов.
_Вы можете использовать любые единицы времени, пока они согласованы. ?_

```fsharp
// > start a coroutine
// запускаем сопрограмму
c.Start <| seq {
    for i = 1 to 5 do
        printf "[%d] " i
        // > yield execution until time units pass
        // ждем, пока не придет достаточное число единиц
        yield Wait.time 3
    }

// > run update loop
// > output: [1] 1 2 3 [2] 4 5 6 [3] 7 8 9
// запускаем циклы обновления
// вывод: [1] 1 2 3 [2] 4 5 6 [3] 7 8 9 
// _?кажись в конце еще должно быть [4]?_
for i = 1 to 9 do
    // > increment time units and run pending coroutines
    // увеличиваем время и запускаем корутины
    c.Step 1
    c.Run()
    printf "%d " i
```

### Multithreading | Многопоточность

> It's often useful to run physics in parallel with other processing that doesn't depend on its output, but the event system currently has no built-in features to facilitate multiple threads reading or writing. 
> Instead, you can use the actor system for parallel execution at a higher level, or you can implement your own multithreading at the container level.

Часто бывает необходимо запускать физику параллельно с другой обработкой, которая не зависит от ее вывода, но система событий в данные момент не имеет встроенных функций помогающих многопоточному вводу/выводу.
Вместо этого, вы можете либо использовать акторную систему для параллельного выполнения как высокоуровневое решение, либо самостоятельно реализовать многопоточность на уровне контейнера.

### Event ordering | Порядок событий

> For systems that subscribe to the same event and access the same resources or components, you need to consider whether one is dependent on the other and should run first.

Для систем, которые подписываются на одно и то же событие, имеют доступ к одному и тому же ресурсу или компоненту, необходимо решить, зависит ли системы друг от друга и кому из них выполняться первой.

> One way to guarantee ordering is to define individual sub-events for the systems and publish those events in the desired order as part of a coroutine started from the original event (with waits following each event to ensure all subscribers are run before proceeding).

Одним из способов гарантировать порядок, это определение вложенных событий и публикация этих событий в необходимом порядке как часть сопрограммы вызванной из изначального события. В этом случае необходимо ждать завершения после каждого события, чтобы гарантировать, что все подписчики будут запущены перед продолжением.

```fsharp
// > events
// события
type Update = struct end
type UpdatePhysicsBodies = struct end
type UpdateHashSpace = struct end

// > systems
// системы
let updateSystem =
    c.On<Update> <| fun e -> 
        c.Start <| seq {
            // > sending and suspending execution to 
            // > achieve ordering of sub-updates
            // публикация и приостановка выполнения
            // для достижения порядка вложенных обновлений
            c.Send <| UpdatePhysicsBodies()
            yield Wait.defer
            c.Send <| UpdateHashSpace()
            yield Wait.defer
        }
let system1 = 
    c.On<UpdatePhysicsBodies> <| fun e ->
        // > [update positions]
        // [обновляем позиции]
        printfn "%A" e
let system2 = 
    c.On<UpdateHashSpace> <| fun e ->
        // > [update hash space from positions]
        // [обновляем хеш пространство от позиций]
        printfn "%A" e
```

### Composing systems | Компоновка система

> Since systems are just named event subscriptions, you can compose them into larger systems. 
> This allows for bundling related functionality.

Так как сисетмы являются лишь именнованными подписками на события, их можно объединять в более крупные системы.
Что позволяет связывать _родственную_ функциональность.

```fsharp
module CoreSystems =        
    let definition =
        Registration.combine [
            MovementSystem.definition
            HashSpaceSystem.definition
        ]
```

## Actors | Акторы

> While ECS containers provide a simple and fast means of storing and updating shared memory state using a single thread, actors share no common state and communicate only through messages, making them suitable for parallel processing.

В то время как контейнеры ECS предоставляют простые и быстре средства для хранения и изменения общего состояния в одном потоке, акторы не имеют общего состояния и взаимодействую только через сообщения, что делает их максимально подходящими для паралелльной обработки.

### Definitions | Определения

> Actors are identified by an actor ID. 
> They are statically defined and created on demand when a message is sent to a nonexistent actor ID. 
> At that point, an actor consisting of a message handler is created based on any definitions registered in the actor system that match the actor ID. 
> It's closer to a mailbox processor than a complete actor model since these actors can't dynamically create arbitrary actors or control actor lifetimes.

Акторы имеют свой собственный иидентификатор.
Они определяются статически и создаются по требованию, когда сообщение отправляется несуществующему актору.
В этот момент актор состоящий из обработчика сообщений, создается на основе любых определений, зарегистрированных в акторной системе по данной идентифиатору.
Garnet-реализация ближе к мейлбоксу, чем к полной модели акторов, поскольку эти акторы не могут динамически создавать произвольные акторы или управлять их временем жизни.

```fsharp
// > message types
// типы сообщений
type Ping = struct end
type Pong = struct end

// > actor definitions
// определения акторов
let a = new ActorSystem()
a.Register(ActorId 1, fun c ->
    c.On<Ping> <| fun e -> 
        printf "ping "
        c.Respond(Pong())
    )
a.Register(ActorId 2, fun c ->
    c.On<Pong> <| fun e -> 
        printf "pong "
    )
    
// > send a message and run until all complete
// > output: ping pong
// отправляем сообщение и ждем завершения
// вывод: ping pong 
a.Send(ActorId 1, Ping(), sourceId = ActorId 2)
a.RunAll()
```

### Actor messages versus container events | Акторые сообщения и события контейнеров

> "Events" and "messages" are often used interchangeably, but here we use separate terms to distinguish container 'events' from actor 'messages'. 
> Containers already have their own internal event system, but the semantics are a bit different from actors because container events are always stored in separate channels by event type rather than a single serialized channel for all actor message types. 
> The use of separate channels within containers allows for efficient batch processing in cases where event types have no ordering dependencies, but ordering by default is preferable in many other cases involving actors.

"События" (`events`) и "сообщения" (`messages`) часто используются как взаимозаменяемые термины, однако мы будет использовать различные термины, чтобы отличить "события" контейнеров от "сообщений" акторов.
Контейнеры уже имеют свою собственную встроенную систему событий, но семантически она имеет несколько различий. В частости, в контейнере события одного типа всегда хранятся в соответсвуюшем канале, в то время как у акторов все сообщения хранятся в общем сериализованном канале для все типов сообщений.
Использование разделенных каналов в контейнерах позволяет эффективно выполнять пакетную обработку сообщений взаимонезависимых типов. Но в ряде других случаев с участием акторов может быть предпочтительна упорядоченная обработка данных.

### Wrapping containers | Упаковка контейнеров

> It's useful to wrap a container within an actor, where incoming messages to the actor automatically dispatched to the container, and systems within the container have access to an outbox for sending messages to other actors. 
> This approach allows keeping isolated worlds, such as a subset of world state for AI forces or UI state.

Полезно обернуть контейнер в актор, где входящие сообщения автоматически перенаправляются в контейнер, а системы контейнера имеют доступ к исходящему ящику для отправки сообщений другим акторам.
Этот подход позволяет изолировать миры, например, подмножество мирового состояния для сил ИИ или состояние пользовательского интерфейса.

### Replay debugging | Отладка ... ?

> If you can write logic where your game state is fully determined by the sequence of incoming messages, you can log these messages and replay them to diagnose bugs. 
> This works best if you can isolate the problem to a single actor, such as observing incorrect state or incorrect outgoing messages given a correct input sequence.

Если вы можете написать логику игры так, что состояние будет однозначно определено входящими сообщениями, то вы можете записать все сообщения и воспроизвести их позднее для диагностики ошибок.
Еще лучше, если вы можете изолировать проблему для конкретного актора, наблюдая некорректное состояние или неправильные исходящие сообщения при правильной последовательности ввода.

### Message ordering | Порядок сообщений

> Messages sent from one actor to another are guaranteed to arrive in the order they were sent, but they may be interleaved with messages arriving from other actors. 
> In general, multiple actors and parallelism can introduce complexity similar to the use of microservices, which address scaleability but can introduce race conditions and challenges in synchronization.

Сообщения отправленные от одного актора к другому гарантированно поступаю в том порядке, в котором они были отправлены, но могут перемешиываться с сообщениями от других акторов.
В целом, множество акторов и параллельность могут приводить к увеличению сложности аналогично микросервисам, которые решают проблему масштабируемости, но могут приводить к состоянию гонки и проблемам синхронизации.

### Multithreading | Многопоточность

> You can designate actors to run on either the main thread (for UI if needed) or a background thread. 
> Actors run when a batch of messages is delivered, resembling task-based parallelism. 
> In addition to running designated actors, the main thread also delivers messages among actors, although this could change in the future if it becomes a bottleneck. 
> Background actors currently run using a fixed pool of worker threads.

Вы можете назнать акторов для работы в основном потоке (например, для пользовательского интерфейса) или фоновом.
Акторы запускаются при доставке пакета сообщений, что напоминает параллелизм на основе задач.
Кроме запуска назначенных акторов основной поток также занимается доставкой сообщений между акторами, хотя этот пункт может измениться в дальнейшем, если обнаружится ботлнек.
Фоновые акторы в настоящее время выполняются с использование фиксированного пула рабочих потоков.

## Integration | Интеграция

> How does Garnet integrate with frameworks or engines like Unity, MonoGame, or UrhoSharp? 
> You have a few options depending on how much you want to depend on Garnet, your chosen framework, and your own code. 
> This approach also works for integrating narrower libraries like physics or networking.

Как Garnet интегрируется с фреймворками или движками типа Unity, MonoGame или UrhoSharp?
У вас есть несолько вариантов в зависимости от того, насколько вы хотите зависеть от Garnet, выбранного фреймворка и вашего кода.
Этот подход также работает при подключении более узкоспециализированных библиотек для физики или работы с сетью.

### Abstracting framework calls | Абстрагирование от обращений к фреймворку

> When you need to call the framework (e.g. MonoGame) from your code, you can choose to insulate your code from the framework with an abstraction layer. 
> This reduces your dependency on it, but it takes more effort and may result in less power to use framework-specific features and more overhead in marshaling data. 
> If you decide to abstract, you have several options for defining the abstraction layer:

Когда вам требуется вызвать фреймворк (например MonoGame) из вашего кода, вы можете изолировать свой код от фреймворка с помощью уровня абстракции.
Это уменьшит вашу зависимость от фреймворка, но потребует больше усилий и может привести к потерям в производительности из-за снижения роли платформо-специфичных функций и большим накладным расходам при передаче и преобразовании данных.
Если вы таки решили абстрагироваться, то есть несколько вариантов определения абстрактного уровня:

> - **Services**: Register an interface for a subsystem and provide an implemention for the specific framework, e.g. *ISpriteRenderer* with *MonoGameSpriteRenderer*. 
> This makes sense if you want synchronous calls or an explicit interface.
>
> - **Events**: Define interface event types and framework-specific systems which subscribe to them, e.g. a sprite rendering system subscribing to *DrawSprite* events. 
> This way is more decoupled, but the interface may not be so clear.
>
> - **Components**: Define interface component types and implement framework-specific systems which iterate over them, e.g. a sprite rendering system which iterates over entities with a *Sprite* component. 

- **Сервисы**: Зарегистрируйте интерфейс для подсистемы и предоставьте реализацию для конкретного фреймворка, например `ISpriteRenderer` с `MonoGameSpriteRenderer`. 
Данный способ предпочтителен, если необходимы синхронные вызовы или явные интерфейсы.

- **События**: Определите интерфейсные типы событий и специфичные для фреймворка системы, которые будут на них подписаны, например система рендеринга спрайтов подписывается на события `DrawSprite`.
В этом случае достигается меньшая связность, но интерфейс становится менее очевидным.

- **Компоненты**: Определите интерфейсные типы компонентов и реализуйте специфичные для платформы системы, которые будут итерироваться по ним, например система рендеринга будет итерироваться по сущностям имеющим компонент `Sprite`.

### Sending framework events | ...

> For the reverse direction, when you want the framework to call your code, you can simply send interface event types and run the container or actors.

Для обратных вызовов, когда требуется, чтобы фреймворк вызывает ваш код, вы можете просто отправить сообщение интерфейсного типа и запустить контейнер или актор.

```fsharp
    let world = Container()
    // > [configure container here]
    // [настраиваем контейнер здесь]
    override c.Update gt = 
        world.Run { deltaTime = gt.ElapsedGameTime }
    override c.Draw gameTime = 
        world.Run <| Draw()
```

### Code organization | Организация кода

> To minimize your dependency on Garnet and your chosen framework, you can organize your code into layers:

Чтобы минимизировать зависимость от Garnet и выбранного фреймворка, вы можете организовывать свой код следующим образом:

> - Math code
> - Domain types
> - Domain logic
> - Framework interface types
> - [Garnet]
> - System definitions
> - [Framework]
> - Startup code

- Математика
- Доменные типы
- Доменная логика
- Типы для взаимодействия с фреймворком
- [Garnet]
- Определение систем
- [Framework]
- Точка входа

> See the [sample code](https://github.com/bcarruthers/garnet/tree/master/samples) for more detail.

Посмотрите [код примеров](https://github.com/bcarruthers/garnet/tree/master/samples) для большей информации.

## Roadmap | Дорожная карта

> - Performance improvements
> - Container snapshots and entity replication 
> - More samples
> - More test coverage
> - Benchmarks
> - Comprehensive docs
> - Urho scripts with hot reloading
> - Extensions or samples for networking and compression
> - Fault tolerance in actor system
> - Guidance for managing game states

- Улучшения производительности
- Слепки (snapshot-ы) контейнеров и тиражирование сущностей
- Больше примеров
- Большее покрытие тестами
- Бенчмарки
- Полноценная документация
- Urho скрипты с горячей заменой
- Расширения или примеры для работы с сетью или _?сжатия?_
- Отказоустойчивость в акторной системе
- Руководства по управлению состоянием игры

## FAQ

> - **Why F#?** F# offers conciseness, functional-first defaults like immutability, an algebraic type system, interactive code editing, and pragmatic support for other paradigms like OOP. 
> Strong type safety makes it more likely that code is correct, which is especially helpful for tweaking game code that changes frequently enough to make unit testing impractical.
> 
> - **What about performance?** Functional code often involves allocation, which sometimes conflicts with the goal of consistent performance when garbage collection occurs. 
> A goal of this library is to reduce the effort in writing code that minimizes allocation. 
> But for simple games, this is likely a non-issue and you should start with idiomatic code.
> 
> - **Why use ECS over MVU?** You probably shouldn't start with ECS for a simple game, at least not when prototyping, unless you already have a good understanding of where it might be beneficial. 
> MVU avoids a lot of complexity and has stronger type safety and immutability guarantees than ECS, but you may encounter issues if your project has demanding performance requirements or needs more flexibility than it allows. 

- **Почему F#?** F# предлагает краткость изложения, functional-first подход с фишками типа иммутабельности, алгебраической системы типов, интерактивного редактирования кода и прагматичной поддержки иных парадигм типа ООП.
Сильная безопасность типов дает больше гарантий, что код корректен, что особенно полезно для доводке кода игры, который довольно часто изменяется и unit-тестирование на практике невозможно.

- **Что насчет производительности?** Функциональный код часто провоцирует аллокации, что в некоторых случаях может послужить проблемой для производительности и GC в частности.
Цель данной библиотеки снизить усилия требуемые для написания кода минимизируюшего аллокации.
Но для простых игр, это скорее всего не проблема, и вы можете начать с идеоматически верного кода.

- **Зачем использовать ECS, а не MVU?** Вы, вероятно, не должны начинать с ECS для простых игр, как минимум не на стадии прототипа. Кроме тех случаев, когда вы отлично понимаете, зачем это нужно и какие выгоды оно принесет.
MVU избегает большой  сложности и дает очень строгую типобезопасность и иммутабельность, в отличии от ECS, но вы можете столкнуться с проблемами, если ваш проект имеет высокие требования к производительности или нуждается в большей гипкости, чем это позволяет MVU.

## License | Лицензия

> This project is licensed under the [MIT license](https://github.com/bcarruthers/garnet/blob/master/LICENSE).

Проект находится под лицензией [MIT license](https://github.com/bcarruthers/garnet/blob/master/LICENSE).

## Maintainer(s)

- [@bcarruthers](https://github.com/bcarruthers)