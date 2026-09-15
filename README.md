# A no-backend-experience-required guide to bolting a real AI chatbot onto any Framer site — portfolio, SaaS, law firm, doesn't matter.

---

Here's a confession: I don't particularly enjoy writing backend code. I like Framer precisely because it lets me skip that entire universe of servers, environment variables, and cryptic error codes. But at some point I looked at my portfolio and thought: *what if, instead of visitors reading about me, they could just ask me things?*

Not email-me-and-wait-three-days ask. Instant, always-on, "hey are you free for projects" ask, answered the way I'd actually answer it.

So I built one. It's called **Odin LLM**, it's open source, and this article walks you through installing it on your own Framer site — even if the words "serverless function" currently mean nothing to you. By the end, you'll have a slick, branded, draggable AI chat widget that knows everything you tell it and nothing you didn't.

Repo's here, for reference: **[github.com/abduseK/odin-chatbot](https://github.com/abduseK/odin-chatbot)**

Let's get into it.

---

## First, what is this thing actually doing?

Before touching any code, it helps to understand the shape of what you're building, because it's genuinely simpler than it sounds. There are only three moving parts:

1. **The frontend widget** — a single Framer code component. This is the chat bubble, the popup panel, the typing animation, all of it. It lives entirely inside Framer.
2. **The backend** — a tiny serverless function (one file, no server to babysit) that sits between your widget and the AI provider. Its whole job is to hold your API key safely and forward questions to the AI.
3. **The AI provider** — the actual language model answering questions. We're using free tiers here, so this costs you nothing.

Here's the flow, spelled out:

> Visitor types a question → Framer widget sends it to your backend → backend adds your bio/background info and forwards it to the AI → AI replies → backend sends the reply back → widget displays it, one flickering character at a time like a very polite hacker movie.

That's it. No database, no user accounts, no infrastructure to maintain. If that sentence made your eyes glaze over, good news — you don't need to understand *why* it works, just *that* it works, and how to wire it up.

---

## Step 1: Decide how much backend you want to touch

This is the fork in the road, and it's worth pausing on because it changes how much setup you're signing up for.

