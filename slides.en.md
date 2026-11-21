---
marp: true
theme: aifa
paginate: true
lang: en
title: Design for Failure
description: Wait, my system fails?
author: Luiz Schons
footer: 'Design for Failure · luizschons.com'
---

<!-- _class: cover -->
<!-- _paginate: false -->
<!-- _footer: '' -->

# Design for Failure

wait, my system fails?

<br>

**Luiz Schons**

<!--
This talk is based on the article "Design for Failure: wait, my system fails?" at luizschons.com/posts/design-for-failure/. I gave versions of it at SoftWeek, PHPeste and TLC Share Day, from Tech Leads Club, in 2025.
The idea is simple: give names to the patterns we already use, sometimes without knowing it. Retry, fallback, circuit breaker and saga are not recent inventions. Understanding where each one comes from helps you know when to use it, and above all when not to.
All the code I mention is for teaching: it exists to show the idea, not to go straight to production. For that there are well-tested libraries, which I mention along the way.
Transition: start with the hook. The personal introduction comes right after.
-->

---

<!-- _class: statement -->

> Every system **fails**.

<p class="muted">The question is what happens to everything else when it does.</p>

<!--
Let the line sit for a couple of seconds.
The question is not whether a service will go down. It is what happens to everything else when it does. That is today's conversation.
Transition: "Before we go on, let me introduce myself."
-->

---

<div class="about">
<div>

# Luiz Schons

<p class="role">Senior Software Engineer @ <b>PicPay</b></p>

<div class="chips">
<span>Guarapuava, Paraná, Brazil</span>
<span>DevParaná</span>
<span>UTFPR</span>
<span>PHP</span>
</div>

<p class="site">luizschons.com</p>

</div>
<img class="photo" src="assets/luiz.jpg" alt="Luiz Schons">
</div>

<!--
Quick introduction (30 seconds): who I am, where I work, where I come from.
Transition: "I'll split the talk into four parts, following an online store that grows and starts to fail."
-->

---

## The path

<div class="cols4">
<div class="card"><p class="muted small">Part 1</p><h3>Points of failure</h3></div>
<div class="card"><p class="muted small">Part 2</p><h3>Retry, fallback, breaker</h3></div>
<div class="card"><p class="muted small">Part 3</p><h3>Saga</h3></div>
<div class="card"><p class="muted small">Part 4</p><h3>Putting it together</h3></div>
</div>

<!--
Four parts, all along the same thread of an online store.
- Part 1: the store grows, the infrastructure spreads out, and every arrow becomes a point of failure.
- Part 2: what to do when a call fails — retry, fallback and circuit breaker.
- Part 3: when a purchase is a set of steps, the saga and its compensations come in.
- Part 4: the patterns together, and why none of them is a silver bullet.
A single example runs through the talk: e-commerce, stock, shipping and payment.
-->

---

<!-- _class: divider -->

<div class="kicker">Part 1 · The store</div>

# Many points of failure

<!--
First act: from one server to a distributed system, and why that multiplies the ways things can fail.
-->

---

## One store, one server

<div class="cols">
<div class="card"><p class="muted small">At the start</p><h3>20%</h3><p class="small">of capacity. All is well.</p></div>
<div class="card warn"><p class="muted small">The store grows</p><h3>80%</h3><p class="small">It is time to scale.</p></div>
</div>

<p class="muted">One server, one application, one database.</p>

<!--
Every online store starts the same way: one server, one application and one database. While the server runs at 20% of its capacity, all is well. Then the store grows, usage goes to 80%, and it is time to scale.
In the live version I tell this story with more color: the old computer in the room, the "super server" with 4 GB, the bill that arrives when we only buy a bigger machine. Here I go straight to what matters: there are two ways to scale.
-->

---

## Two ways to scale

