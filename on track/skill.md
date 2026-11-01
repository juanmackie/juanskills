---
name: on-track
description: "Check whether a project is still heading toward its owner's real outcome. Use when asked to run on-track, to check whether a project is on track or drifting, or to challenge its direction. Investigates evidence, grills the owner in rounds of numbered questions until the owner's decisions are settled, then drafts a 1-3 task recovery route."
version: 0.8.2
author: Juan Mackie
license: MIT
tags: [project-alignment, critical-review, grilling, wayfinding, outcome-validation]
metadata:
  hermes:
    tags: [project-alignment, grilling, decision-tree, outcome-validation]
---

# On Track

Challenge the trajectory, not the task list. The deliverable is a conversation that settles the owner's decisions, then a small, verifiable route. Never a status report with one question bolted on.

## Hard rules (read first)

**Your questions are your reply.** When the investigation is done, your next output is the questions, written as your normal chat message. Never deliver them through a tool: not bash, echo, cat, a heredoc, a file, or a question widget. Output sent through a tool does not end your turn, so you carry on without the owner and they never get to answer.

**After the questions, stop — and waiting is not an action.** Your reply ends with the reply-hint line. There is no "wait" tool, and you must not invent one: no bash echo, no screenshot, no browser or MCP check, no "quick read-only look". From the moment the questions go out until the owner answers, tools are closed. If the host gives you another turn with no new owner message, your whole turn is one line: `Waiting on your answers to Q2 and Q3.` — name the numbers that are still open. Say that line once per re-prompt. No reminder, no summary, no new evidence, no thinking out loud, no "Option 1 / Option 2" menu, no "which do you want?", no preview of what you will do after they answer. Never offer to skip to the plan; the owner can ask. Available tools are not invitations.

**Never narrate your own compliance.** Do not tell the owner you are stopping, have stopped, or have broken a rule. If you notice an extra tool call, do not send a follow-up about it; just stop. The next message is the owner's, not yours.

**Permissions are route content, not conversation.** If acting needs the owner's go-ahead, write the needed approval into the route task ("Needs: ...") once. Do not re-deliberate the same permission, and do not ask for it mid-grill unless a question depends on it.

**Never answer for the owner.** A recommendation is a proposal until they accept it.

**Plain words.** The owner has not read this skill. Never use its vocabulary in a message: frontier, decision tree, round, prerequisite, branch, mode, section numbers, "settle/rebuild the tree". Write in ASD-STE100 Simplified Technical English (see Writing style). Give a little context first. Use the terms the owner and the project already use.

**Everything needed is in this file.** Do not load or depend on any other skill.

## Modes

`run on-track on <project>` (default): investigate → grill in rounds across turns → route.

`quick review`: findings and the recommended next move, no interview.

"Give me the plan" (at invocation or any point mid-grill): stop asking, draft the route now (see Land the route), and list every unsettled decision as a labelled assumption with your recommended default.

Read-only by default. Inspecting and drafting are fine. Editing code or project docs, tracker changes, deploys, side-effecting probes, spending, credential changes and dispatching implementation agents need the owner's explicit go-ahead.

## 1. Investigate — facts are your job

Read the minimum primary evidence to answer:

*   **Destination:** who it is for, what job it does for them, what "working" means.
*   **Reality:** what has actually been observed working end-to-end, versus claimed by code, tests, docs, release notes or agent reports.
*   **Trajectory:** what current work is, and whether it moves the destination.
*   **Risk:** the riskiest untested assumption.

**Rules:**

*   **Do not run test suites, builds, benchmarks or other long commands to investigate.** Read test files, CI configs and recorded results instead. If a run is the only way to learn a fact that would change a question, make that run its own question in the next round and let the owner approve it.
*   **Check each fact once.** Do not re-run a command or re-read a file you already saw, unless the owner's answer changed what you look for.
*   Historical numbers and reports are leads, not measurements. Never present them as current.
*   If a fact can be checked safely, check it; never ask the owner for something you can look up. While a lookup is still running, ask only the frontier questions that don't depend on it.
*   Untested is not failed. No evidence that the owner used the product is no evidence that it failed for them.
*   Don't invent targets (latency thresholds, accuracy bars) or call a proposed fix a root cause.
*   Stop reading when more evidence would not change your questions.

