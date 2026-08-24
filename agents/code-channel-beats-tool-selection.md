# Give an agent one sandbox, not twenty tools

## The thing

A general Python execution sandbox turned out to be worth more than a catalogue of specialist
tools — and not for the reason I expected. The win wasn't expressiveness. It was removing a
step the model was measurably bad at.

## Where I learned it

Helionix agent runtime, mid-2026. Local models (Gemma, Qwen) via Ollama, against a document
and memo task.

## The observation

Two measurements, which point in a direction the intuition doesn't:

- **Payload escaping was not the problem.** 4.9KB of code passed as a JSON tool argument
  parsed 10/10. The thing everyone worries about was fine.
- **Selection was the problem.** Choosing correctly among nine similar tools topped out at
  60–70%, even after tuning the descriptions. And passing a `file_id` between tools was 0% on
  Gemma — it simply would not carry an opaque identifier from one call to the next.

So the failure wasn't in expressing the action. It was in picking which action, and in
threading state between actions.

## The conclusion

If selection is the unreliable step, the fix isn't a better-described tool catalogue — it's an
interface with nothing to select. The model writes code in a fenced block, the runtime extracts
and executes it, stdout and any produced files come back as the observation.

That doesn't improve the odds on the hard step. **It removes the step.**

The `file_id` problem dissolved the same way: pre-mount any document already referenced in the
conversation into the sandbox's working directory, and the model never has to carry an
identifier at all — it just sees a file.

## Why it generalises

Three bespoke capabilities had been carded — read PDFs, edit Office files, analyse
spreadsheets. Each was one path for one format. Four lines of pandas replaced the whole class,
and it extends to the next format without anyone building anything.

The pattern: **when you find yourself adding a capability per file type, you probably want one
general capability instead.**

There's a cost measurement hiding in there too. The spreadsheet path had been dumping 200 CSV
rows into the conversation for the model to reason over — expensive in tokens, and precisely
the arithmetic that models get wrong. Computing beats recalling.

## What makes it safe

Running model-written code is only acceptable because of one structural property:

**The runtime does the I/O; the sandbox does the computation.**

The sandbox has **no network** — it can't reach the database, the APIs, the internet, or the
runtime itself. Which means **no credential ever enters it**. Files are fetched outside, written
into its working directory, and results read back out. The model never handles a token.

Get that boundary right and the rest is ordinary container hygiene. Get it wrong and you've
given a language model your credentials.

## Gotchas

- **Failures must come back as results, not exceptions.** A traceback returned as readable text
  is something the model corrects on the next turn. That iteration loop is where much of the
  value lives, and collapsing it into "the tool failed" throws it away.
- **Output needs to be more than stdout.** A capability that can only print can't produce a
  chart or a document — which is half the point. Return files written, too.
- **The fence must close before execution.** With streaming, it's easy to start running a
  partial block.
- **Intent vs illustration.** A model writing example code in prose looks identical to a model
  asking for execution. You need a marker that distinguishes them, or you'll run things nobody
  asked you to run.
- **Getting arbitrary data in is where the design pressure lands.** Once it works for files,
  you want it for a fetched page, a query result, a previous run's output. The obvious fixes
  are a parameter per source (which accretes until the function has eleven arguments) or a
  persistent scratch space (which reintroduces state between calls). Both are real decisions;
  neither should be made by accident while adding a convenience.
- **This was measured on local models.** Cloud models with strong native tool calling may not
  show the same selection ceiling. The lesson is *measure your interface before blaming the
  model* — the first instinct was that the models weren't capable enough, and that was wrong.
