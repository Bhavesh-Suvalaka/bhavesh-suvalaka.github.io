---
theme: seriph
background: sahaj.png
layout: cover
canvasWidth: 890
class: text-center
highlighter: shiki
lineNumbers: false
info:
drawings:
  persist: false
transition: fade-out
title: Loom
mdc: true
---

# Java Loom - Virtual thread

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

---

# Agenda

- 💻 **Need of Concurrency**
- 🔗 **Thread-Per-Request-Style**
- 🧵 **Limitations of native threads**
- 😵‍💫 **Async - Non blocking**
- 🪡 **So, Loom**
- 𐄷  **Virtual vs Platform threads**
- ⚙  **How ?**
- 🚦 **Adoption guide**
- ❓ **Questions and feedback**

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

# Need of concurrency

<br>
<p v-click> To utilise the CPU properly </p>
<p v-click> To full-fill the need of modern application </p>
<p v-click> 
        <li> High throughput </li>
        <li> Low latency </li>
</p>

---

# thread-per-request style

<br>

<v-after>

```ts {all|2|3|4|6-8|all} 
    public List<Product> recommendProducts(String customerId) {
        Optional<Customer> customer = customerService.fetchCustomer(customerId);
        List<CustomerPreference> custPref = preferenceService.fetchCustomerPrefs(customerId);
        List<Order> orders = orderService.fetchOrderHistory(customerId);

        return customer
                .map(it -> prepareProductRecommendation(it, custPref, orders))
                .orElse(Collections.emptyList());

    }
```

</v-after>

<!--
Throughput = no of con req / latency 
10/ 50 ms = 200 req /s 
1000  20000 req/ s
-->

---

# Limitations of platform threads

<div class="container" style="display: flex;">
    <div v-click style="flex-grow: 3;">
        <img src="/images/native_threads.png" class="m-0 h-100 rounded shadow" />
    </div>
 <div  style="flex-grow: 2; margin-left: 50px;"> <br> <br>
    <p v-click> Each native thread is mapped to corresponding kernel thread </p>
    <p v-click> Thread creation is heavy operation </p>
    <p v-click> High memory footprint </p>
    <p v-click> Context switching is heavy </p>
    <p v-click>JDK caps the application's throughput  </p>
 </div>
</div>

---

# Asynchronous style

<div class="container" style="display: flex;">
    <div v-click style="flex-grow: 1;">
        <img src="/images/seqencial_execution.png" class="m-5 h-70 w-70 rounded shadow" />
    </div>
    <div v-click style="flex-grow: 2;">
        <img  src="/images/concurrent_execution.png" class="m-5 h-70 rounded shadow" />
    </div>
</div>

---

# Asynchronous- Non Blocking - style

<div class="container" style="display: flex;">
    <div style="flex-grow: 2;">
        <img  src="/images/concurrent_execution.png" class="m-5 h-70 rounded shadow" />
    </div>
    <div  style="flex-grow: 2; margin-left: 50px;"> <br> <br>
        <p v-click> Stack traces provide no usable context. </p>
        <p v-click> Debugging and profiling becomes hard </p>
        <p v-click> Compose less elegantly.</p>
        <p v-click> Application's unit of concurrency is no longer the platform's unit of concurrency.</p>
     </div>
</div>

---

# So, Virtual Threads:

<div class="container" style="display: flex;">
    <div v-click style="flex-grow: 3;">
        <img src="/images/virtual-threads.png" class="m-0 h-100 rounded shadow" />
    </div>
 <div  style="flex-grow: 2; margin-left: 50px;"> <br> <br>
    <p v-click> Uses thread-per-request style more efficiently </p>
    <p v-click> JVM handles the lifecycle and scheduling </p>
    <p v-click> Can achieve same scale as asynchronous</p>
    <p v-click> Cheap to create and almost infinitely plentiful</p>
    <p v-click> Does not require learning new concepts </p>
 </div>
</div>

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

---
max-height: 100%
---

# How ?

<br/>
<v-click>Implementation of java.lang.Thread </v-click>
<v-click>
<p> It has 2 main construct: 
        <li> Continuation </li>
        <li> Scheduler </li>
</p> 
</v-click>
<br/>

--- 

