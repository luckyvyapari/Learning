# Cloudflare Stack — Explained Like You're 10

A simple guide to the 12 core Cloudflare building blocks: what each one is, when to use it, and where the official docs live.

---

## The Big Picture

Imagine the internet is a giant city.

Normally, your website lives in **one building** in **one city** (a server). If a kid in Japan wants to visit your website, they have to travel all the way to that one building. That takes time.

Cloudflare says: *"Why not put copies of your stuff in 300+ cities around the world?"*

So when the kid in Japan visits, they walk to the building **next door** instead. Fast.

That network of 300+ cities is called the **edge**. Everything below runs on it.

📖 Official start page: https://developers.cloudflare.com/

---

## 1. Workers — The Robot Helper

**Kid version:** A tiny robot that lives in every one of those 300 cities. When someone visits your site, the nearest robot wakes up, does a small job (like "give them the homepage" or "check their password"), and goes back to sleep.

**Why it's cool:** The robot wakes up in less than a millisecond. And you only pay when it's awake.

**Grown-up version:** Serverless JavaScript/TypeScript/Python/Rust functions running on V8 isolates at the edge. No cold starts like traditional serverless. This is the *center* of the whole stack — almost everything else is something Workers talk to.

**Use it for:** APIs, websites, redirects, auth checks, image resizing, webhooks — the "brain" of your app.

```js
export default {
  async fetch(request, env, ctx) {
    return new Response("Hello from the nearest city to you!");
  }
};
```

📖 Docs: https://developers.cloudflare.com/workers/

---

## 2. Durable Objects — The One Special Robot Who Remembers

**Kid version:** Normal Workers forget everything the moment they finish. Durable Objects are special robots who **remember**, and there is only **ever one of them** for each thing.

Think of a multiplayer game room. If 5 friends join "Room #42", there must be **exactly one** referee robot for Room #42 — otherwise the friends would see different scores. Durable Objects guarantee that one referee exists, and everyone talks to it.

**Grown-up version:** Single-instance, globally-unique stateful compute with attached storage. Solves coordination and consistency problems that stateless Workers can't. Each object has a unique ID; requests for that ID always route to the same instance.

**Use it for:** Chat rooms, multiplayer games, live collaboration (like Google Docs), rate limiting, WebSocket connections, counters that must be exactly right.

📖 Docs: https://developers.cloudflare.com/durable-objects/

---

## 3. KV — The Sticky Note Wall

**Kid version:** A giant wall where you stick notes. Each note has a **name** ("user-123") and a **message** ("Ravi, likes pizza"). You ask for the note by name, and you get it back super fast.

But there's a catch: if you change a note, it takes a few seconds for every city in the world to see the new version. So don't use it for things that must be right *this instant*.

**Grown-up version:** Globally distributed key-value store. **Eventually consistent** — reads are cached at the edge and extremely fast; writes propagate globally in seconds. Optimized for read-heavy workloads.

**Use it for:** Config, feature flags, cached API responses, routing tables, session lookups — things read a million times and written rarely.

**Do NOT use it for:** Bank balances, inventory counts, anything where a few seconds of stale data breaks things.

📖 Docs: https://developers.cloudflare.com/kv/

---

## 4. R2 — The Giant Toy Box

**Kid version:** A huge box where you keep big things: photos, videos, PDFs, backups. You put stuff in, you take stuff out.

The magic trick: most companies charge you money **every time you take a toy out**. Cloudflare charges you **nothing** to take things out. That fee is called "egress," and R2 has zero of it.

**Grown-up version:** S3-compatible object storage with **zero egress fees**. Works with existing S3 SDKs and tools by changing the endpoint.

**Use it for:** User uploads, images, video, ML model files, data lakes, static assets, backups — anything large and file-shaped.

📖 Docs: https://developers.cloudflare.com/r2/

---

## 5. D1 — The Notebook With Neat Rows

**Kid version:** A notebook with tables in it — like a class attendance register. Rows and columns. You can ask questions like "who is absent today?" and it answers.

**Grown-up version:** Serverless SQL database built on SQLite. Supports real SQL, indexes, joins, transactions, and time-travel (point-in-time restore). Reads can be served from read replicas near the user.

**Use it for:** Users, orders, posts, comments — normal relational app data where you want SQL queries.

**Keep in mind:** It's SQLite-based, so it's built for many small-to-medium databases (one per tenant/user is a common pattern) rather than one enormous multi-terabyte database. Check the current size and query limits in the docs before designing around it.

```sql
SELECT name FROM users WHERE city = 'Mumbai';
```

📖 Docs: https://developers.cloudflare.com/d1/

