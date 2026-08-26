---
title: "From Excalidraw to a Fork of AFFiNE: Shipping a Whiteboard on a Startup Budget"
date: 2026-08-26 10:00:00
tags: [affine, excalidraw, canvas, architecture, self-hosting]
---

## 1. The feature nobody warns you about

The product I work on is an e-learning platform. Course creators lay out material on visual boards, learners work on those boards and submit their work, and reviewers leave feedback directly on the canvas. On the roadmap this looked like one feature among many: "add a collaborative canvas."

It is not one feature. A canvas is a product-within-a-product. It has its own rendering engine, its own data model, its own real-time sync problem, its own undo stack, its own zoom/pan/selection interaction model that users compare—consciously or not—against the best tool they've ever used. And for most users today, that reference point is FigJam.

This post is the story of how we got there: from Excalidraw, through a failed bet on an editor library, to running our own fork of AFFiNE. The interesting part isn't the specific tools—it's how one hard constraint made the decision for us.

## 2. v0: the options on the table

When we first scoped the canvas, two options made the shortlist.

**Build it ourselves on Konva.** A canvas rendering library like Konva gives you full control: your data model, your interactions, your look. It also gives you full responsibility for everything I listed above. For a small startup team with a product to ship, "we'll build our own FigJam" is a multi-quarter commitment disguised as a library choice. We passed.

**Excalidraw.** Batteries included: drawing tools, shapes, images, export, a well-maintained open-source project with a permissive license. We could embed it, wire up persistence, and ship. So we did, and for a while it did its job.

## 3. The rejection

The feedback that eventually killed it wasn't about bugs or missing tools. It was about how it *looked*.

Excalidraw's signature hand-drawn aesthetic—wobbly lines, sketchy fills—is a deliberate design choice, and a good one for its intended use: it signals "this is a rough whiteboard, don't overthink it." But our creators weren't sketching. They were building course material that learners would study and that reflects on the creator's professionalism. On that content, the hand-drawn style read as unpolished. Almost toy-like.

What users kept asking for was, in their words, "something like FigJam": clean lines, precise shapes, a professional finish. Not FigJam's feature set—FigJam's *feel*.

That's worth pausing on, because it's the transferable lesson of this chapter: users don't evaluate an embedded canvas against your product, they evaluate it against the canvas tools they already know. If your audience lives in FigJam, an intentionally sketchy whiteboard will feel wrong no matter how well it works.

## 4. The bake-off, and the constraint that actually decided it

So we surveyed the field for a clean, FigJam-like canvas.

**tldraw** was the best technical fit, full stop. Clean visuals, excellent interaction model, designed to be embedded. If the decision had been purely technical, this post would be about tldraw. But tldraw's business license comes with a fee that a startup-stage company couldn't justify for one feature.

**Miro and FigJam embeds** had the same problem in a different shape: per-seat SaaS pricing that scales with our user count, plus no real control over the experience inside the frame.

Notice what happened here: the decision tree wasn't pruned by a feature comparison. It was pruned by a single hard constraint—**licensing cost**. Every technically-excellent option fell to the same axe. That's the pattern I'd point other teams at: figure out your hard constraint *first*, because it eliminates candidates faster than any bake-off spreadsheet, and it saves you from falling in love with an option you can't afford.

What survived was **AFFiNE**: an open-source knowledge base with an "edgeless" canvas mode that is visually and interactionally the closest thing to FigJam we found—and free to self-host.

## 5. The detour: building on BlockSuite

Our first instinct was *not* to take the whole AFFiNE app. AFFiNE's editor is built on BlockSuite, its underlying editor framework, and the tidy-looking plan was: take the library, build our own canvas component on top, keep full control of the shell. Best of both worlds—AFFiNE's editor quality, our own integration surface.

We ran a spike, and the spike killed the plan within days. Two findings:

1. **Customization was much harder than the architecture diagram suggested.** BlockSuite is built to power AFFiNE; bending it to a different host app meant fighting assumptions baked in at every layer.
2. **The standalone repo had gone quiet.** Development had moved into the AFFiNE monorepo, and the library we'd be betting our canvas on hadn't seen meaningful standalone updates in a long time. We had been burned by this smell before, and building a core feature on an effectively unmaintained foundation was not a bet we were willing to make.

