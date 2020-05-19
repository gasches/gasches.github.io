---
layout: post
title:  Akka Streams Basics
date:   2020-05-17 00:00:00 +0000
categories: reactive programming
---
> конспект курса [**«Lightbend Akka Streams for Java - Professional»** by Lightbend Academy][course-link]{:target="_blank"}

* TOC:
{:toc}

## Введение

Stream - это последовательность данных, состоящая из отдельных элементов. Обычно размер stream'а заранее неизвестен, а размер данных, проходящих через stream, слишком велик для размещения в памяти. Примеры: онлайн видео поток, данные фитнес-браслета.

[Reactive Streams][reactive-streams]{:target="_blank"} - это инициатива по созданию стандарта асинхронной потоковой обработки данных с обеспечением неблокирующего контроля интенсивности их поступления (back pressure). Готовность к получению новой порции данных, при этом, исходит от потребителя данных (_subscriber_) к поставщику (_publisher_). Основная цель данной инициативы - обеспечение взаимодействия между различными (но соответствующими стандарту) реализациями.

Стандартными компонентами Reactive Streams в Java являются:

* [`Publisher`][flow-publisher]{:target="_blank"}
* [`Processor`][flow-processor]{:target="_blank"}
* [`Subscriber`][flow-subscriber]{:target="_blank"}
* [`Subscription`][flow-subscription]{:target="_blank"}

Модуль [Akka Streams][akka-streams]{:target="_blank"} предоставляет высокоуровневое API для работы со stream'ами, а интерфейсы Reactive Streams используются только в низкоуровневом API.

Схема линейного потока данных:

```
{Внешний источник данных} --> [Source] --> [Flow] --> [Sink] --> {Внешний потребитель данных}
```

Пример простого stream'а:

```java
akka.actor.ActorSystem actorSystem = akka.actor.ActorSystem.create("system");
akka.stream.Materializer materializer = akka.stream.Materializer.createMaterializer(actorSystem);

akka.stream.javadsl.Source.from(List.of(1, 2, 3, 4))
    .via(akka.stream.javadsl.Flow.of(Integer.class).map(elem -> elem * 2))
    .to(akka.stream.javadsl.Sink.foreach(System.out::println))
    .run(materializer);
```

## Sources

[`Source`][source]{:target="_blank"} - это _вход_ в stream.

`Source` _публикует_ данные **только** по запросу от последующих стадий.

Некоторые типы `Source`'ов:

* [`Source::empty()`][source-empty]{:target="_blank"}
* [`Source::single(elem: T)`][source-single]{:target="_blank"}
* [`Source::repeat(elem: T)`][source-repeat]{:target="_blank"}
* [`Source::tick(delay: Duration, interval: Duration, elem: T)`][source-tick]{:target="_blank"}
* [`Source::from(Iterable)`][source-from-iterable]{:target="_blank"}
* [`Source::fromIterator(akka.japi.function.Creator<Iterator<T>>)`][source-from-iterator]{:target="_blank"}
* [`Source::cycle(akka.japi.function.Creator<Iterator<T>>)`][source-cycle]{:target="_blank"}
* [`Source::unfold(initialValue: T, Function<T, Optional<Pair<T, R>>>)`][source-unfold]{:target="_blank"}
* [`Source::unfoldAsync(initialValue: T, Function<T, CompletionStage<Optional<Pair<T, R>>>>)`][source-unfold-async]{:target="_blank"}
* [`Source::actorRef(completionMatcher: akka.japi.function.Function<Object,​ Optional<akka.stream.CompletionStrategy>>, failureMatcher: akka.japi.function.Function<Object, Optional<Throwable>>, bufferSize: int, OverflowStrategy)`][source-actor-ref]{:target="_blank"}
* [`FileIO::fromPath(java.nio.file.Path, chunkSize: int)`][source-file-for-path]{:target="_blank"}
* [`Tcp.get(akka.actor.ActorSystem)::bind(interface: String, port: int)`][source-tcp-bind]{:target="_blank"}
* [`StreamConverters::fromInputStream(akka.japi.function.Creator<InputStream>)`][source-from-input-stream]{:target="_blank"}