<div class="cols">
<div class="card"><p class="muted small">Vertical</p><h3>Upgrade what you already have</h3><p class="small">More CPU, more memory. It is simple, but it has a ceiling and gets expensive fast.</p></div>
<div class="card accent"><p class="muted small">Horizontal</p><h3>Add more machines</h3><p class="small">More servers, containers, a cluster. There is no clear ceiling.</p></div>
</div>

<p class="muted">Now the system is distributed.</p>

<!--
Vertical: you upgrade the hardware you already have. More CPU, more memory, a bigger server. It is simple, but it has a ceiling and gets expensive fast.
Horizontal: you add more machines. More servers, containers, a Kubernetes cluster orchestrating it all. There is no clear ceiling, but now the system is distributed.
And it is in a distributed system that you start to notice how many points of failure there are.
-->

---

## Cloud, Docker, Kubernetes

<div class="cols3">
<div class="card"><h3>Cloud</h3><p class="small">Infrastructure on demand</p></div>
<div class="card"><h3>Docker</h3><p class="small">The application in a box</p></div>
<div class="card"><h3>Kubernetes</h3><p class="small">Replicas and orchestration</p></div>
</div>

<p class="muted">More arrows. More points of failure.</p>

<!--
The store moves to the cloud, the application is packed into containers, and Kubernetes starts orchestrating replicas. That takes away some of the infrastructure pain: fewer machines in the room, fewer heroes up in the middle of the night.
But we are not here to talk about infrastructure. We are here to talk about how the system behaves when things go wrong. Because they always do. And the more distributed the design, the more arrows there are between the parts.
-->

---

## Every arrow is a network call

<svg viewBox="0 0 1100 280" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Two boxes linked by a network arrow that can fail">
  <defs>
    <marker id="arr1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="8" markerHeight="8" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#3b5bdb"/></marker>
  </defs>
  <rect x="80" y="90" width="280" height="100" rx="18" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
  <text x="220" y="150" text-anchor="middle" font-family="Inter, sans-serif" font-size="32" font-weight="700" fill="#1d1f24">Service A</text>
  <line x1="380" y1="140" x2="700" y2="140" stroke="#3b5bdb" stroke-width="4" marker-end="url(#arr1)"/>
  <text x="540" y="118" text-anchor="middle" font-family="Inter, sans-serif" font-size="22" font-weight="600" fill="#d6455d">can fail</text>
  <rect x="740" y="90" width="280" height="100" rx="18" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
  <text x="880" y="150" text-anchor="middle" font-family="Inter, sans-serif" font-size="32" font-weight="700" fill="#1d1f24">Service B</text>
</svg>

<p class="muted">A timeout, the other side is down, a lock holding a resource.</p>

<!--
A call between two services can take too long, the other side may be down, a lock may be holding a resource. Every arrow on an architecture diagram is a network call, and every network call can fail.
-->

---

## The purchase journey

<svg viewBox="0 0 1100 360" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="E-commerce talking to stock, shipping and payment">
  <defs>
    <marker id="arr2" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="8" markerHeight="8" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#3b5bdb"/></marker>
  </defs>
  <rect x="40" y="130" width="280" height="100" rx="18" fill="#3b5bdb" stroke="#3b5bdb" stroke-width="2"/>
  <text x="180" y="190" text-anchor="middle" font-family="Inter, sans-serif" font-size="32" font-weight="700" fill="#ffffff">E-commerce</text>
  <line x1="340" y1="160" x2="520" y2="70" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arr2)"/>
  <line x1="340" y1="180" x2="520" y2="180" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arr2)"/>
  <line x1="340" y1="200" x2="520" y2="290" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arr2)"/>
  <rect x="540" y="20" width="280" height="90" rx="18" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
  <text x="680" y="75" text-anchor="middle" font-family="Inter, sans-serif" font-size="30" font-weight="700" fill="#1d1f24">Stock</text>
  <rect x="540" y="135" width="280" height="90" rx="18" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
  <text x="680" y="190" text-anchor="middle" font-family="Inter, sans-serif" font-size="30" font-weight="700" fill="#1d1f24">Shipping</text>
  <rect x="540" y="250" width="280" height="90" rx="18" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
  <text x="680" y="305" text-anchor="middle" font-family="Inter, sans-serif" font-size="30" font-weight="700" fill="#1d1f24">Payment</text>
  <rect x="880" y="135" width="180" height="90" rx="18" fill="#e6f6ef" stroke="#e6f6ef" stroke-width="2"/>
  <text x="970" y="190" text-anchor="middle" font-family="Inter, sans-serif" font-size="28" font-weight="700" fill="#0f9d6b">200 OK</text>
  <line x1="840" y1="180" x2="868" y2="180" stroke="#0f9d6b" stroke-width="3" marker-end="url(#arr2)"/>