<div>
    <img src="/images/VT-in-1.png" class="m-0 h-95 rounded shadow" v-click="[0,1]"/>
    <img src="/images/VT-in-2.png" class="m-0 h-95 rounded shadow" v-click="[1,2]"/>
    <img src="/images/VT-in-3.png" class="m-0 h-95 rounded shadow" v-click="[2,3]"/>
    <img src="/images/VT-in-4.png" class="m-0 h-95 rounded shadow" v-click="[3,4]"/>
    <img src="/images/VT-in-1.png" class="m-0 h-95 rounded shadow" v-click="[4,5]"/>
    <img src="/images/VT-in-5.png" class="m-0 h-95 rounded shadow" v-click="5"/>
</div>
---

# Adoption Guide

<br/>
<p v-click>
    1. Write blocking code using Thread-Per-Request style
</p>

<v-click>

```ts {all}
CompletableFuture.supplyAsync(info::getUrl, pool)
   .thenCompose(url -> getBodyAsync(url, HttpResponse.BodyHandlers.ofString()))
   .thenApply(info::findImage)
   .thenCompose(url -> getBodyAsync(url, HttpResponse.BodyHandlers.ofByteArray()))
   .thenApply(info::setImageData)
   .thenAccept(this::process)
   .exceptionally(t -> { t.printStackTrace(); return null; });
```

</v-click>

<v-click>

```ts {all}

try {
   String page = getBody(info.getUrl(), HttpResponse.BodyHandlers.ofString());
   String imageUrl = info.findImage(page);
   byte[] data = getBody(imageUrl, HttpResponse.BodyHandlers.ofByteArray());   
   info.setImageData(data);
   process(info);
} catch (Exception ex) {
   t.printStackTrace();
}

```

</v-click>

---

<p> 2. Never pool a virtual thread </p>

<v-click>

```ts
Future<ResultA> f1 = sharedThreadPoolExecutor.submit(task1);
Future<ResultB> f2 = sharedThreadPoolExecutor.submit(task2);
// ... use futures
```

</v-click>

<v-click>

```ts
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
   Future<ResultA> f1 = executor.submit(task1);
   Future<ResultB> f2 = executor.submit(task2);
   // ... use futures
}
```

</v-click>

---

<p> 3. Use Semaphores to Limit Concurrency</p>

<v-click>

```ts {all|1|4|6|8|all}
Semaphore sem = new Semaphore(10);
...
Result foo() {
    sem.acquire();
    try {
        return callLimitedService();
    } finally {
        sem.release();
    }
}
```

</v-click>

---

<p> 4. Don't Cache Expensive Reusable Objects in Thread-Local Variables </p>
<br>

```ts {all}
static final ThreadLocal<SimpleDateFormat> cachedFormatter = 
       ThreadLocal.withInitial(SimpleDateFormat::new);

void foo() {
  ...
	cachedFormatter.get().format(...);
	...
} 
```

---

<p> 5. Avoid lengthy and frequent pinning. </p> 

<v-click>

```ts {all|1-3|7-12|}
synchronized(lockObj) {
    frequentIO();
}

---

lock.lock();
try {
    frequentIO();
} finally {
    lock.unlock();
}
```

</v-click>

---

<h3>6. Meant for I/O intensive application </h3>

<li v-click> Don't use for CPU intensive application </li>
<li v-click> Don't use in parallel stream </li>

---

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

# Questions?

---

# Learn More

[Loom Proposal](https://cr.openjdk.org/~rpressler/loom/Loom-Proposal.html) <br/>
[Second Preview](https://openjdk.org/jeps/425) <br/>
[Documentations](https://docs.oracle.com/en/java/javase/21/core/virtual-threads.html#GUID-DC4306FC-D6C1-4BCC-AECE-48C32C1A8DAA)<br/>
[Loom notes](https://jdk.java.net/loom/) <br/>
[SprinBoot Adoption](https://www.infoq.com/news/2023/12/spring-boot-virtual-threads/#:~:text=Spring%20Boot%203.2%20has%20integrated,now%20operate%20on%20virtual%20threads) <br/>
[All Loom Related blogs](https://inside.java/tag/loom) <br/>
[One Million Concurrent Connection](https://josephmate.github.io/2022-04-14-max-connections/)<br/>
[GitHub](https://github.com/Bhavesh-Suvalaka/loom-demo) <br/>
