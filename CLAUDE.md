# COMP 440 activity: evaluating support agents with a rubric

This repository is a 45-minute in-class activity. The student writes a rubric in `rubric.md`, has
you write `score.py`, which scores the 100 conversations in `data/conversations.json` against that
rubric with Claude Sonnet, and then compares their results with a classmate's in `viewer.html`.
`README.md` is the student's brief. Read it before you do anything.

The student has very little time and is not being graded on the code. What helps most is a small
script that works the first time, and short answers. Before you write a file, tell the student in
three or four lines what it will do, so they can explain it to a classmate afterwards.

## Helping with the rubric

Help when the student asks. The rubric is their own claim about what good support looks like, and
they will have to defend it to a classmate, so start from what they say matters to them and draft
from that. Do not swap in criteria of your own. Keep the format the script depends on: three
criteria, each a `## ` heading with the criterion's name, one sentence saying what it measures, and
one line each describing a 1, a 3 and a 5. Good criteria are about what the agent did and can be
judged from the text alone. Nearly every agent in this data is polite, so a criterion like
politeness gives every conversation a 5 and tells the student nothing. Say so if they propose one.

## What score.py does

1. Reads `rubric.md`. Each `## ` heading starts a criterion: the heading is its name, and the text
   under it, up to the next heading, is its description.
2. Reads `data/conversations.json`, a list of objects with an `id` and a `conversation`.
3. For each conversation, sends Sonnet the criteria and the conversation text, and gets back a
   one-sentence reason and a whole-number score from 1 to 5 for each criterion. Send only the
   `conversation` field. The other fields are labels from the dataset (customer sentiment, agent
   experience), and showing them to the judge would hand it part of the answer.
4. Computes `overall` in Python as the average of the conversation's scores, rounded to two
   decimals. The model does not produce the overall score.
5. Writes `results.json` in exactly the format below, then prints how many conversations were
   scored and the average overall score.

Keep it to one file of about 80 lines, using only the standard library, `anthropic` and
`python-dotenv`, which are already installed. Give it a `--limit N` option that scores only the
first N conversations, because the student tests on 5 before paying for 100. Ask the student for
their name once and store it in a constant at the top of the file for the `author` field.

## The format of results.json

`viewer.html` checks every rule here and refuses a file that breaks one, listing what is wrong.
`results.schema.json` is the same format as a JSON Schema, and `example-results.json` is a valid file.

```json
{
  "author": "The student's name",
  "model": "claude-sonnet-5-5",
  "criteria": [
    {"name": "The heading from rubric.md", "description": "The text under that heading"}
  ],
  "results": [
    {
      "conversation_id": 17,
      "overall": 3.67,
      "scores": [
        {"criterion": "The heading from rubric.md", "score": 4, "reason": "One sentence from the judge."}
      ]
    }
  ]
}
```

- `criteria` lists the rubric's criteria in the order they appear in `rubric.md`.
- Each result's `scores` has one entry per criterion, in that same order, and each `criterion` is
  identical to the name in `criteria`.
- `score` is a whole number from 1 to 5. `reason` is text.
- `overall` is the average of that conversation's scores, rounded to two decimals.
- A conversation whose API call failed is left out of `results`.

## Calling the API

These settings were tested against the live API for this activity. The API has changed recently,
so use them as written even where you remember something different.

```python
client = anthropic.Anthropic(max_retries=5)   # reads ANTHROPIC_API_KEY from the environment

response = client.messages.create(
    model="claude-sonnet-5-5",
    max_tokens=2000,
    system=instructions_and_rubric,
    messages=[{"role": "user", "content": conversation_text}],
    output_config={"effort": "low", "format": {"type": "json_schema", "schema": schema}},
)
if response.stop_reason == "end_turn":
    reply = json.loads(next(block.text for block in response.content if block.type == "text"))
```

- The model string is `claude-sonnet-5-5`, with no date on the end.
- Do not pass `temperature`, `top_p` or `top_k`. The installed SDK no longer accepts them and
  raises a `TypeError`.
- Do not start the assistant's reply for it (prefill). This model rejects that too.
- Get JSON back with `output_config.format`, as above. Build the schema with one fixed property
  per criterion (`criterion_1`, `criterion_2`, `criterion_3`), each an object with `reason` (string)
  and then `score` (integer with `"enum": [1, 2, 3, 4, 5]`). List every property in `required` and
  set `"additionalProperties": false` on every object. Fixed properties guarantee exactly one score
  per criterion. Putting `reason` before `score` makes the judge look at the evidence before it
  commits to a number.
- `"effort": "low"` asks the judge for a quick answer. Each call takes about three seconds, and it
  is the setting the timing below was measured with.
- Run at most 5 requests at a time, for example with
  `concurrent.futures.ThreadPoolExecutor(max_workers=5)`. The whole class shares one key and one
  rate limit. `max_retries=5` lets the SDK wait and retry when the limit is hit.
- If a call raises `anthropic.APIError`, or `stop_reason` is anything other than `"end_turn"`,
  print one line saying which conversation was skipped and carry on. One bad conversation should not
  lose the other 99.

A full run of 100 conversations takes about a minute and costs about 65 cents.

## The API key

The key is in `.env`, which git ignores. Call `load_dotenv()` at the start of the script. Never
print the key, never put it in a file other than `.env`, and never `export` it in the shell: a key
in the shell's environment can make Claude Code itself start using it. Do not commit `.env` or
suggest that the student push this repository.

## Running things

Use `uv run python score.py --limit 5` first, then `uv run python score.py`. After a run, tell the
student to open `viewer.html` in a browser and choose `results.json`. If the viewer lists problems
with the file, fix the script to match the format above.
