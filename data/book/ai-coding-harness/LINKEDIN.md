# LinkedIn post — Coding Agent Harness

Your coding agent isn't the system. The harness around it is.

Most teams start the same way: prompt the agent, get a PR, have a human read every line. It works — until review toil scales linearly with agent throughput, and the agent repeats the same mistakes every session because nothing it learns survives the context window.

The fix isn't a better model. It's harness engineering: treating the agent as a component you *bought*, and building the trust around it yourself.

I turned this into an interactive system-design walkthrough — the kind of step-by-step design you'd do in an interview:

1️⃣ Baseline: raw agent, human review as the only sensor
2️⃣ Feedforward guides — rules files, docs, and scaffolds that steer *before* the agent acts
3️⃣ Feedback sensors — linters, type checks, and architecture rules wired into a self-correction loop
4️⃣ Review agents as inferential sensors (LLM-as-judge, used carefully)
5️⃣ Keeping quality left across the whole lifecycle
6️⃣ Drift detection and runtime feedback
7️⃣ A behaviour harness: specs and tests you can actually trust
8️⃣ Scaling it: harness templates and the human steering loop

My favorite detail: sensor output should be optimized for the LLM, not for humans. A lint message that says "move this behind the port interface in domain/ports" is a positive kind of prompt injection. A 400-line stack trace is context poison.

Feedforward-only harnesses drift into folklore. Feedback-only agents keep repeating themselves. You need both directions.

Inspired by Birgitta Böckeler's harness-engineering article: https://martinfowler.com/articles/harness-engineering.html

Explore the full interactive walkthrough — diagrams, trade-offs, and an explainer comic:
👉 https://zeljkoobrenovic.github.io/system-design-interviews/book/interview.html#ai-coding-harness

How much of your AI code review is still funneling through one human at the end?

#AIEngineering #SoftwareEngineering #SystemDesign #CodingAgents #DevTools