## Sinks

[`Sink`][sink]{:target="_blank"} - это _выход_ из stream.

`Sink` управляет поступлением новых данных.

Некоторые типы `Sink`'ов:

* [`Sink::ignore()`][sink-ignore]{:target="_blank"}
* [`Sink::forEach(akka.japi.function.Procedure<T>)`][sink-for-each]{:target="_blank"}
* [`Sink::head()`][sink-head]{:target="_blank"}
* [`Sink::last()`][sink-last]{:target="_blank"}
* [`Sink::headOption()`][sink-head-option]{:target="_blank"}
* [`Sink::lastOption()`][sink-last-option]{:target="_blank"}
* [`Sink::seq()`][sink-seq]{:target="_blank"}
* [`Sink::fold(T zero, akka.japi.function.Function2<T, ​In, T> f)`][sink-fold]{:target="_blank"}
* [`Sink::reduce(akka.japi.function.Function2<In,​ In,​ In> f)`][sink-reduce]{:target="_blank"}
* [`Sink::actorRef(ref: ActorRef, onCompleteMessage)`][sink-actor-ref]{:target="_blank"}
* [`Sink::actorRefWithBackpressure(ref: ActorRef, onInitMessage, ackMessage, onCompleteMessage, onFailureMessage: Function<Throwable, Object>)`][sink-actor-ref-with-backpressure]{:target="_blank"}
* [`FileIO::toPath(java.nio.file.Path)`][sink-file-to-path]{:target="_blank"}
* [`StreamConverters::fromOutputStream(out: scala.Function0<java.io.OutputStream>, autoFlush: boolean)`][sink-from-output-stream]{:target="_blank"}

## Flows

[`Flow`][flow]{:target="_blank"} - используется для манипулирования данными идущими от `Source` к `Sink`.

Некоторые типы `Flow`:

* [`Flow::map(akka.japi.function.Function<R, T>)`][flow-map]{:target="_blank"}
* [`Flow::mapAsync(parallelism: int, Function<R,​ java.util.concurrent.CompletionStage<T>>)`][flow-map-async]{:target="_blank"}
* [`Flow::mapConcat(akka.japi.function.Function<Out,​ Iterable<T>>)`][flow-map-concat]{:target="_blank"}
* [`Flow::grouped(chunkSize: int)`][flow-grouped]{:target="_blank"}
* [`Flow::sliding(windowSize: int, step: int)`][flow-sliding]{:target="_blank"}
* [`Flow::fold(zero: T, akka.japi.function.Function2<T,​ R, T>)`][flow-fold]{:target="_blank"}
* [`Flow::scan(zero: T, akka.japi.function.Function2<T,​ R, T>)`][flow-scan]{:target="_blank"}
* [`Flow::filter(akka.japi.function.Predicate<T>)`][flow-filter]{:target="_blank"}
* [`Flow::collect(scala.PartialFunction<R,​ T>)`][flow-collect]{:target="_blank"}
* [`Flow::takeWithin(Duration)`][flow-take-within]{:target="_blank"}
* [`Flow::dropWithin(Duration)`][flow-drop-within]{:target="_blank"}
* [`Flow::groupedWithin(chinkSize: int, Duration)`][flow-grouped-within]{:target="_blank"}
* [`Flow::log(name: String)`][flow-log]{:target="_blank"}
* [`Flow::zip(source: akka.stream.Graph<akka.stream.SourceShape<T>,​ ?>)`][flow-zip]{:target="_blank"}
* [`Flow::flatMapConcat(akka.japi.function.Function<R,​ ? extends Graph<SourceShape<T>, ​M>>)`][flow-flat-map-concat]{:target="_blank"}
* [`Flow::flatMapMerge(breadth: int, akka.japi.function.Function<R, ? extends Graph<SourceShape<T>, ​M>>)`][flow-flat-map-merge]{:target="_blank"}
* [`Flow::buffer(size: int, akka.stream.OverflowStrategy)`][flow-buffer]{:target="_blank"}
* [`Flow::batch(max: long, seed: akka.japi.function.Function<Out, ​S>, aggregate: akka.japi.function.Function2<S,​ Out, ​S>)`][flow-batch]{:target="_blank"}