**Option A — Use my hosted backend (fastest, zero setup)**
I run a working backend already. If you just want the widget on your site today, you can point it at my endpoint and skip straight to Step 4. The tradeoff: you're sharing infrastructure with everyone else doing this, and you can't customize the "about me" content it draws from (it'll be about me, not you — fine for testing, not for your actual site).

**Option B — Deploy your own copy from the repo (recommended, still genuinely easy)**
You get your own backend, your own AI usage limits, and — critically — you control what the bot knows. This is what the rest of this guide covers, and it takes about ten minutes even if you've never deployed anything before in your life.

**Option C — Bring your own backend**
If you already have a Vercel project, or prefer Cloudflare Workers, or whatever your team already runs — the repo's `api/chat.js` file is small and readable enough to drop into any Node-compatible serverless setup. Everyone else, keep reading.

---

## Step 2: Get your own copy of the backend running

This is the "scary" part that turns out not to be scary. Here's the whole process:

1. Go to the repo: **github.com/abduseK/odin-chatbot**
2. Click **Fork** (top right) — this makes your own copy under your GitHub account, so you can edit it freely.
3. Go to **[vercel.com](https://vercel.com)** and sign up using "Continue with GitHub" (this links the two automatically).
4. Click **Add New Project**, select your forked repo, and hit **Deploy**.

Vercel reads the repo, sees the `api/chat.js` file, and automatically turns it into a live endpoint. No configuration screens, no build settings to fiddle with. In about 60 seconds you'll have a URL that looks like:

```
https://your-project-name.vercel.app/api/chat
```

Save that URL. You'll need it in Step 5.

---

## Step 3: Get a free AI API key

The backend needs an actual AI model to talk to. We use **OpenRouter**, because unlike some providers, it doesn't demand a credit card, doesn't organize you into confusing "teams" you can't escape, and has genuinely free models available.

1. Go to **openrouter.ai** and sign up (Google or GitHub login is fastest).
2. Click your profile → **Keys** → **Create Key**.
3. Copy the key it gives you. Treat it like a password — never paste it into your Framer component or anywhere public.

---

## Step 4: Tell your backend the key (without ever touching a terminal)

Back in Vercel:

1. Open your deployed project → **Settings → Environment Variables**.
2. Add a new variable:
   - **Key:** `OPENROUTER_API_KEY`
   - **Value:** the key you just copied
3. Go to **Deployments** → find the latest one → click the three dots → **Redeploy**.

That last redeploy step matters — environment variables only take effect after a fresh deployment, and skipping it is the single most common reason people think "it's broken" when it's actually just waiting for a restart.

---

## Step 5: Make the bot actually know things about *you*

Here's the part that makes this worth doing instead of using a generic chatbot widget: the bot's knowledge lives in one plain-English block of text in the code, called `ABOUT_ME`, sitting at the top of `api/chat.js`.

Open that file in your forked repo (directly on GitHub — click the pencil icon to edit), and replace the placeholder content with your own background, organized loosely by topic:

```
IDENTITY: who you are, what you do, where you're based.
BACKGROUND: your story — education, how you got here.
WHAT YOU DO: services, focus areas, specialties.
FEATURED WORK: projects, case studies, results — with real links if you have them.
CONTACT: where people can actually reach you.
```

No special formatting required — write it like you're briefing a very smart new employee on your first day. The more specific and organized this is, the better the bot's answers will be. Commit the change, and Vercel redeploys automatically.

This is also the file where the bot's *personality* lives — there's a `systemPrompt` further down that controls tone (currently: warm, concise, no fake humor, and — importantly — a hard rule against inventing personal details it wasn't told.

If you're setting this up for a law firm, SaaS product, or agency instead of a personal portfolio, this is exactly where that adaptation happens — swap "IDENTITY" for firm info, "FEATURED WORK" for case results or product features, and adjust the tone instructions to match your brand voice.

---

## Step 6: Drop the widget into Framer

Now the fun part — the visible bit.

1. Copy the component code from https://www.framer.com/community/resources/components/odin-ai-chatbot
2. Paste the component onto your canvas, anywhere convenient — it renders as a small pill/icon trigger and manages its own popup, so its exact position on the page barely matters.
3. In the right-hand **Properties panel**, you'll see a genuinely long list of editable fields. The two that matter most right away:
   - **Backend URL** → paste the `.../api/chat` URL from Step 2
   - **Assistant Name** → whatever you want the bot called (this flows through to the header, the input placeholder, and how it refers to itself)

Everything else — colors, panel size, position on screen, animation speed, whether there's a left-side image, hover colors, the works — is a property control you can play with visually. No code editing needed beyond this point.

---

## Step 7: Test it like a nervous parent

Open your Framer preview, click the chat trigger, and ask it something real. A few things to check:

- Does it answer using *your* actual info, not generic filler? (If not, double check your `ABOUT_ME` block saved and redeployed.)
- Does it handle a follow-up question naturally? (It should — the widget sends conversation history with each message, so it remembers context within the same session.)
- Does clicking a suggested question work, and do new suggestions appear after?
- Try it on mobile view. The layout should adapt — image panel disappears on small screens, trigger can collapse to icon-only.

If something errors out, it almost always falls into one of two buckets: the Backend URL is mistyped, or the environment variable in Vercel wasn't set (or wasn't redeployed after setting). Both are two-minute fixes.

---

## What you actually get, once it's live

- **Persistent-within-session memory** — it remembers the conversation as you go, resets cleanly on page refresh (so nobody's chat history lingers awkwardly)
- **Honest by design** — it won't invent facts about you it wasn't told; it deflects instead
- **Fully brandable** — colors, sizing, position, fonts, all editable visually in Framer
- **Rate-limit aware** — built-in question caps and graceful "come back later" messaging instead of ugly error dumps if the free AI tier gets maxed out
- **Genuinely free to run** — no card required for OpenRouter's free tier, no Vercel cost for a project this size

---

## Where this goes from here

Once it's running, the same pattern scales to basically any site that wants visitors to *ask* instead of *dig*: a SaaS landing page answering "does this integrate with X," a law firm site answering "do you handle Y kind of case," an agency site answering "what's your process." Same widget, same backend pattern, different `ABOUT_ME` block.

You just built the thing most people assume needs a dev team and a monthly SaaS subscription — for free, in an afternoon, using a Framer template and a public GitHub repo.

Go forth and let your website talk back.

**Repo:** [github.com/abduseK/odin-chatbot](https://github.com/abduseK/odin-chatbot)
