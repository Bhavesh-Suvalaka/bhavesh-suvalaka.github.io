---
theme: seriph
background: ./images/background.jpg
class: text-center
highlighter: shiki
lineNumbers: false
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
drawings:
  persist: false
transition: slide-left
title: Loom
mdc: true
---

# Java Loom - Fiber

### Synchronously-Asynchronous

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Wait what? <carbon:arrow-right class="inline"/>
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/Bhavesh-Suvalaka/fibers" target="_blank" alt="GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
transition: fade-out
---

# Agenda

- 💻 **Need of Concurrency** - Low latency, high throughput, huge scale
- 🚕 **Journey of Concurrency** - processes, threads, async programming
- 🧵 **Limitations of native threads** - Bound by kernel threads
- 🪡 **So, Loom** - Brief introduction
- 𐄷 **Compare Loom with Native threads** - Performance difference and how and why ?
- 👀 **Caveats** - Things to lookout.
- ❓ **Questions** - feedback time!

<br>
<br>

<style>

h1 {

  background-color: #2B90B6;

  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);

  background-size: 100%;

  -webkit-background-clip: text;

  -moz-background-clip: text;

  -webkit-text-fill-color: transparent;

  -moz-text-fill-color: transparent;

}
</style>
---
layout: default
---
# Journey of Concurrency
    
## Need of concurrency
    * To utilise the CPU properly
    * To full-fill the need of modern application
        <v-click>
          * High throughput
          * Low latency
        </v-click>

## Evolution   
    * Parallel vs concurrent processes 
    * Processes and threads
    * Multi-threading
    * Asyncrounous 

---
transition: fade-out
---
# thread-per-request style
    Pros:
        * Easy to understand and program
        * Easy to debug and profile
        * it uses the platform's unit of concurrency to represent the application's unit of concurrency
    Cons:
        * The scalability of server application is governed by Little's law
            * L(throughput) = 𝜆(concurrent request) * W(latenncy)
            * 200(request per second) = 10 * 50ms
            * 2000(request per second) = ?
            * 2000(request per second) = 100(concurrent request) * 50ms
        * No of threads needs to increase to keepup with this throughput

---
transition: fade-out
---

# Limitations of native threads
<div class="container" style="display: flex;">
    <div v-click style="flex-grow: 3;">
        <img src="/images/native_threads.png" class="m-0 h-100 rounded shadow" />
    </div>
 <div  style="flex-grow: 2; margin-left: 50px;"> <br> <br>
    <p v-click> Each native thread is mapped to corresponding kernel thread </p>
    <p v-click> Creating a new thread is a heavy operation </p>
    <p v-click> The default stack size is 512Kb on 32 bit and 1Mb for 64bit OS </p>
    <p v-click> Context switching is heavy - this increases as no of threads increases </p>
    <p v-click> So, The JDK' caps the application's throughput to a level well below 
      what the hardware can support </p>
 </div>
</div>

---
transition: fade-out
---

# Asynchronous style
    * Returns thread to a pool when it waits for I/O operation to complete
    * Can achieve high number of concurrent operation w/o consuming high no of threads
    * But it come with higher price:
        * Learning curve for async style 
        * Increases verbosity - for e.g reactive programing
        * Stack trace provides no usable context
        * Debugging and profiling becomes hard
        * the application's unit of concurrency  is no longer the platform's unit of concurrency.

---
transition: fade-out
---

# So, Virtual Threads:

<div class="container" style="display: flex;">
    <div v-click style="flex-grow: 3;">
        <img src="/images/fibers.png" class="m-0 h-100 rounded shadow" />
    </div>
 <div  style="flex-grow: 2; margin-left: 50px;"> <br> <br>
    <p v-click> Uses, thread-per-request style more efficiently </p>
    <p v-click> JVM handles the lifecycle and scheduling </p>
    <p v-click> Can achieve same scale as asynchronous style but more transparently</p>
    <p v-click> Cheap to creat and almost infinitely plentiful</p>
    <p v-click> Does not require learning new concepts </p>
 </div>
</div>

---
transition: fade-out
---

# How to create?

```ts {all|1|4|1-8|none}
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.range(0, 10_000).forEach(i -> {
        executor.submit(() -> {
            Thread.sleep(Duration.ofSeconds(1));
            return i;
        });
    });
} 
```
<br>
<p style="color:gold;" v-click> let's see things in action</p>

[^1]: [Learn More](https://sli.dev/guide/syntax.html#line-highlighting)

<style>
.footnotes-sep {
  @apply mt-20 opacity-10;
}
.footnotes {
  @apply text-sm opacity-75;
}
.footnote-backref {
  display: none;
}
</style>

---
# Learn More

[Documentations](https://sli.dev) · [GitHub](https://github.com/slidevjs/slidev) · [Showcases](https://sli.dev/showcases.html)
