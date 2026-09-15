---
title: "Misfortune Game"
date: 2025-06-19
summary: "A relative-order card game implemented in React and Express/SQLite. Features real-time round timers, anti-cheat hidden card indexes, secure scrypt authentication, and match history tracking"
tags: ["JavaScript", "Web Development"]
---

<div class="flex flex-wrap gap-2 mb-6">
  {{< button href="https://github.com/giulio-arecco/misfortune-game#client-side-routing-and-navigation-guards" target="_blank" >}}
    {{< icon "github" >}} View Source Code
  {{< /button >}}
</div>

This project implements a full-stack web application centered around an order-estimation card game. Players must maintain a hand of cards sorted by a concealed numerical "misfortune index" evaluating the severity of various scenarios. Engineered as a decoupled client-server architecture, the project integrates a React 19 Single-Page Application with a stateless Node.js Express REST API, backed by a relational SQLite database for historical record-keeping.

**Tech Stack:**

* **Frontend:** React 19, React Router v7, Vite, Bootstrap.
* **Backend:** Node.js, Express, Passport.js, express-validator.
* **Database:** SQLite, custom asynchronous DAOs.

## Architectural Overview

* **React 19 Client:** a Single-Page Application bundled via Vite, utilizing React Router for navigation guards and the `useActionState` hook for asynchronous form submission pipelines.
* **Express REST API:** a Node.js backend exposing game management, round handling, and user session endpoints, secured via Passport.js local authentication and `express-session` cookies.
* **Relational Storage:** a strict SQLite schema enforcing foreign key constraints, managed through asynchronous Data Access Objects executing parameterized queries.
* **Request Validation Pipeline:** an `express-validator` middleware chain that enforces type coercion, boundary checks, and input sanitization across all API payloads.
* **Cryptographic Security:** password hashing and validation utilizing Node.js native `crypto.scrypt` and `crypto.timingSafeEqual` primitives.

## Engineering Highlights

### Concurrency Control and Race Condition Elimination

The core gameplay loop enforces a strict 30-second countdown timer per round. A critical race condition arises if a player submits their card placement at the exact moment the timer expires, potentially triggering both a submission outcome and a timeout penalty simultaneously. To eliminate this, the frontend interval relies on functional state updates. When the timer reaches zero, the update handler inspects the previous state's round result. If the form submission pipeline has already resolved the round (assigning a `'win'` or `'loss'`), the timer aborts without incrementing the error counter, ensuring deterministic game state progression.

### Asynchronous Form Pipeline and Boundary Evaluation

Card insertion logic is managed without manual pending-state tracking by utilizing React `useActionState` hook. The user interface renders an interleaved array of potential insertion slots. Each radio input slot dynamically computes and serializes its boundary parameters (`prevMisfortune` and `nextMisfortune`) into a JSON string. Upon submission, the action state parses these boundaries and evaluates whether the target card's true misfortune value falls correctly between the adjacent constraints.

### Anti-Cheat and Information Hiding

To prevent clients from inspecting network traffic to cheat the sorting mechanic, the architecture implements an information-hiding protocol. When the client draws a new card, the API endpoint is queried with a `getMisfortune=false` parameter. The backend DAO actively strips the numerical misfortune value, returning `null` to the client. The true numerical value remains isolated on the server and is only retrieved for validation after the player definitively locks in their insertion choice via the frontend form.

### Data Deduplication and Relational Mapping

Generating random cards requires ensuring no duplicates are drawn within a single game session. This is handled directly in the SQL persistence layer via a nested subquery that filters the `Card` table against all `cardId` records already associated with the current `gameId` in the `RoundCard` junction table. Furthermore, querying a user's match history requires reconstructing deeply nested hierarchical data (Games containing Rounds containing Cards) from flat SQL `JOIN` rows. The DAO implements a two-pass mapping algorithm utilizing a `Map` data structure to progressively reconstitute the nested arrays while preserving histories.