</svg>

<p class="muted">If everyone answers, the customer gets a 200 OK.</p>

<!--
For the rest of the talk, we will use a store split into services. A purchase goes through the e-commerce service, which talks to stock, shipping and payment. If everyone answers, the customer gets a 200 OK.
Transition: "But what if the shipping service is down?"
-->

---

<!-- _class: statement -->

> What if **shipping** is down?

<!--
Leave the question with the audience for a moment.
The most natural answer is to try again. That is where part 2 starts.
-->

---

<!-- _class: divider -->

<div class="kicker">Part 2 · A call that fails</div>

# Retry, fallback, circuit breaker

<!--
Three patterns for a single call: try again the right way, have a plan B, and stop hammering a service that is already down.
-->

---

## Retry: trying again, the right way

<div class="chips accent">
<span>A lost packet</span>
<span>A timeout at a peak</span>
<span>A container restarting</span>
</div>

<p class="muted">Many network failures are temporary.</p>

<!--
The most natural answer is to try again. Many network failures are temporary: a lost packet, a timeout at a busy moment, a container restarting. A second attempt, a moment later, often works.
But it is not about looping forever. There is a right way to do this, and it is older than it looks.
-->

---

## Backoff and jitter

<div class="cols">
<div class="card"><p class="muted small">Exponential backoff</p><h3>1 s, 2 s, 4 s, 8 s</h3><p class="small">The wait grows with each attempt.</p></div>
<div class="card accent"><p class="muted small">Jitter</p><h3>4 s ± 20%</h3><p class="small">A bit of randomness in the interval.</p></div>
</div>

<p class="muted">The idea comes from Ethernet, in the 70s and 80s.</p>

<!--
This idea dates back to the 70s and 80s, when the first shared networks had to deal with collisions. Ethernet, for example, picks a random time to wait before transmitting again, from a range that doubles with each collision. That is where the two ingredients of a good retry come from.
Exponential backoff: the wait grows with each attempt (1 s, 2 s, 4 s, 8 s). That way you do not keep hammering a service that is already struggling.
Jitter: a bit of randomness in that wait, for example 4 s ± 20%. It looks like a detail, but it is what keeps every client from retrying at the same instant.
Without jitter, a thousand clients fail together, wait one second and retry together. The service, which was trying to recover, gets a thousand requests at the same instant and goes down again. That has a name: thundering herd. With ±20%, the peak drops; with full jitter, it drops further. It is the same number of attempts, just spread out.
In the teaching example, the call timeout is 2 seconds, and we only retry connection failures and 5xx errors. A 4xx, such as a 400 or a 422, means the request is wrong: sending the same request three times only produces three errors.
-->

---

## Three caveats

<ol class="steps">
<li>Only retry what is <b>idempotent</b></li>
<li>Count the <b>total time</b></li>
<li>In PHP-FPM, waiting <b>costs a process</b></li>
</ol>

<p class="muted">A 2 s timeout + waits of 1 s and 2 s = 9 s in the worst case.</p>

