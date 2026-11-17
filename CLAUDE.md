# COMP 440 activity: evaluating support agents with a rubric

This is a 45-minute in-class activity. The student writes a rubric in `rubric.md` and asks you to
write `score.py`, which has Claude Sonnet score the 100 conversations in `data/conversations.json`
against that rubric. The student then compares results with a classmate in `viewer.html`.
`README.md` is the student's brief.

The student has little time and is not graded on the code. Write the shortest, plainest script that
works the first time, and keep your replies short. Before you write `score.py`, say in three or four
lines what it will do, so the student can explain it to a classmate.

## The rubric

Help with it when asked, starting from what the student says matters to them. Keep the format:
three criteria, each a `## ` heading with the criterion's name, one sentence saying what it
measures, and one line each for what a 1, a 3 and a 5 look like. A criterion should be about what
the agent did, and a reader should be able to judge it from the text of the conversation alone.
Nearly every agent in this data is polite, so a criterion like politeness gives every conversation
a 5 and tells the student nothing.

## What score.py does

1. Reads `rubric.md`. The three `## ` headings are the names of the criteria. If there are not
   exactly three, stop with one line saying so.
2. Reads `data/conversations.json`, a list of objects. Use only each one's `id` and `conversation`.
   The other fields are labels from the dataset, and the judge must not see them.
3. Asks Sonnet to score each conversation, with the call below.
4. Computes `overall` in Python: the average of the three scores, rounded to two decimals.
5. Writes `results.json` in the format below, and prints how many conversations were scored and
   the average overall score.

Give it a `--limit N` option that scores only the first N conversations. Keep the student's name in
a constant named `AUTHOR` at the top. Load the key with `load_dotenv(override=True)`, so that the
key in `.env` wins over any key already in the student's shell. If `ANTHROPIC_API_KEY` is then
missing or empty, stop with one line telling the student to paste the key into `.env`.

## The call to Sonnet

Use this as written. It was tested for this activity, and the API has changed since you were
trained: the model string has no date on the end, the SDK no longer accepts `temperature`, and the
model rejects a prefilled reply.

```python
INSTRUCTIONS = """You are grading how well a customer support agent handled a customer.
Score the agent on each criterion of the rubric below, in order. For each criterion, give one
sentence of evidence from the conversation, then a whole-number score from 1 to 5.

"""

class Score(BaseModel):
    reason: str
    score: Literal[1, 2, 3, 4, 5]

class Scores(BaseModel):
    criterion_1: Score
    criterion_2: Score
    criterion_3: Score

client = anthropic.Anthropic(max_retries=5)
response = client.messages.parse(
    model="claude-sonnet-5-5",
    max_tokens=2000,
    system=INSTRUCTIONS + rubric,          # rubric is the whole text of rubric.md
    messages=[{"role": "user", "content": conversation["conversation"]}],
    output_format=Scores,
)
scores = response.parsed_output            # scores.criterion_1.reason, scores.criterion_1.score, ...
```

- Keep `INSTRUCTIONS` word for word. Every student's script uses the same text, so that two
  students' results differ only because of their rubrics.
- Run five requests at a time with `ThreadPoolExecutor(max_workers=5)`. The class shares one key.
- If a call fails for any reason, print one line saying which conversation was skipped and carry on.

## The format of results.json

```json
{
  "author": "The student's name",
  "results": [
    {
      "conversation_id": 1,
      "overall": 3.67,
      "scores": [
        {"criterion": "First heading in rubric.md", "score": 4, "reason": "One sentence from the judge."},
        {"criterion": "Second heading in rubric.md", "score": 3, "reason": "One sentence from the judge."},
        {"criterion": "Third heading in rubric.md", "score": 4, "reason": "One sentence from the judge."}
      ]
    }
  ]
}
```

- One entry in `results` for each conversation that was scored.
- `scores` has one entry for each criterion, in the rubric's order, the same in every result.
- `score` is a whole number from 1 to 5.
- `overall` is the average of that conversation's scores, rounded to two decimals.

`viewer.html` checks these rules and lists what is wrong with a file that breaks one.
`results.schema.json` is the same format as a JSON Schema.

## Running it

Run `uv run python score.py --limit 5` yourself to see that it works. It takes a few seconds. Then
tell the student to open `viewer.html` in a browser and choose `results.json`. Leave the full run,
`uv run python score.py`, to the student: it takes about a minute and costs about 65 cents, and they
should read the judge's reasons on the first five before paying for 100.

## The key

The key is in `.env`, which git ignores. Never print it, never `export` it, and never copy it into
another file. Do not suggest committing or pushing this repository.