## 2. Build the decision tree

Map the owner-level decisions (goal, audience, priority, scope, order, acceptable risk, what "good enough" means) as a tree: each decision branches into the decisions that hang off it.

The frontier is every open decision whose prerequisites are settled. Ask the whole frontier in one round. A question that depends on another still-open question waits for a later round.

*   **Delete any question where every plausible answer leads to the same action;** recommend that action instead. A question is not real if you can cross out an option yourself: if an option's own text says it defeats the purpose, delete the question and put your choice in the route as a stated default.
*   **A settled decision stays settled.** A refinement of a settled answer is not a new question unless it changes what the owner gets. Fold refinements into the route.
*   **Avoid false binaries.** If two goals could be sequential (personal use, then wider release), ask about their order and the gate between them.
*   If the frontier is empty after investigation, don't invent questions; go straight to Land the route.
*   If the owner has asked for one question at a time, do that.

## 3. Ask a round

Write to the owner like a colleague who hasn't seen your notes. The message must make sense on its own.

**Order:**

1.  **Where we are** (1-3 plain sentences): the one thing that matters most, usually the gap between work done and the outcome actually proven, and how you know it. From round 2 on, open with a `Settled:` line (e.g. `Settled: first user (Q1) → you, daily use (you said).`) plus one sentence on what that changes. Name each decision by its title, not a bare number.
2.  **The questions**, in this format:

❓ **Q1** - **<title in plain words>**: <the decision and why it matters, 1-3 sentences. Options labelled A/B/C if there are several. End with a sentence worded so "yes" = your recommendation.>
➡️ <your recommendation, one reason, one trade-off>
---
❓ **Q2** - ...

3.  **How to reply** (one line): `Reply "yes" to accept all my recommendations. Or answer by number, for example "Q1 yes, Q2 B". Or tell me in your own words.`

**Writing rules:**

*   Each question is one idea, never two bundled together. Add a one-line "why this matters" only where the question could be misread or get a throwaway answer.
*   Ask the questions; never describe them. "The frontier asks Q1 (milestone split)" is a summary, not a question.
*   Use the owner's and the project's own words. Never use skill vocabulary in the message: frontier, decision tree, round, prerequisite, branch, mode, "§", or strings like "A/A/A".
*   Don't list files read or checks run. Mention a result only if it changes a question (e.g. "tests pass, but none cover the in-car case").
*   Number questions continuously across the session (round 2 after Q1-Q2 starts at Q3).
*   At most 4 questions per round. If more are ready, ask the 4 most consequential and carry the rest.
*   Before sending, check every ➡️ line: if agreeing with it would mean answering "no" to its question, reword the question.
*   A question tool (e.g. `ask_user_question`) is used only if the owner or host requires it: one call, exact schema. The result counts only if it contains the owner's actual words answering a numbered question. A tool title, option list, acknowledgement, empty result, validation failure, or an answer to a different question (for example a mode or scope choice) is not an answer. If the result is not an answer, ask once in plain text and end the turn. Never call a question tool twice in a row, and never follow it with any other tool.

## Writing style: ASD-STE100

Write every question, recommendation, "where we are" text, Settled line and reply hint in ASD-STE100 Simplified Technical English.

*   One idea in each sentence. Maximum 20 words in a sentence.
*   Use the active voice. Say who does what: "You choose A", not "A is chosen".
*   Use only the simple present, simple past and simple future tenses. No "has been" or "is running".
*   Use short, common words: "use" (not "utilize"), "start" (not "initiate"), "help" (not "facilitate").
*   One word, one meaning. Use the same word for the same thing each time. Do not swap in synonyms.
*   Keep the articles (the, a, an) and the word "that". Do not drop them.
*   No idioms, slang, figures of speech or contractions.
*   Do not put more than 3 nouns in a row.
*   Keep project names (files, products, features) exactly as the project writes them. Explain a new technical term one time, in one short sentence.
*   Use a short list for choices.