<!--
Three caveats that apply to any retry.
1. Only retry what is idempotent. Retrying a shipping quote is safe. Retrying a charge, without an idempotency key, can bill the customer twice, because the first attempt may have worked and only the response was lost.
2. Count the total time. With a 2 s timeout and waits of 1 s and 2 s, the worst case is 9 seconds. Someone is waiting on the other side.
3. In PHP-FPM, waiting costs a process. That usleep holds a worker still, doing nothing. In Swoole and Hyperf, the same call only pauses the coroutine, and the process keeps serving other requests.
Retry handles short failures. But what if shipping stays down for ten minutes?
-->

---

## Fallback: an alternative answer

<div class="chips accent">
<span>A default value</span>
<span>A cached response</span>
<span>A value computed beforehand</span>
</div>

<p class="muted">The system does not stop entirely. Graceful degradation.</p>

<!--
Trying three times and returning an error to the customer means no purchase goes through during that time, because of a single service.
This is where fallback comes in, an idea that gained ground in the 90s and 2000s together with high availability and graceful degradation: when a critical service fails, the system does not stop entirely, it returns an alternative answer. It can be a default value, a cached response or a value computed beforehand.
In the store example, if shipping does not answer, you can use the last value calculated for that postcode, or a default rate, for example $29.90 and 10 days. The purchase goes on. The customer may pay a slightly different shipping fee, but they can still buy.
-->

---

## Essential × non-essential

<div class="cols">
<div class="card good"><p class="muted small">Shipping</p><h3>Can have a fallback</h3><p class="small">The last rate for that postcode, or a default rate.</p></div>
<div class="card bad"><p class="muted small">Payment</p><h3>There is no “default payment”</h3><p class="small">Here the fallback is not returning a value. It is stopping safely.</p></div>
</div>

<p class="muted">Fallback is a business decision, not only a technical one. And it needs an alert.</p>

<!--
Two important things about fallback.
It is a business decision, not only a technical one. A default rate may cost the store more than the real one, and someone needs to agree to that. And not everything has a fallback: there is no "default payment".
That is the difference between what is essential and what is not. Shipping can degrade and the purchase still goes through. Payment cannot invent an answer: if it is down, the operation has to stop safely.
And going through the fallback is never a good sign. It keeps the system standing, but it hides the problem. That is why the log in there is not decoration: every fallback needs a metric and an alert, or shipping can stay down for a week with nobody noticing.
-->

---

## Circuit breaker

<p class="muted" style="margin-bottom: 24px;">Stop kicking a dead horse</p>

<div class="cols3">
<div class="card"><h3>Retry</h3><p class="small">Three attempts, waiting for the timeout.</p></div>
<div class="card"><h3>Fallback</h3><p class="small">The customer is served.</p></div>
<div class="card bad"><h3>Underneath</h3><p class="small">Shipping keeps taking a beating.</p></div>
</div>

<!--
With retry and fallback, the customer is served. But look at what is happening underneath: on every purchase, the e-commerce service tries shipping three times, waits for the timeouts, and only then falls back. Multiply that by every purchase, and the shipping service, which is down, keeps getting a flood of calls. It is kicking a dead horse: nobody gains anything, and it gets in the way of the recovery.
Ideally, the shipping service could say "stop trying, I'm still recovering". Since it cannot, the caller has to notice on its own. That is the job of the circuit breaker, inspired by electrical breakers and popularized by Michael Nygard's book Release It!, in 2007. When failures cross a threshold, the breaker "trips" and cuts the calls for a while.
-->

---

## Three states

<div class="flow">
<div class="ok">CLOSED<br><span class="small">Calls go through</span></div>
<span>→</span>
<div class="bad">OPEN<br><span class="small">Straight to fallback</span></div>
<span>→</span>
<div class="wait">HALF-OPEN<br><span class="small">One test call</span></div>
</div>

<p class="muted">Inspired by electrical breakers. Release It!, 2007.</p>

