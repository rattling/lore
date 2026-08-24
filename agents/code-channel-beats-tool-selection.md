# Give an agent one sandbox, not twenty tools

## The thing

A general Python sandbox beat a catalogue of specialist tools, but not for the reason expected.
The win wasn't expressiveness. It was removing a step the model was measurably bad at.

## Where I learned it

Helionix agent runtime, mid-2026. Local models (Gemma, Qwen) via Ollama, on a document and memo
task.

## The measurements

- **Escaping was fine.** 4.9KB of code passed as a JSON tool argument parsed 10/10.
- **Selection was not.** Choosing among nine similar tools topped out at 60–70% after tuning
  the descriptions. Passing a `file_id` between tools was 0% on Gemma.

The failure wasn't expressing the action. It was picking one, and threading state between them.

## The conclusion

A better-described catalogue won't fix a selection problem. An interface with nothing to select
will. The model writes code in a fenced block, the runtime extracts and runs it, stdout and any
files produced come back as the observation.

`file_id` went the same way: pre-mount any document already referenced in the conversation, and
the model sees a file instead of carrying an identifier.

## Why it generalises

Three capabilities had been planned: read PDFs, edit Office files, analyse spreadsheets. One
path each. Four lines of pandas replaced all three and extends to the next format for free.

If you're adding a capability per file type, you want one general capability instead.

The spreadsheet path had been dumping 200 CSV rows into the conversation for the model to reason
over. Expensive, and exactly the arithmetic models get wrong. Computing beats recalling.

## What makes it safe

The runtime does the I/O. The sandbox computes.

No network in the sandbox, so it can't reach the database, the APIs, the internet or the
runtime, so no credential ever enters it. Files are fetched outside, written into its working
directory, results read back out. The model never handles a token.

Get that boundary right and the rest is container hygiene. Get it wrong and you've handed a
language model your credentials.

## Gotchas

- **Failures come back as results, not exceptions.** A traceback the model can read is one it
  corrects next turn. Collapsing it into "the tool failed" removes the iteration loop, which is
  where much of the value is.
- **Return files written, not just stdout.** A capability that can only print can't produce a
  chart or a document.
- **The fence must close before execution.** Easy to start running a partial block when
  streaming.
- **Intent vs illustration.** Example code in prose looks identical to a request to execute.
  You need a marker, or you'll run things nobody asked for.
- **Getting arbitrary data in is where the pressure lands.** Next you want a fetched page, a
  query result, a previous run's output. The options are a parameter per source, which accretes
  until the signature has eleven arguments, or a persistent scratch space, which brings back
  state between calls. Both are real decisions. Neither should happen by accident.
- **Measured on local models.** Cloud models with strong native tool calling may not hit the
  same ceiling. The transferable part: measure the interface before blaming the model. The first
  instinct here was that the models weren't good enough.