---

## 6. Queues — The "I'll Do It Later" Basket

**Kid version:** Someone hands you 500 letters to mail. You don't have to mail them all right now while they watch. You drop them in a basket, say "done!", and a helper mails them one by one in the background.

**Grown-up version:** Managed message queue. Producers push messages, consumer Workers pull them in batches, with automatic retries and a dead-letter queue for messages that keep failing. Decouples slow work from the fast request path.

**Use it for:** Sending emails, generating thumbnails, syncing to a third-party API, background jobs, smoothing out traffic spikes.

📖 Docs: https://developers.cloudflare.com/queues/

---

## 7. Pub/Sub — The School Announcement Speaker

**Kid version:** One person speaks into the microphone, and **everyone** in every classroom hears it at the same time. Nobody has to ask "any news?" — the news comes to them.

Difference from Queues: a Queue letter goes to **one** helper. A Pub/Sub announcement goes to **everyone listening**.

**Grown-up version:** A managed MQTT-compatible message broker for publish/subscribe messaging, aimed at IoT fleets and many-listener fan-out.

⚠️ **Heads up:** Pub/Sub has been the least-stable product on this list — it has spent a long time in limited beta and its availability has changed. **Check the docs page for its current status before building on it.** For most fan-out needs today, people use Queues, Durable Objects (with WebSockets), or Workers + an external broker instead.

📖 Docs: https://developers.cloudflare.com/pub-sub/

---

## 8. Vectorize — The "Find Me Something Similar" Machine

**Kid version:** Normal search finds the **exact word** you typed. This one finds things that **mean the same thing**.

Type "big cat" and it finds a page about tigers — even though the word "tiger" was never typed. It does this by turning every sentence into a list of numbers (a "vector"), then finding numbers that sit close together.

**Grown-up version:** A vector database for storing and querying embeddings with approximate nearest-neighbour search, plus metadata filtering. The retrieval half of RAG (Retrieval-Augmented Generation).

**Use it for:** Semantic search, recommendations, "related articles," and giving an AI chatbot your own documents to answer from.

📖 Docs: https://developers.cloudflare.com/vectorize/

---

## 9. Workers AI — The Robot's Brain

**Kid version:** Cloudflare put real AI models on GPUs in those same 300 cities. Your robot helper can now say "hey, describe this picture" or "translate this to Hindi" or "write me a summary" — and get an answer from a machine sitting right nearby.

**Grown-up version:** Run open models (text generation, embeddings, image generation, speech-to-text, translation, classification) on Cloudflare's GPU network via a Worker binding or REST API. Pay per usage, no GPU to manage.

**Use it for:** Chatbots, summarizing, generating the embeddings you store in Vectorize, transcribing audio, moderating content.

```js
const answer = await env.AI.run('@cf/meta/llama-3.1-8b-instruct', {
  prompt: "Explain gravity to a 5 year old"
});
```

📖 Docs: https://developers.cloudflare.com/workers-ai/
📖 Model catalog: https://developers.cloudflare.com/workers-ai/models/

---

## 10. AI Gateway — The Traffic Cop for AI