<!--
CLOSED: the normal state. Calls go through, and failures are counted.
OPEN: failures reached the threshold. No call goes out; everything goes straight to the fallback, with no timeout to wait for. The service on the other side gets some breathing room.
HALF-OPEN: the cooldown has passed. One test call is let through. If it works, the circuit closes and life goes on. If it fails, it opens again and the count restarts.
In the lean example: 5 failures in a row open the circuit, and the cooldown is 60 seconds until a test call is let through.
With the circuit open, a purchase no longer waits nine seconds to reach the fallback: it goes straight there. The customer is served faster and shipping stops taking a beating.
-->

---

## Where does the state live?

<div class="cols">
<div class="card warn"><p class="muted small">PHP-FPM</p><h3>Redis or APCu</h3><p class="small">Every request starts from scratch. In memory, the breaker never opens.</p></div>
<div class="card good"><p class="muted small">Swoole and Hyperf</p><h3>In-memory per worker</h3><p class="small">The process stays alive. And ready-made components already exist.</p></div>
</div>

<p class="muted">A circuit opening needs to show up in your monitoring.</p>

<!--
A detail that catches many people in PHP: where does this state live? In PHP-FPM, every request starts from scratch, so an in-memory CircuitBreaker object forgets everything at the end of each request and never opens. The state has to live in a shared place, such as Redis or APCu. In Swoole and Hyperf, the process stays alive between requests, and in-memory state works per worker. Hyperf already has ready-made components for this: hyperf/retry and hyperf/circuit-breaker.
And, as with fallback, a circuit opening is an event someone needs to see. It should raise an alert and show up in your monitoring.
So far, each pattern has dealt with a single call that fails. But a purchase is a set of steps.
-->

---

<!-- _class: divider -->

<div class="kicker">Part 3 · The whole purchase</div>

# Saga

<!--
Third act: consistency when there is no transaction that can span every service.
-->

---

## A purchase is a set of steps

<div class="flow">
<div>Reserve stock</div>
<span>→</span>
<div>Charge payment</div>
<span>→</span>
<div class="bad">Schedule delivery</div>
</div>

<p class="muted">What if delivery fails after the charge?</p>

<!--
Reserve the stock, charge the payment, schedule the delivery. What if the stock was reserved and the payment charged, but delivery failed?
In a single database, that would be a transaction: either everything happens, or nothing does. But each service has its own database, and there is no transaction that can span all of them.
The answer is the saga, described by Hector Garcia-Molina and Kenneth Salem in 1987: a sequence of local transactions in which each action has a matching compensation.
-->

---

## Every action has a compensation

<svg viewBox="0 0 1100 300" xmlns="http://www.w3.org/2000/svg" role="img" aria-label="Saga steps and their compensations">
  <defs>
    <marker id="arr3" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="8" markerHeight="8" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#3b5bdb"/></marker>
    <marker id="arr4" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="8" markerHeight="8" orient="auto"><path d="M0,0 L10,5 L0,10 z" fill="#d6455d"/></marker>
  </defs>
  <rect x="40" y="30" width="280" height="80" rx="16" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
  <text x="180" y="80" text-anchor="middle" font-family="Inter, sans-serif" font-size="28" font-weight="700" fill="#1d1f24">Reserve stock</text>
  <line x1="340" y1="70" x2="420" y2="70" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arr3)"/>
  <rect x="440" y="30" width="280" height="80" rx="16" fill="#ffffff" stroke="#e4e1da" stroke-width="2"/>
  <text x="580" y="80" text-anchor="middle" font-family="Inter, sans-serif" font-size="28" font-weight="700" fill="#1d1f24">Charge</text>
  <line x1="740" y1="70" x2="820" y2="70" stroke="#3b5bdb" stroke-width="3" marker-end="url(#arr3)"/>
  <rect x="840" y="30" width="220" height="80" rx="16" fill="#fdecef" stroke="#fdecef" stroke-width="2"/>
  <text x="950" y="80" text-anchor="middle" font-family="Inter, sans-serif" font-size="28" font-weight="700" fill="#d6455d">Delivery failed</text>
  <rect x="40" y="190" width="280" height="80" rx="16" fill="#fff4e0" stroke="#fff4e0" stroke-width="2"/>
  <text x="180" y="240" text-anchor="middle" font-family="Inter, sans-serif" font-size="24" font-weight="700" fill="#c27803">Release reservation</text>
  <line x1="420" y1="230" x2="340" y2="230" stroke="#d6455d" stroke-width="3" marker-end="url(#arr4)"/>
  <rect x="440" y="190" width="280" height="80" rx="16" fill="#fff4e0" stroke="#fff4e0" stroke-width="2"/>
  <text x="580" y="240" text-anchor="middle" font-family="Inter, sans-serif" font-size="28" font-weight="700" fill="#c27803">Refund</text>
  <line x1="820" y1="230" x2="740" y2="230" stroke="#d6455d" stroke-width="3" marker-end="url(#arr4)"/>
  <text x="950" y="240" text-anchor="middle" font-family="Inter, sans-serif" font-size="20" font-weight="600" fill="#d6455d">compensate in reverse</text>
