---
layout: post
title:  "AI-First Development Without the Black Box"
date:   2025-09-03 12:30:00
image:  ai-wt-no-blkbx/cover.png
tags:   [GenAI, Software Engineering]
---

*Field notes for leaders on using AI to accelerate — not undermine — software engineering*

------

## The Temptation of “AI-First”

In my earlier essay, The Medium is the Engineer, I argued that the tools we use don’t just support engineering — they shape it. With AI, that’s truer than ever.

Across industries, leaders are making bold declarations: “We’ll be AI-first by next year.” The logic is seductive. If AI can generate 80–90% of the code, why not restructure teams around prompting instead of programming? For some, that might lead to why not reduce engineering headcount and lean on machines?

Here’s the danger: treat AI this way and you don’t get acceleration, you get vibe coding — prototypes masquerading as production. In startups, that might fly. In an enterprise, it’s a recipe for brittle, unmaintainable, and risky systems.

AI can be transformative. But without the right posture and guardrails, it quickly becomes a cash cab — a ride where the meter is always running and you end up paying more than you saved.

------

## The Experiment: Pairing with Cursor and Claude

To test how far AI-first can go, I used Cursor as my development environment, with Claude 4 Sonnet as a companion for harder tasks. I set up context and rules for cursor — TDD, immutability, DRY, run tests and security checks before every push — and handed the AI the keyboard as much as possible. My role was to guide and curate.

### What worked well:
* Small, scoped prompts produced clean, reviewable code.
* Templates for features and bug fixes kept the AI focused and aligned with expectations.
* Documentation hygiene was excellent — AI updated ADRs and READMEs more consistently than humans usually do.
* In areas where I already knew what “good” looked like, AI accelerated my typing and sometimes caught mistakes.


### Where the AI struggled:
* **Occasionally ditched rules.** Skipped tests, forgot TDD after a few interactions, or cut corners to push code faster. This made me think of the Mars Climate Orbiter, which NASA lost because one team used imperial units while another used metric. A tiny rules violation — one that “saved time” in the moment — destroyed a $125 million mission. AI cutting corners feels similar: it may look faster, but without enforced discipline, it can create silent failures with enormous downstream cost.
* **Introduced duplication.** Instead of reuse, making debugging harder over time.
* **Hard-coded examples.** At times it embedded literal values from the scenarios I provided — things that were meant only as test data or illustrations — directly into the codebase. That kind of leakage is dangerous in production, where a forgotten hard-coded constant can silently break scaling, security, or compliance.
* **Context drift.** As the project grew, so did confusion. The risk of black-box fragility increased.
* **Token costs piled up.** Without intervention, the experiment turned into that cash cab — speed at the cost of higher bills.


I still believe this is powerful too. Using AI to make engineers faster and more efficient is a tremendous opportunity - but only if it’s paired with proper training. We need to use AI responsibly and effectively, not just chase speed, but to safeguard long-term quality and maintainability. Speed matters - but speed without discipline is an illusion. Chasing velocity at the cost of quality and security looks like progress today, but it creates fragility tomorrow. 

------

## The Black Box Problem

This is where leaders must slow down and think.

Some parts of software can safely be treated as **black boxes**. We rely on compilers, operating systems, and stable libraries without inspecting every line of code. Why? Because they’re mature, relatively stable, evolve slowly — and, crucially, **we know who owns them**. If something fails, we know where to turn.

But your **deliverables** — the business logic, the workflows, the integrations — are different. They live at the shifting edge of customer needs and regulatory requirements. They evolve constantly. Making that code a black box is dangerous because:

* **Duplication compounds silently.** AI often generates new code instead of reusing existing modules. When one thing breaks, it breaks in ten places.
* **Debugging slows down.** Engineers can’t fix what they don’t understand. If AI wrote it and no one curated it, the team wastes time unraveling spaghetti instead of solving business problems.
* **Onboarding costs rise.** New engineers ramp up by reading and reasoning about code. If the code is opaque, they can’t form a mental model — they become dependent on the AI itself just to navigate.
* **Risk increases.** Unlike a library, which is tested across millions of users, your business logic is unique. If AI generated it and nobody reviewed it, who owns the failure when it leaks PII or violates compliance?


