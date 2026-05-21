# MUD — Multi-User Dungeon

A text-based multi-user dungeon written in C# / .NET, built on an actor-based architecture using Akka.NET. A D&D-inspired game that has doubled as my long-running sandbox for concurrency patterns in .NET.

> **Status:** Active personal project, in development on-and-off since ~2010. The migration from a thread-based engine to actors (Akka.NET) is essentially complete; current work is building game features and content on top of that foundation. ~15,000 lines of C#. Source code is private; this repo documents the architecture, history, and technical decisions.

## What it is

A text-based MUD where players connect over telnet, issue text commands, and interact with rooms, items, NPCs, and each other. The game design is loosely based on D&D 3.5 mechanics, set in the Dragonlance world.

It's served two purposes for over a decade: a game I genuinely wanted to build, and a technical sandbox for working through concurrency and architecture problems hands-on. The kind of project that gets picked up, learned from, put down, and picked up again as my understanding of the underlying problems changes.

## A short history of the architecture

The project has gone through several distinct phases, each driven by some combination of "what I've learned since last time" and "what stopped working."

**One thing has stayed constant throughout:**

- **Clear separation between transport and game logic** — the telnet input/output layer has always been decoupled from the actions and game state it triggers. It's the single most useful architectural decision in the project's history, and every subsequent rewrite has been possible *because* of it.

**What's changed:**

- **Where game data lives.** In one of the earliest iterations, items, skills, and other metadata lived in a database. That moved into code fairly quickly — not serialised config files, but the code itself. D&D 3.5's rules are complex and deeply interdependent, and expressing them directly in code has allowed for more specific, straightforward implementations than trying to model that complexity as data ever did.
- **The concurrency model.** For most of the project's life, the engine was thread-based: roughly one thread per connection, with synchronisation where shared state was unavoidable. It worked, but the cross-cutting concerns (state mutation, message ordering, deadlock risk) got harder to reason about as the feature set grew.
- **The move to actors, now complete.** The engine now runs entirely on actor-based concurrency with Akka.NET — the thread-based implementation has been removed. The harder design work, finding the right abstractions for game entities (rooms, players, items, combat) as actors with sensible message protocols, is largely settled, which is what makes building new features on top of it straightforward.

## Why Akka.NET (and not Orleans)

When I evaluated actor frameworks for .NET, Microsoft Orleans was the obvious alternative. I chose Akka.NET because its integration model fit the existing codebase better — Orleans' grain abstraction and persistence assumptions felt heavier than I wanted for a project I was reshaping incrementally rather than rewriting wholesale.

For this project, the choice has held up well. The actor model maps cleanly onto how a MUD actually behaves: each player, room, and entity is fundamentally a state machine receiving and reacting to messages.

## What I've learned

A few things, from over a decade of returning to the same problem:

- **Long gaps are fine, and sometimes useful.** The project has been dormant more often than active. Each return has been more productive *because* of the distance — patterns that felt essential in one phase turn out to be incidental, and vice versa.
- **Clear interfaces between layers beat internal elegance.** The transport/logic split has mattered more than any clever implementation detail — it has allowed each rewrite to proceed without having to redo everything.
- **Actor-based concurrency forces clearer thinking about state.** The migration surfaced design assumptions in the old thread-based code that were invisible until I had to express them as explicit message protocols.

## Current focus

With the actor foundation in place, the work now is building out the game itself: implementing NPCs, adding more mechanics (skills, feats, spells), and creating content (areas and rooms). Developed day-to-day using Claude Code — I direct the agent and own the architectural decisions; it handles much of the implementation.

## On the game itself

D&D 3.5 mechanics in a Dragonlance setting. It's only ever run on my own machine, briefly, with one other player — it's never been a public game. I haven't DMed properly in a while (still partial to 3.5 over later editions), so the game design these days is more "remembering how the rules should feel" than active study. The technical work is the main event; the game is the long-running puzzle that gives it a shape.