</svg>

<p class="muted">To the user it looks atomic. The consistency is eventual.</p>

<!--
Reserved stock? The compensation is to release the reservation. Charged? The compensation is to refund.
If delivery fails, the payment is refunded and then the stock is released.
To the user, the operation looks atomic, but technically the consistency is eventual. The system guarantees "all or nothing" not with a real transaction, but with the ability to undo each step if the process fails.
There are two main ways to coordinate a saga.
-->

---

## Two ways to coordinate

<div class="cols">
<div class="card"><p class="muted small">Choreographed</p><h3>Nobody coordinates at the center</h3><p class="small">Each service reacts to events and undoes its own part.</p></div>
<div class="card accent"><p class="muted small">Orchestrated</p><h3>An orchestrator knows the flow</h3><p class="small">It calls in order and fires compensations in reverse.</p></div>
</div>

<p class="muted">Less coupling on one side. A single place to read, on the other.</p>

<!--
In a choreographed saga, nobody coordinates: each service reacts to events and publishes its own. When something fails, whoever notices publishes an event, and each service undoes its own part. On stage I used Hyperf's event dispatcher to keep the example short. Between different services, that event travels over a broker such as Kafka or RabbitMQ, and the listener becomes a consumer. The strength is decoupling: there is no central service that, if it goes down, stops everything. The weakness is that the flow is not written down anywhere. It lives spread across the listeners, and understanding "what happens when an order is cancelled" means looking at several services.
In an orchestrated saga, an orchestrator knows the whole flow: it calls each service in order and, if one fails, it fires the compensations of the steps that already happened, in reverse order. The flow lives in one place, easy to read and to test. In exchange, the orchestrator becomes a central piece: if it goes down, no saga moves.
In a real system, one thing is missing from that picture: storing the saga's state. If the orchestrator dies in the middle of a compensation, it needs to know, when it comes back, where it stopped. And a compensation can also fail, so it needs retry and it needs to be idempotent — the same event may arrive twice. That is why, in production, sagas usually run on tools such as Temporal, or on a carefully kept state table.
-->

---

## Three kinds of saga

<div class="cols3">
<div class="card"><p class="muted small">Epic</p><h3>HTTP, atomic, orchestrated</h3><p class="small">The orchestrator calls and waits. No service talks to another.</p></div>
<div class="card"><p class="muted small">Parallel</p><h3>Kafka, eventual, orchestrated</h3><p class="small">It keeps the orchestrator, but steps can run in parallel.</p></div>
<div class="card"><p class="muted small">Anthology</p><h3>Kafka, eventual, choreographed</h3><p class="small">No orchestrator. Each service handles its own rollback.</p></div>
</div>

<p class="muted">The further to the right, the less coupling — and the harder it is to follow.</p>