**Kid version:** Imagine your app talks to lots of different AI helpers (OpenAI, Anthropic, Google, Cloudflare's own). AI Gateway sits in the middle like a traffic cop and:

- **Writes down** every question and answer (logs)
- **Remembers** repeat questions so you don't pay twice (caching)
- **Stops** anyone asking too many questions too fast (rate limiting)
- **Switches** to a backup helper if one is asleep (fallbacks)

**Grown-up version:** An observability and control proxy in front of any LLM provider. You change your API base URL to the gateway URL; you get analytics, caching, rate limiting, retries/fallbacks, and cost tracking without changing your app code.

**Use it for:** Every production AI app. Cheap to add, saves real money on cache hits, and you finally get to *see* what your AI is doing.

📖 Docs: https://developers.cloudflare.com/ai-gateway/

---

## 11. Pages — The Easy Website Publisher

**Kid version:** You put your website's files on GitHub. Cloudflare watches that folder. Every time you change something, it automatically builds your site and puts it in all 300 cities. You do nothing.

**Grown-up version:** Git-connected build-and-deploy for frontend apps (React, Next.js, Astro, Vue, plain HTML). Automatic preview URLs for every branch and pull request, plus Pages Functions for backend logic.

⚠️ **Heads up:** Cloudflare now recommends **Workers with static assets** as the path forward for new projects — Workers absorbed Pages' features and gets the new development. Pages still works and is still supported, but read the current migration guidance before starting something new.

📖 Pages docs: https://developers.cloudflare.com/pages/
📖 Workers static assets: https://developers.cloudflare.com/workers/static-assets/

---

## 12. Zero Trust — The Bouncer at Every Door

**Kid version:** Old security was like a castle wall: get inside once, and you can walk into any room. Zero Trust is different — there's a **bouncer at every single door**, and he checks your ID every single time. Even if you're already inside. Even if you're the boss.

**Grown-up version:** Cloudflare One / Zero Trust — identity-aware access control (Access), secure web gateway for outbound traffic (Gateway), tunnels that expose internal apps without opening firewall ports (Tunnel), browser isolation, and CASB. Nothing is trusted by default; every request is authenticated and authorized.

**Use it for:** Replacing a VPN, protecting internal dashboards, locking down staging environments, contractor access, filtering employee web traffic.

📖 Docs: https://developers.cloudflare.com/cloudflare-one/

---

## How They Fit Together

A realistic AI-powered app, end to end:

```
User's browser
      |
      v
[ Pages / Workers static assets ]  <- the frontend they see
      |
      v
[ Workers ]  <- the brain, runs your code near the user
      |
      +--> [ D1 ]              users, orders, comments (SQL)
      +--> [ KV ]              config + cached responses (fast reads)
      +--> [ R2 ]              uploaded files, images, video
      +--> [ Durable Objects ] live chat room, presence, rate limits
      +--> [ Queues ]          "send the email later"
      +--> [ Vectorize ]       "find docs similar to this question"
      +--> [ Workers AI ]      "now write an answer using those docs"
            |
            v
      [ AI Gateway ]  <- logs, caches, and rate-limits every AI call
      
[ Zero Trust ]  <- guards the admin panel so only staff get in
```

---

## Quick Cheat Sheet

| I want to... | Use |
|---|---|
| Run code near the user | **Workers** |
| Keep one shared live state (chat, game, counter) | **Durable Objects** |
| Read a small value very fast, very often | **KV** |
| Store big files cheaply | **R2** |
| Query structured data with SQL | **D1** |
| Do slow work in the background | **Queues** |
| Broadcast to many listeners / IoT | **Pub/Sub** *(check status first)* |
| Search by meaning, not keywords | **Vectorize** |
| Run an AI model | **Workers AI** |
| Watch, cache, and control AI calls | **AI Gateway** |
| Deploy a frontend from Git | **Pages** *(or Workers static assets)* |
| Let only the right people in | **Zero Trust** |

---

## Two Rules of Thumb

**1. Pick storage by how you read it, not how you write it.**
Read constantly, write rarely → **KV**.
Ask questions with filters and joins → **D1**.
Big and file-shaped → **R2**.
Must be exactly right, right now, shared between people → **Durable Objects**.

**2. Anything slow should not make the user wait.**
Push it to **Queues** and answer the user immediately.

---

## Where to Start

1. **Install the CLI:**
   ```bash
   npm create cloudflare@latest
   ```
2. **Read the Workers tutorial** — everything else plugs into Workers, so learn that first: https://developers.cloudflare.com/workers/get-started/guide/
3. **Learn bindings** — how a Worker gets access to KV, R2, D1, etc.: https://developers.cloudflare.com/workers/runtime-apis/bindings/
4. **Check limits and pricing before you design:** https://developers.cloudflare.com/workers/platform/limits/

---

## Official Links (All in One Place)

| Product | Documentation |
|---|---|
| Workers | https://developers.cloudflare.com/workers/ |
| Durable Objects | https://developers.cloudflare.com/durable-objects/ |
| KV | https://developers.cloudflare.com/kv/ |
| R2 | https://developers.cloudflare.com/r2/ |
| D1 | https://developers.cloudflare.com/d1/ |
| Queues | https://developers.cloudflare.com/queues/ |
| Pub/Sub | https://developers.cloudflare.com/pub-sub/ |
| Vectorize | https://developers.cloudflare.com/vectorize/ |
| Workers AI | https://developers.cloudflare.com/workers-ai/ |
| AI Gateway | https://developers.cloudflare.com/ai-gateway/ |
| Pages | https://developers.cloudflare.com/pages/ |
| Zero Trust | https://developers.cloudflare.com/cloudflare-one/ |
| Wrangler CLI | https://developers.cloudflare.com/workers/wrangler/ |
| Pricing | https://developers.cloudflare.com/workers/platform/pricing/ |
| Cloudflare Blog | https://blog.cloudflare.com/ |

---

*Note: Cloudflare ships fast and product statuses change (especially Pub/Sub and Pages). Treat the links above as the source of truth over this document.*