Ownership is the heart of the issue. A black box is safe when someone owns it and takes responsibility for its correctness. It’s unsafe when nobody owns it — when your team is essentially building with SOUP (Software of Unknown Provenance). That’s when you’ve crossed from engineering into mystery.

------

## The Core Analogy: AI as Pair - but Not an Equal

Twenty years ago, many managers thought pair programming was wasteful: two people working on one task instead of two. What they missed was the multiplier effect — two developers pairing often finished a feature faster and with higher quality than two working alone.

AI can play a similar role: it accelerates, unblocks, and provides a fast sounding board when no human peer is available. But unlike a human partner, AI is not an equal collaborator. It’s closer to a fast but undisciplined assistant - capable of fast computation, processing and producing results quickly, but only under strong direction and control.

That’s why constraints matter. Managers cannot just say “pair with AI” and assume discipline will follow. Leadership still needs to define goals, establish what’s acceptable and what’s not, and enforce architectural standards.

If AI is allowed to improvise unchecked, it will happily break boundaries - imagine letting front-end code talk directly to a production database. Guardrails, such as ArchUnit or CI/CD policies, are non-negotiable. They ensure that engineers remain firmly in charge, with AI serving as an accelerator rather than an autonomous decision maker

------

## Misconceptions Leaders Must Avoid

The biggest misconception I see from executives is this: “We can just fire developers and let product managers prompt AI.”

Wrong. That path leads to unreviewed code, black-box systems, and long-term fragility. AI can generate code, but it doesn’t understand context, business trade-offs, or edge cases. Without trained engineers in the loop, quality, security, and maintainability all collapse.

The right mindset: AI is not a replacement. It’s a force multiplier. Developers shift from being coders to **curators** — **reviewing**, guiding, and holding the system accountable.


------

## The Playbook for AI-First Success

If you’re a CTO, VP, or team lead moving toward AI-first, here are the principles to bake in from day one:

1. Guardrails are non-negotiable.
* Org-level rules enforced in CI/CD: tests, coverage, security scans, PR size limits.
* Project-level rules: duplication checks, ADR updates, design standards.
* Block AI-generated pushes if these rules aren’t met.
2. Measure by time-to-feature, not hype.
* The key metric: does AI reduce delivery time while maintaining quality, security, and clarity? Speed is valuable only when it doesn’t create downstream debt.
* Track the hidden cost: if code becomes opaque, every new engineer ramps slower. That’s debt, not acceleration.
3. Keep humans in the loop - with discipline.
* Engineers remain the medium — the custodians of context, judgment and accountability.
* AI is not an equal partner; it’s more like a fast but undisciplined assistant that must be directed and constrained.
* To keep humans truly in control, structure the workflow so the code volume stays manageable:
  * Use smaller iterations - break features into reviewable chunks.
  * Apply test-driven development (TDD) so AI’s output is immediately validated.
  * Enforce CI/CD guardrails (tests, security scans, architecture checks, etc.) so nothing bypasses discipline.
	
	In short: don’t just “have a human in the loop.” Design the process so the human stays in command - guiding AI, not following it.

4. Design for long consequences.
* Cutting corners for speed looks like progress today but creates fragility tomorrow.
* Leaders must resist the “ship fast at any cost” instinct and enforce standards that prevent black-box debt.


------

## Closing Thought

AI-first development can accelerate delivery, but only if leaders balance speed with discipline. AI is not a replacement and not an equal partner. It’s a powerful but undisciplined assistant that requires direction, guardrails, and human accountability.

Engineers are still the medium. Their role is shifting — from coders to curators, from builders of every brick to guardians of context and quality.

If you’re a decision-maker, your job is to set the guardrails, enforce the rules, and measure the right things. AI will amplify discipline — but it will also amplify chaos if you let it.

The choice, and the consequences, are yours.