Некоторые, доступные во `Flow`, трансформации могут быть выполнены методами `Source`.

## Runnable Graphs

[RunnableGraph][runnable-graph]{:target="_blank"} - представляет собой соединенные друг с другом `Source`, `Flow` (необязательно) и `Sink`.

[Материализация][stream-materialization]{:target="_blank"} (materialization) заключается в выделение необходимых ресурсов для запуска stream'а, и выполняется с помощью так называемых _терминальных операций_, таких как `run()` и `runWith()`.

Результатом материализации stream'а является [материализованное значение][materialized-values]{:target="_blank"} которое предоставляет возможность взаимодействия с ним.

## Отказоустойчивость

Отказоустойчивость в Akka Streams реализована в виде [стратегий][supervision-strategies]{:target="_blank"}, которые настраиваются с помощью [атрибутов][attributes-supervision-strategy]{:target="_blank"} `RunnableGraph`'а (или отдельных стадий).

Для завершения, в случае ошибки, stream'а с определенным значением используется метод [`recover`][recover]{:target="_blank"}.

## Графы

Объединения и разветвления stream'ов реализованы в виде [соединителей][constructing-graphs]{:target="_blank"} (junctions).

Для создания сложных графов Akka Streams предоставляет [Graph DSL][composing-complex-systems]{:target="_blank"}.

## Fusing

Все операторы Akka Streams, по умолчанию, _сливаются_ (_fusing_) вместе и выполняются последовательно. Т.е. каждый элемент должен пройти все этапы перед тем как в stream поступит новый элемент.

Данное поведение настраивается параметром `akka.stream.materializer.auto-fusing`.

Для [асинхронного][stream-parallelism]{:target="_blank"} выполнения любой стадии используется метод `async()`.

