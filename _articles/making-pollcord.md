---
layout: post
title: "Building a Discord Poll Wrapper"
date: 2025-02-15
author: Myst1cS04p
tags: [discord, api, python, library design, tooling]
color: "#1b76ffff"
excerpt: "My first async Python project, first time writing unit tests, first time actually shipping something meant to be used as a library. It worked perfectly in isolated testing. Then I tried integrating it into my actual bot, and everything fell apart."
description: "My first async Python project, first time writing unit tests, first time actually shipping something meant to be used as a library. It worked perfectly in isolated testing. Then I tried integrating it into my actual bot, and everything fell apart."
reading_time: "5 min read"
css: assets/css/articles.css
cover: assets/img/articles/covers/pollcord.gif
title-image: assets/img/articles/titles/pollcord.png
---

## Why I Built It

Discord recently introduced a native polling system. It's clean, built into the platform, and honestly much nicer than the reaction-based polls most bots use. The problem? There wasn't a simple, lightweight Python wrapper that let you create and manage these polls without dragging in an entire Discord bot framework.

Most existing solutions assumed you were already using a large library like `discord.py` or similar. That's fine if you're building a full bot, but not great if you just want to integrate polling into an existing async system or microservice.

So I built **Pollcord**: a small wrapper around Discord's poll API. It could create polls, track state, run callbacks when polls ended, fetch vote counts. Lightweight. Framework-agnostic. Exactly what I wanted.

It worked perfectly in isolated testing.

Then I tried integrating it into my actual bot, and everything fell apart.

This is a postmortem on what went wrong, not a library announcement. Pollcord is not stable, not production-ready, and I didn't have immediate plans to finish it when I started writing this. What I do have is a reasonably clear understanding of why async systems fail in ways that unit tests don't catch, which is the part worth writing about.

> **Update:** About a week after writing this, I did end up doing a lot more work on Pollcord. It still hasn't gone through live testing, but it's considerably more stable now.

---

## Context

This project was a pile of firsts for me.

First time writing async Python. First time actually writing unit tests and setting up CI/CD workflows. And the first time I'd managed to fully execute the vision of building something as a *library* - something other code consumes as a dependency.

That last one matters more than it sounds. Writing a library is a different mindset than writing an application. An application can make assumptions about its environment. A library can't. The caller controls the execution context, the event loop, the timing, and a dozen other things you'd normally take for granted. I didn't fully appreciate that going in.

I was also in that phase again as a developer where you look at polished, stable libraries and think: "I could do that." I saw the Discord poll API, saw that nothing lightweight existed for it, and said "Pfft. I'll just make it."

It was meant to be simple.

It was not simple T-T

The initial build actually went surprisingly smoothly. I designed it like an event system, inspired by how I'd seen Unity handle events, since that felt natural and clean to me. I could create polls, let them expire, trigger callbacks, fetch results. Everything behaved exactly as expected.

Confident, I released a beta. Then I tried integrating it into my actual bot.

What worked in controlled testing started breaking in real usage almost immediately:

- Polls ending twice
- Callbacks firing inconsistently
- End logic creating weird loops
- Edge cases I hadn't begun to conceptualize

What didn't help was my callback system, which was literally three different functions at three different layers of the system, all capable of triggering a callback, and all named essentially the same thing. Every debugging session started with ten minutes of just figuring out which one had fired.

---


## Technical Challenges

### 1. The Two-Clock Problem

Discord manages poll expiration on its servers. My wrapper also tracked duration locally to know when to fire the callback. Two independent clocks. On paper that's fine; however, in practice it meant `_schedule_end` was setting `self.ended = True` inline, completely separate from the `end()` method that was supposed to be the single source of truth for poll termination.

So you'd end up with situations where the local scheduler fired, set `ended = True`, ran the callback, and then a manual `end_poll()` call came in a fraction of a second later, didn't see `ended = True` yet due to timing, and ran the callback again. Or the reverse: Discord expired the poll, the local clock hadn't fired yet, and the callback never ran because the code path that checked `ended` was in the wrong place.

The fix was giving Discord's clock priority and routing everything through a single `end()` method. Once that was the only place that could set `ended = True` and trigger the callback, the two clocks stopped conflicting.

---

### 2. The Recursion Trap I Built On Purpose

The old `Poll.end()` had an optional `client` parameter:

```python
async def end(self, client: Optional["PollClient"] = None):
    if not self.ended:
        if client:
            await client.end_poll(self)
        self.ended = True
        ...
```

And `PollClient.end_poll()` called `await poll.end()` after hitting the API.

So: `poll.end(client)` ->> `client.end_poll(poll)` ->> `poll.end()`. A full loop. I built it thinking it was clean separation, in the sense that, the poll handles its own lifecycle, the client handles the API call. What I'd actually built was a recursion trap that would either blow up immediately or, worse, silently eat API calls and produce duplicate callbacks depending on where in the cycle the `ended` flag got checked.

Async recursion through network calls doesn't fail immediately. It just produces increasingly wrong behavior until you stare at logs long enough to trace the call stack manually.

---

### 3. The Race Condition Hidden in Sequential Loops

`get_vote_users` and `get_vote_counts` both used a plain `for` loop to fetch results per option:

```python
for index in range(len(poll.options)):
    users = await self.fetch_option_users(poll, index)
    results.append(users)
```

Sequential. One request at a time. Which meant if you called both at roughly the same time, say, a scheduled expiry and a manual results fetch landing within the same event loop tick, then they'd both be making overlapping API calls with no coordination between them. Combined with the callback system that could fire from multiple places, you'd get duplicate results and state that diverged depending on which awaited call returned first.

---

### 4. The Rate Limiter That Assumed Everything

The original retry logic looked like this:

```python
if r.status == 429:
    data = await r.json()
    wait_time = data["retry_after"]
    await asyncio.sleep(wait_time)
```

I later realized there there were actually three assumptions baked in here: the 429 response is always JSON, `retry_after` is always present in the body, and there's no such thing as a global rate limit. Discord violates all three of these regularly. Sometimes the rate limit response is plain text. Sometimes `retry_after` is in the headers rather than the body. Sometimes you've hit a global limit that applies across all endpoints, not just the one you're hammering.

Each assumption was a separate crash waiting to happen. And because the retry logic lived inline in `__get_request` and `__post_request`, duplicated, with `max_retries` passed as a parameter to each individual call, there was no single place to fix any of it.

---

## What Actually Transferred

Overall, I learnt a lot. Ever since then I've been writing tests for basically anything I developer, and leveraging async ruthlessly. Not to mention the absolute powerhouse that CI/CD is. 

Importantly, I learnt that: async code that passes tests is not the same as async code that's correct.

Unit tests, by default, test the happy path. They run in isolation, in a controlled order, with predictable timing. Real usage is none of those things. Real usage has users doing two things at once, network calls taking variable time, and Discord occasionally returning a plain text error where you expected JSON.

The tests aren't wrong, they're just not asking the right questions. You have to explicitly test for concurrency, for timing, for failure modes. The happy path is the easy part.

---

The wrapper is sitting on GitHub. Maybe I'll come back to it for real. Maybe I won't.

Either way: first async project, first unit tests, first library. All three taught me something I couldn't have learned by just reading about it.

[Check it out here](https://github.com/Myst1cS04p/Pollcord/) if you want to see the code.