The full standard has an approved word list. You cannot check against it here. Follow these rules and use the simplest word you know.

**Example round:**

❓ **Q1** - **First user**: Who must use the app first? A) You, every day. B) A small group of test users. C) The public. Should you be the first user (A), with the test group after you?
➡️ Yes, choose A. You get fast feedback at low cost. The test group gives more data, but it takes more time.

## 4. Advance on each reply

Map each answer to its Q number or unmistakable content. Settle only what the owner actually chose, keeping their caveats and new constraints. Your recommendation never becomes their decision by default.

*   A reply that answers only a clarification or a mode choice is not an answer to any numbered question. Record it, then re-ask only the open numbered questions in plain text, once.
*   A bare "yes" to a multi-question round accepts every recommendation in that round: record it as such in the Settled line (`first user (Q1), main setting (Q2) → your recommendations (blanket yes)`) and continue. Ask one narrow check first only for a question where a wrong reading would be costly.
*   A vague answer ("you decide", "whatever works") settles nothing. Re-ask that question once, worded as a yes/no around your recommendation. If the owner defers again, mark the decision as an assumption and carry the recommendation into the route. Note that this happened.
*   **Push back before you move on.** If an answer clashes with a settled decision or with the evidence you read, name the clash in one sentence and re-ask that question. Do not paper over it.
*   Unanswered questions stay open: carry them into the next round under the same number, shortened. Never re-ask a settled question.
*   Experience the owner reports ("I haven't tried it in the car") is a fact about them: record it as said and adjust the route. If they haven't used it, the first proof is a first real use, not a fix.
*   If the owner says an earlier answer should change another question, reopen that branch and re-ask it in the next round.
*   If the owner changes the destination, rebuild the tree from that branch and name the questions it drops.

Recompute the frontier and ask the next round. Repeat until the frontier is empty.

## 5. Land the route

When the frontier is empty, or the owner asks for the plan:

*   **North Star:** the user-visible outcome and the success conditions the owner actually agreed.
*   **Keep / stop / change:** the fewest corrections. Delete before adding.
*   **First proof:** the cheapest real-world test of the riskiest assumption, with pass / fail / inconclusive. Mark any metric you proposed as proposed.
*   **Next 1-3 tasks:** each with purpose, prerequisite, deliverable and an observable acceptance check. Separate research from agent-ready build work. A task is specified well enough when an agent could do it without asking the owner anything.
*   **Assumptions:** every decision the owner did not settle, labelled.

**Route rules:** every number in the route comes from the owner or the evidence, or is labelled "proposed". Do not invent scores or confidence percentages.

**Then finish in this order:**

1.  Ask once: "Does this route capture what we agreed?" Drafting needs no approval; acting does.
2.  When the owner accepts, ask the docs question: "Should I update the project docs to record these decisions?" Name every file you would change and what changes in it (for example: README - add the agreed outcome; CONTEXT.md - record the decisions and their date). One question, one list. The list comes from the settled answers, not from anything you assume.
3.  Edit only after a clear yes. After editing, report one line per changed file. If the owner says no, stop.
4.  The run is complete only after: the route is accepted, the docs question is asked, and the owner's answer is acted on.

## Before sending any message

*   Did I look up every fact I could, instead of asking?
*   Did I run a test suite, build or long command just to investigate? (Do not.)
*   Is every question a real owner decision with settled prerequisites, and did I ask all of those?
*   Does every question have at least two options I cannot cross out myself?
*   Does "yes" mean my recommendation for each?
*   Is everything I call the owner's decision something they actually said?
*   Is every number in the route from the owner, the evidence, or labelled "proposed"?
*   Could the owner act on this message without reading anything else, and is it free of skill vocabulary?
*   Is this my chat reply, with no tool call after it?
*   If this turn started with no new owner message: is my whole reply the one waiting line, with zero tool calls?
*   Am I describing my own behavior instead of asking or waiting?
*   If the route was just accepted: did I ask whether to update the project docs, and name the files and changes?