Следует учитывать, что асинхронное выполнение стадии в stream приведет к накладным расходам в виде дополнительных акторов (и их mailbox'ов) и буферов.

[course-link]: https://academy.lightbend.com/courses/course-v1:lightbend+LTJ-P+v1/course/
[reactive-streams]: http://www.reactive-streams.org/
[flow-processor]: https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/Flow.Processor.html
[flow-publisher]: https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/Flow.Publisher.html
[flow-subscriber]: https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/Flow.Subscriber.html
[flow-subscription]: https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/Flow.Subscription.html
[akka-streams]: https://doc.akka.io/docs/akka/current/stream/index.html
[source]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Source.html
[sink]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Sink.html
[flow]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Flow.html
[source-empty]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Source.html#empty()
[source-single]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Source.html#single(T)
[source-repeat]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Source.html#repeat(T)
[source-tick]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Source.html#tick(java.time.Duration,java.time.Duration,O)
[source-from-iterable]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Source.html#from(java.lang.Iterable)
[source-from-iterator]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Source.html#fromIterator(akka.japi.function.Creator)
[source-cycle]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Source.html#cycle(akka.japi.function.Creator)
[source-unfold]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Source.html#unfold(S,akka.japi.function.Function)
[source-unfold-async]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Source.html#unfoldAsync(S,akka.japi.function.Function)
[source-actor-ref]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Source.html#actorRef(akka.japi.function.Function,akka.japi.function.Function,int,akka.stream.OverflowStrategy)
[source-file-for-path]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/FileIO.html#fromPath(java.nio.file.Path,int)
[source-tcp-bind]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Tcp.html#bind(java.lang.String,int)
[source-from-input-stream]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/StreamConverters.html#fromInputStream(akka.japi.function.Creator)
[sink-ignore]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Sink.html#ignore()
[sink-for-each]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Sink.html#foreach(akka.japi.function.Procedure)
[sink-head]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Sink.html#head()
[sink-last]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Sink.html#last()
[sink-head-option]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Sink.html#headOption()
[sink-last-option]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Sink.html#lastOption()
[sink-seq]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Sink.html#seq()
[sink-fold]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Sink.html#fold(U,akka.japi.function.Function2)
[sink-reduce]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Sink.html#reduce(akka.japi.function.Function2)
[sink-actor-ref]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Sink.html#actorRef(akka.actor.ActorRef,java.lang.Object)
[sink-actor-ref-with-backpressure]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Sink.html#actorRefWithBackpressure(akka.actor.ActorRef,java.lang.Object,java.lang.Object,java.lang.Object,akka.japi.function.Function)
[sink-file-to-path]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/FileIO.html#toPath(java.nio.file.Path)
[sink-from-output-stream]: https://doc.akka.io/japi/akka/current/akka/stream/scaladsl/StreamConverters.html#fromOutputStream(scala.Function0,boolean)
[flow-map]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Flow.html#map(akka.japi.function.Function)
[flow-map-async]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Flow.html#mapAsync(int,akka.japi.function.Function)
[flow-map-concat]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Flow.html#mapConcat(akka.japi.function.Function)
[flow-grouped]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Flow.html#grouped(int)
[flow-sliding]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Flow.html#sliding(int,int)
[flow-fold]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Flow.html#fold(T,akka.japi.function.Function2)
[flow-scan]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Flow.html#scan(T,akka.japi.function.Function2)
[flow-filter]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Flow.html#filter(akka.japi.function.Predicate)
[flow-collect]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Flow.html#collect(scala.PartialFunction)
[flow-take-within]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Flow.html#takeWithin(java.time.Duration)
[flow-drop-within]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Flow.html#dropWithin(java.time.Duration)
[flow-grouped-within]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Flow.html#groupedWithin(int,java.time.Duration)
[flow-log]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Flow.html#log(java.lang.String)
[flow-zip]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Flow.html#zip(akka.stream.Graph)
[flow-flat-map-concat]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Flow.html#flatMapConcat(akka.japi.function.Function)
[flow-flat-map-merge]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Flow.html#flatMapMerge(int,akka.japi.function.Function)
[flow-buffer]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Flow.html#buffer(int,akka.stream.OverflowStrategy)
[flow-batch]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/Flow.html#batch(long,akka.japi.function.Function,akka.japi.function.Function2)
[runnable-graph]: https://doc.akka.io/japi/akka/current/akka/stream/javadsl/RunnableGraph.html
[stream-materialization]: https://doc.akka.io/docs/akka/current/stream/stream-flows-and-basics.html#stream-materialization
[materialized-values]: https://doc.akka.io/docs/akka/current/stream/stream-composition.html#materialized-values
[supervision-strategies]: https://doc.akka.io/docs/akka/current/stream/stream-error.html#supervision-strategies
[attributes-supervision-strategy]: https://doc.akka.io/japi/akka/current/akka/stream/ActorAttributes.html#withSupervisionStrategy(akka.japi.function.Function)
[recover]: https://doc.akka.io/docs/akka/current/stream/stream-error.html#recover
[constructing-graphs]: https://doc.akka.io/docs/akka/current/stream/stream-graphs.html#constructing-graphs
[composing-complex-systems]: https://doc.akka.io/docs/akka/current/stream/stream-composition.html#composing-complex-systems
[stream-parallelism]: https://doc.akka.io/docs/akka/current/stream/stream-parallelism.html
