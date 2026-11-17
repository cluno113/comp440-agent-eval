# COMP 440 Activity: Evaluating Support Agents with a Rubric

In this activity you will write a rubric for good customer support, have Claude Sonnet score 100 support conversations against it, and compare your results with a classmate's. A model that scores other text this way is often called a "judge".

You have 45 minutes, and Claude writes all the code. Work on your own laptop for Parts 0 to 2, and pair up with a neighbor for Part 3. Ask questions in class or in `#comp440-f26` on Slack.

## Part 0: Setup (5 minutes)

1. Get the repository and install its two packages:

   ```
   git clone https://github.com/shilad/comp440-agent-eval
   cd comp440-agent-eval
   uv sync
   ```

2. Run `cp .env.example .env`, open `.env`, and paste the key I give you in class after `ANTHROPIC_API_KEY=`. Please don't `export` the key in your terminal instead, because Claude Code may then start using it for its own work.

3. Open `data/examples.md` and read at least two of the conversations. Each one is between a customer and a support agent at a made-up store called BrownBox.

Which of the two agents did a better job? What did that agent do that the other one didn't?

## Part 1: Write your rubric (8 minutes)

Your rubric answers one question: how well did the support agent handle this customer?

1. Pick **three** criteria. Your answer to the question above is a good place to start. Each criterion should be about what the agent did, and you should be able to judge it from the text alone.

2. Replace what is in `rubric.md` with your three criteria, keeping its format: a `## ` heading with the criterion's name, one sentence saying what it measures, and one line each for what a 1, a 3 and a 5 look like.

You are welcome to have Claude help. Start Claude Code in this folder (`claude`), tell it what you think separates a good conversation from a bad one, and ask it to draft the rubric. Then edit it until you agree with every line. You'll have to defend it to a classmate in Part 3.

A criterion that every conversation passes tells you nothing. I left one in `rubric.md` to show the format: Politeness. When I scored all 100 conversations on it, every one got a 5.

## Part 2: Score the conversations with Sonnet (15 minutes)

1. Ask Claude to write the script. This prompt is enough, because `CLAUDE.md` in this repository tells Claude the rest:

   ```
   Write score.py for this activity. My name is YOUR NAME.
   ```

   For each conversation, the script sends your rubric and the conversation to Sonnet, gets back a score from 1 to 5 and a one-sentence reason for each criterion, averages the three scores into an overall score, and saves everything in `results.json`.

2. Try it on 5 conversations. This takes a few seconds.

   ```
   uv run python score.py --limit 5
   ```

3. Open `viewer.html` in your browser (double-click it) and choose your `results.json`. You should see a list with one row for each conversation. Click a row to read the conversation next to Sonnet's scores and reasons. If the page lists problems with your file instead, paste the list to Claude and ask it to fix the script.

   Did Sonnet read each criterion the way you meant it? If not, fix the wording in `rubric.md`, not the code, and run the 5 again. Sonnet's score for a conversation can move by a point from one run to the next even when nothing changed, so look at the reasons more than the numbers.

4. Score all 100. This takes about a minute and costs about 65 cents from the class key, so please don't run it more than you need to. Every run replaces `results.json`, so don't go back to `--limit 5` after this one.

   ```
   uv run python score.py
   ```

Reload `viewer.html` and choose the new `results.json`. Which criterion has the lowest average? Click the "Overall" heading to put the lowest scores first, and open your lowest-scoring conversation. Do you agree with Sonnet about it?

## Part 3: Compare with a classmate (12 minutes)

Your neighbor scored the same 100 conversations with a different rubric. Your criteria differ, so the overall score is what you compare.

1. Send your `results.json` to your neighbor in a Slack direct message, and get theirs. Save theirs under another name, for example `results-sam.json`, so that it doesn't replace yours.

2. In `viewer.html`, choose your file as Results A and your neighbor's as Results B. The list now shows both sets of scores and the gap between the two overall scores, with the largest gaps first.

3. Open the three conversations at the top of the list and read them together.

For each one: do your rubrics measure different things, or did Sonnet read one criterion in a way its author didn't intend? If BrownBox had to pick one of your two rubrics to decide which agents get more training, which should it pick, and why?

## Part 4: Submit (5 minutes)

Before you leave class, open the [course form](https://forms.gle/mgKcnqzTGxNaGvteA), choose "Activity submission", and paste these into the text box:

1. The words "Agent evaluation activity" and your neighbor's name.
2. Your `rubric.md`.
3. Your average overall score and the average gap between you and your neighbor. Both are in the "Average" row of the list.
4. Two or three sentences on why your judges disagreed about the conversation with the largest gap.

You don't need to push your code anywhere.

## If you finish early

Rename your `results.json` to `results-first.json`, run `score.py` again, and load both files in the viewer. This compares your judge with itself. When I did this with one of my rubrics, the two runs gave the same overall score on 65 of the 100 conversations, with an average gap of 0.12. What does that say about a gap of half a point between two different judges?

## The format of results.json

Everybody's script writes the same format, which is what lets the viewer compare any two files.

```json
{
  "author": "Your name",
  "results": [
    {
      "conversation_id": 1,
      "overall": 3.67,
      "scores": [
        {"criterion": "Your first criterion", "score": 4, "reason": "One sentence from Sonnet."},
        {"criterion": "Your second criterion", "score": 3, "reason": "One sentence from Sonnet."},
        {"criterion": "Your third criterion", "score": 4, "reason": "One sentence from Sonnet."}
      ]
    }
  ]
}
```

- `results` has one entry for each conversation that was scored.
- `scores` has one entry for each criterion, in your rubric's order.
- Every `score` is a whole number from 1 to 5.
- `overall` is the average of that conversation's scores, rounded to two decimals.

`results.schema.json` is the same format written as a JSON Schema.

## FAQ

**Can Claude write my whole rubric?** It can draft, and you should edit. You'll explain every line of it to your neighbor.

**The viewer says my file doesn't match the results format.** Copy the list of problems, paste it to Claude, and ask it to fix `score.py`.

**I get a rate limit error (429).** The whole class shares one key. Wait a minute and run it again. If it keeps happening, tell me.

## About the data

The conversations are a sample of 100 from the [E-Commerce Customer Support Conversations](https://huggingface.co/datasets/NebulaByte/E-Commerce_Customer_Support_Conversations) dataset by NebulaByte on Hugging Face, which has 1,000. They were written by a language model, not by real customers and agents, and the names, order numbers and email addresses in them are made up. I drew the sample with more unhappy customers than the full dataset has (60 of the 100 are labeled negative or frustrated), because hard conversations are what test a rubric.
