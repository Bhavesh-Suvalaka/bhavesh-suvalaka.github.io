---
theme: seriph
background: ./images/background.jpg
layout: cover
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
- 🚕 **Journey of Concurrency**
- 🧵 **Limitations of native threads**
- 🪡 **So, Loom**
- 𐄷  **Virtual vs Platform threads**
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

# Journey of Concurrency
<br>
<p v-click> Parallel vs concurrent </p> 
<p v-click> Processes and threads </p>
<p v-click> Multi-threading </p>
<p v-click> Asynchronous style </p> 

---

# thread-per-request style
<br>

```ts {all|2|3|4|6-8|all}
    public List<Product> recommendProducts(String customerId) {
        List<Order> orders = orderService.fetchOrderHistory(customerId);
        List<CustomerPreference> customerPreference = preferenceService.fetchCustomerPreferences(customerId);
        Optional<Customer> customer = customerService.fetchCustomer(customerId);

        return customer
                .map(it -> fetchProductRecommendation(it, customerPreference, orders))
                .orElse(Collections.emptyList());

    } 
```
<!--

Easy to understand and program.
Easy to debug and profile.
the platform's unit of concurrency is the application's unit of concurrency.

 The scalability of server application is governed by Little's law 
    <li> L(throughput) = 𝜆(concurrent request) * W(latenncy) </li>
    <li v-click> 200(request per second) = 10 * 50ms </li>
    <li v-click> 2000(request per second) = ? </li>
    <li v-click> 2000(request per second) = 100(concurrent request) * 50ms </li>
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

# Asynchronous style

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
        <img src="/images/fibers.png" class="m-0 h-100 rounded shadow" />
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

# Adoption Guide
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

<style>
    
</style>


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

# Feedback and questions?

---

# Learn More
[Loom Proposal](https://cr.openjdk.org/~rpressler/loom/Loom-Proposal.html)
[Second Preview](https://openjdk.org/jeps/425)
[Documentations](https://docs.oracle.com/en/java/javase/21/core/virtual-threads.html#GUID-DC4306FC-D6C1-4BCC-AECE-48C32C1A8DAA) · 

[GitHub](https://github.com/Bhavesh-Suvalaka/fibers) 
[Showcases](https://bhavesh-suvalaka.github.io/loom-slides/)
