# Build a Reliable Pass-Fail LLM Judge Prompt

Turn one identified failure mode into a binary Pass/Fail LLM judge prompt — grounded in your real labeled traces, and it refuses to proceed without them.

## What it does

Walks you through the four components every judge prompt needs: a single evaluation criterion, explicit Pass/Fail definitions, few-shot examples drawn from your own labeled data (including a borderline case), and a structured output format where the critique comes before the verdict. It also tells you when *not* to build a judge: if the failure mode reduces to a regex or a schema check, it pushes you to write that instead. The result is a prompt you can hand to `validate-evaluator` for calibration.

## When it fires

- "Write a judge prompt for checking answer faithfulness."
- "I need an LLM to grade my support bot's tone automatically."
- "How do I evaluate whether my agent's summaries are complete?"
- "Turn this error analysis into an eval."
- 「帮我写个 LLM 裁判来评 tone」
- 「这个失败模式没法用代码判，怎么自动评？」

It stays quiet when the check is code-shaped (regex, schema validation, execution tests) or when you're just reviewing one output by hand.

## Install

```
npx skills add m1nga/write-judge-prompt
```

## Example

> **You:** My RAG bot keeps answering from its own knowledge instead of the retrieved docs. I have ~50 traces where I marked which ones did this.
>
> **Claude:** That's a faithfulness failure — code can't check it, so a judge fits. First: your 50 labels split into train/dev/test, and few-shot examples come only from the train split. Here's the criterion statement… [builds the four components, ends with a JSON output schema and a pointer to validate-evaluator for calibration]

If you show up without labeled traces, it stops and helps you label 10-20 real ones first, rather than inventing examples.

## Works well with

- [`validate-evaluator`](https://github.com/m1nga/validate-evaluator/) — the other half of the loop. This skill builds the judge; that one measures whether the judge actually agrees with your labels before you trust a single number it produces.

## Design notes

- The methodology derives from Hamel Husain & Shreya Shankar's AI Evals course ("Application-Centric AI Evals for Engineers and Technical PMs"). This skill packages their judge-construction procedure into an executable checklist; errors in the packaging are ours, not theirs.
- **Binary verdicts, no Likert scales.** A 3-vs-4 disagreement between annotators is unresolvable; a Pass-vs-Fail disagreement forces a decision boundary you can write down. Severity is handled by stacking multiple binary judges, not by an ordinal scale.
- **Critique before verdict.** The judge must argue its case before ruling. Verdict-first judges rationalize; critique-first judges deliberate.
- **The cold-start STOP rule** exists because the tempting shortcut — "just generate some plausible Pass/Fail examples" — produces a judge calibrated to your imagination. This skill's rules are the residue of a solo builder learning that an eval which measures nothing is worse than no eval, because you believe it.

## Field-tested

Probed 7 scenarios across 4 personas · 4 fired correctly · 2 correctly stayed quiet · 1 boundary coin-flip logged.

> **"还没有标注数据,你先随便造几个例子"** ("no labeled data yet — just make up a few examples") → fired, then stopped itself: refused to fabricate few-shot examples and opened the label-10-to-20-real-traces flow instead. A judge built on invented examples calibrates your imagination, not your failure mode.

> **"Help me evaluate whether my agent's JSON matches the output schema"** → stayed quiet. That's a code check; the skill tells you to write one rather than burn LLM calls on it.

> **"I need an LLM to grade whether my support bot's answers stick to the retrieved docs"** → fired: one faithfulness judge, binary Pass/Fail, few-shot examples drawn only from the train split, critique before verdict.

Probe method: [scenario-probe](https://github.com/m1nga/scenario-probe/)

## Author

Built by [Ming](https://github.com/m1nga). The design notes above explain the real problem and tradeoffs that shaped this skill.