The spike cost us a few days. Committing to the library and discovering all this six months in would have cost us the roadmap. If there's a second transferable lesson in this story, it's this: when your plan is "we'll build on their internal library," spike it before you schedule it.

## 6. The actual decision: fork the app, not the library

So we inverted the approach. Instead of extracting AFFiNE's editor into our app, we would run AFFiNE itself—self-hosted, as its own service—and embed it.

That meant maintaining a fork, and the scope of the fork was the most important decision of the whole project. We kept it deliberately minimal: **embed hardening only**. Our patches strip the UI that makes no sense inside an embedded canvas—app navigation, workspace chrome, anything that would remind the user they're inside "another product." We do not touch the editor internals. The canvas engine, the data model, the sync layer—all of it stays upstream's code.

I'd summarize the principle as: **fork the shell, not the guts.** Every line you change in a fork is a line you re-pay on every upstream merge. Changes to the outer shell are small, localized, and cheap to carry forward. Changes to editor internals are exactly the opposite—and they're also exactly what made the BlockSuite path fail.

## 7. Making the embed feel native

An iframe to a self-hosted app solves rendering, but the naive version has an obvious seam: the user logs into your platform, opens a board, and the iframe greets them with *another* login screen. Fixing that seam is where the actual integration work lives.

Our setup relies on the two apps being siblings on one domain:

```text
             app.example.com                 canvas.example.com
          (e-learning platform)              (forked AFFiNE)
                    │                               ▲
   1. user logs in  │                               │ 4. iframe loads doc;
                    ▼                               │    cookie sent along,
              platform API ──────────────────────►  │    no second login
                    │       2. server-to-server:    │
                    │          sync user, create    │
                    │          canvas session       │
                    ▼                               │
   3. session cookie set with Domain=.example.com ──┘
```

In words:

1. The user logs into the platform as usual.
2. During login, our backend talks server-to-server to the AFFiNE instance: it ensures a matching user exists there and obtains a session for them.
3. The backend sets that canvas session as a cookie scoped to the *parent* domain (`.example.com`), so the browser will send it to any sibling subdomain.
4. When the app renders a board, it embeds an iframe pointing at a specific workspace and document on `canvas.example.com`. The browser attaches the domain-wide cookie, AFFiNE recognizes the session, and the canvas just appears—no auth wall, no second identity.

None of this is AFFiNE-specific. It's a general pattern for embedding any self-hosted, session-based app into your product: put it on a sibling subdomain, sync users server-to-server at login, and share the session via a domain-scoped cookie. The user experiences one product; behind the scenes there are two.

## 8. What this actually costs

A fork story that ends with "and it works great" is an advertisement, so here is the bill we knowingly pay:

- **Fork maintenance.** Every upstream AFFiNE release has to be merged past our patches. The embed-hardening-only scope keeps this manageable, but it's a recurring tax, not a one-time cost.
- **Operational burden.** We now run an entire collaborative-editing stack—server, real-time sync, storage—to power what the roadmap once called "a canvas feature." Self-hosted is free like a puppy is free.
- **The iframe is a sealed box.** Deep integration—custom block types, product features reaching into the canvas, the canvas reaching out—ranges from awkward to impossible across the frame boundary. We traded integration depth for isolation.

Here's the thing, though: all three of these are *soft* costs. They're ongoing taxes we can budget for, staff for, and revisit. A per-seat license fee at a price we couldn't afford was a *hard* constraint—no amount of engineering effort makes it go away. Given the choice between problems we can work on and a problem we can't, we'll take the puppy.

## 9. Takeaways

- **Find your hard constraint first.** Ours was licensing cost, and it pruned the options faster than any feature matrix. Rank constraints by hardness before you rank candidates by quality.
- **Your canvas is judged against FigJam, not against your mockups.** Aesthetic fit with your users' reference tools is a real requirement, not polish.
- **Spike before you bet on a library**—especially one extracted from a larger app. A few days of spiking on BlockSuite saved us months.
- **If you must fork, fork the shell, not the guts.** Keep patches at the edges where upstream merges stay cheap.
- **Sibling subdomain + server-side user sync + domain-scoped session cookie** is a reusable recipe for embedding a self-hosted app without a second login.