<!--
The book Software Architecture: The Hard Parts, by Ford, Richards, Sadalage and Dehghani, organizes sagas by three choices: synchronous or asynchronous communication, atomic or eventual consistency, and orchestrated or choreographed coordination. In the talk I showed three of the combinations.
The Epic Saga is the closest to the orchestrated code: the orchestrator calls each service over HTTP, waits for the answer, and no service talks to another directly. The Parallel Saga keeps the orchestrator but talks through messages, which lets steps run in parallel. The Anthology Saga is pure choreography: no orchestrator, each service keeps its own transaction and handles its own rollback. The further down that list, the less coupling and the more scale, and the harder it is to understand what is going on.
-->

---

<!-- _class: divider -->

<div class="kicker">Part 4 · Closing</div>

# Putting it together

<!--
Last act: the patterns do not show up alone, and each one has a cost.
-->

---

## In a real architecture, they show up together

<div class="cols">
<div class="card accent"><p class="muted small">The call to shipping</p><h3>Retry, breaker, fallback and log</h3><p class="small">Backoff and jitter, a breaker around it, the last calculated value, and a log of every attempt.</p></div>
<div class="card"><p class="muted small">The whole purchase</p><h3>A saga</h3><p class="small">It undoes its steps if something goes wrong along the way.</p></div>
</div>

<p class="muted">Now your systems can fail in peace.</p>

<!--
In a real architecture, these patterns show up together. The call to shipping has retry with backoff and jitter, a circuit breaker around it, a fallback with the last calculated value, and a log of every attempt and failure. The whole purchase is a saga, which undoes its steps if something goes wrong along the way.
"Now your systems can fail in peace."
Leave a second. Then turn the line.
-->

---

<!-- _class: statement -->

> Or maybe **not**.

<!--
Each of these patterns solves one problem and creates another.
-->

---

## Every pattern has a cost

<div class="cols4">
<div class="card"><p class="muted small">Retry</p><h3>Adds load</h3><p class="small">To a service that is already struggling.</p></div>
<div class="card"><p class="muted small">Fallback</p><h3>Hides the failure</h3><p class="small">And may cost money.</p></div>
<div class="card"><p class="muted small">Circuit breaker</p><h3>Needs state</h3><p class="small">And well-tuned thresholds.</p></div>
<div class="card"><p class="muted small">Saga</p><h3>More code</h3><p class="small">Eventual consistency and compensations.</p></div>
</div>

<p class="muted">It is all a trade-off.</p>

<!--
Retry adds load to a service that is already struggling. Fallback hides the failure and may cost money. The circuit breaker needs shared state and well-tuned thresholds, or it opens for no reason or never opens at all. The saga trades a simple transaction for eventual consistency, compensations and a lot more code. It is all a trade-off.
-->

---

<!-- _class: statement -->

> There is no **silver bullet**.

<p class="muted">Figure out what makes sense in your context.</p>

<!--
So do not get attached to patterns. Figure out what makes the most sense in your context: which failure is likely, how much it costs, how long someone can wait, and which wrong answer is acceptable. There is no silver bullet.
Transition: the article has the teaching code and the details that do not fit on stage.
-->

---

<div class="qr-slide">
<div>

<p class="kicker-dark">The written version of this talk</p>

# Design for Failure

<p class="muted">luizschons.com/posts/design-for-failure</p>

</div>
<figure class="qr">
<img src="assets/qr-artigo-en.svg" alt="QR code for the Design for Failure article">
<figcaption>Scan to read the article</figcaption>
</figure>
</div>

<!--
To go deeper: the article at luizschons.com/posts/design-for-failure/ is the written version of this talk, with more room for the details. Leave a few seconds for anyone who wants to scan.
Then go to questions.
-->

---

<!-- _paginate: false -->
<!-- _footer: '' -->

# Questions?

<div class="contacts">
<div><span class="contact-label">Site</span><span class="contact-value">luizschons.com</span></div>
<div><span class="contact-label">LinkedIn</span><span class="contact-value">linkedin.com/in/luiz-schons</span></div>
<div><span class="contact-label">GitHub</span><span class="contact-value">github.com/sschonss</span></div>
</div>

<!--
Open for questions. Leave this slide up during Q&A.
-->
