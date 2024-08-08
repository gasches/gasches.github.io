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

https://kotlinlang.org/spec/pdf/sections/asynchronous-programming-with-coroutines.pdf
https://blog.kotlin-academy.com/kotlin-coroutines-animated-part-1-coroutine-hello-world-51797d8b9cd4
https://blog.kotlin-academy.com/building-kotlin-coroutine-framework-from-scratch-part-2-reinventing-dispatchers-bff72b82e42d
