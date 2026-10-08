---
name: llm-council
description: Pressure-test a real decision by running it past five advisors with different thinking styles, an anonymous peer review, and a chairman's verdict. Use when the user says "council this", "pressure-test this", "stress-test this", or "debate this", or brings a genuine decision with stakes and several options (architecture choice, tool or vendor pick, scope cut, launch call). Do not use for factual lookups, simple yes/no questions, or creation tasks like "write X".
---

# LLM Council

Based on Karpathy's LLM Council (github.com/karpathy/llm-council): independent answers, anonymous peer review, then a chairman's synthesis. Here the advisors are subagents with different thinking styles rather than different models, so treat agreement as a weaker signal than it would be across models.

It costs 11 subagent runs. Only convene it when being wrong is expensive; otherwise just answer.

## 1. Frame the question

- Read the 2-3 most relevant context files (CLAUDE.md, files the user referenced, code the decision touches). Keep it quick.
- If the question is too vague to answer, ask one clarifying question, then proceed.
- Write a neutral framing: the decision, the options, the constraints, and what's at stake. Don't steer.

## 2. Advisors (5 subagents, all launched in one message)

| Advisor | Lens |
|---|---|
| Contrarian | Assume there is a fatal flaw and find it. |
| First Principles | What problem are we actually solving? Is this the right question? |
| Expansionist | What's the upside everyone is missing? What if it works better than expected? |
| Outsider | No context beyond what's written. Flag anything unclear or assumed. |
| Executor | Can this be done, and what's the first concrete step on Monday? |

Prompt each: "You are the {Advisor} on a council. Your lens: {lens}. Question: {framing}. Argue your lens fully, don't balance it; others cover the other angles. 150-300 words, no preamble."

## 3. Peer review (5 subagents, all launched in one message)

Shuffle the five answers and label them A-E so reviewers can't tell who wrote which. Each reviewer gets the framing plus all five and answers, in under 200 words:
1. Which response is strongest, and why?
2. Which has the biggest blind spot, and what is it?
3. What did all five miss?

## 4. Chairman (1 subagent)

Give it the framing, the five answers with advisor names, and the five reviews. It may side with a minority if that reasoning is strongest. Output:

```
## Council Verdict: {topic}
### Where the council agrees
### Where it clashes
### Blind spots caught in review
### Recommendation        (a real answer, not "it depends")
### First step            (one concrete action)
```

## 5. Deliver

Present the verdict in chat. Save a transcript only if the user asks.
