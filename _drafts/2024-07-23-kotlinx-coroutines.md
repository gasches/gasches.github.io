---
layout: post
title:  "Kotlinx coroutines"
date:   2024-07-23 00:00:00 +0000
categories: kotlin coroutines
---

* TOC:
{:toc}

## Введение

В этой статье будет рассмотрена реализация поддержки корутин в библиотеке [`kotlinx.coroutines`](https://github.com/Kotlin/kotlinx.coroutines){:target="_blank"}.

В стандартной библиотеке поддержка корутин представлена в пакетах:

* [kotlin.coroutines](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.coroutines/){:target="_blank"}
  > Basic primitives for creating and suspending coroutines: Continuation, CoroutineContext interfaces, coroutine creation and suspension top-level functions.
* [kotlin.coroutines.cancellation](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.coroutines.cancellation/){:target="_blank"}
* [kotlin.coroutines.intrinsics](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.coroutines.intrinsics/){:target="_blank"}
  > Low-level building blocks for libraries that provide coroutine-based APIs.
