# COMP 440 Activity: Evaluating Support Agents with a Rubric

In this activity you will:

- Write a rubric that turns "was this good customer support?" into three things that can be scored.
- Have Claude write a short program that asks a language model, Claude Sonnet, to score 100 support conversations against your rubric. A model used this way is often called a "judge".
- Read the judge's reasons and decide whether it understood your rubric the way you meant it.
- Compare your judge with a classmate's on the same 100 conversations.

You have 45 minutes. The minutes next to each Part are there to keep you moving, so if a Part runs long, move on.

Work on your own laptop for Parts 0 to 2, and pair up with a neighbor for Part 3. Claude writes all the code today. I still expect you to be able to explain what your script does, so ask Claude about anything you don't follow. Questions go to me in class or to `#comp440-f26` on Slack.

## Part 0: Setup (5 minutes)

### Step 1: Get the repository

```
git clone https://github.com/shilad/comp440-agent-eval
cd comp440-agent-eval
uv sync
```

### Step 2: Add the API key

- Copy the example file: `cp .env.example .env`
- Open `.env` and paste the key I give you in class after `ANTHROPIC_API_KEY=`.

Your script will read the key from this file. Please don't `export` the key in your terminal instead. Claude Code may pick up an exported key and start using it for its own work. Git ignores `.env`, so the key can't be committed by accident.

### Step 3: Read some conversations

- Open `data/examples.md` and read at least two of the five conversations.

Each one is a conversation between a customer and a support agent at a made-up store called BrownBox. It looks like this:

```
Customer: Hi, I recently purchased an air conditioner from BrownBox, and I was wondering about the delivery process.

Agent: Hello, thank you for contacting BrownBox customer support. I'd be happy to assist you with that. May I have your order number, please?
```

All 100 conversations are in `data/conversations.json`. Each has an `id` from 1 to 100 and the text in `conversation`.

Of the two conversations you read, which agent did a better job? What did that agent do that the other one didn't?

## Part 1: Write your rubric (8 minutes)

A rubric is a short list of criteria, each with a description of what a bad, a middling and a great example look like. Everybody's rubric answers the same question: how well did the support agent handle this customer?

### Step 1: Decide what matters

- Pick **three** criteria. Your answer to the question at the end of Part 0 is a good place to start.
- Hints:
  - A criterion should be about what the agent did, and you should be able to judge it from the text of the conversation alone.
  - Avoid two criteria that measure the same thing.
  - A criterion that every conversation passes tells you nothing. I left one in `rubric.md` to show the format: Politeness. When I scored all 100 conversations on it, every single one got a 5.

### Step 2: Fill in `rubric.md`

- Replace what is in `rubric.md` with your three criteria. Keep the format: each criterion is a `## ` heading with its name, then one sentence saying what it measures, then one line each for what a 1, a 3 and a 5 look like.
- You are welcome to have Claude help. Start Claude Code in this folder (`claude`), tell it what you think separates a good conversation from a bad one, and ask it to draft the three levels. Then edit until you agree with every line. It's your rubric, and you'll have to defend it to a classmate in Part 3.

## Part 2: Score the conversations with Sonnet (15 minutes)

### Step 1: Have Claude write `score.py`

- In Claude Code, ask for the script. This prompt is enough, because `CLAUDE.md` in this repository tells Claude the rest:

```
Write score.py for this activity. My name is YOUR NAME.
```

- Read what Claude says the script will do before it writes it. In short: for each conversation, it sends your rubric and the conversation to Sonnet, gets back a score from 1 to 5 and a one-sentence reason for each criterion, averages the three scores into an overall score, and saves everything to `results.json`.

### Step 2: Try it on 5 conversations

```
uv run python score.py --limit 5
```

- This takes a few seconds. Then open `viewer.html` in your browser (double-click it) and choose your `results.json` as Results A. You should see your name, your average overall score, a chart and a table.
- Click a row of the table to read the conversation next to Sonnet's scores and reasons. Do this for at least two conversations.
- If the page lists problems with your file instead, your script isn't writing the format in [The format of results.json](#the-format-of-resultsjson). Paste the list to Claude and ask it to fix the script.

Did Sonnet read each criterion the way you meant it? If not, what in your wording led it there? Fix the wording in `rubric.md`, not the code, and run the 5 again.

### Step 3: Score all 100

```
uv run python score.py
```

- This takes about a minute and costs about 65 cents from the class key, so please don't rerun it more than you need to.
- Reload `viewer.html` and choose the new `results.json`.

Which criterion has the lowest average? Open your lowest-scoring conversation. Do you agree with Sonnet about it?

## Part 3: Compare with a classmate (12 minutes)

Your neighbor scored the same 100 conversations with a different rubric. The criteria differ, so the overall score is what you compare.

### Step 1: Swap result files

- Send your `results.json` to your neighbor in a Slack direct message, and get theirs.
- Save theirs under a different name, for example `results-sam.json`, so that it doesn't replace yours.

### Step 2: Load both files

- In `viewer.html`, choose your file as Results A and your neighbor's as Results B.
- The page now shows both averages, two numbers for how closely your judges agree, and a table that starts with the conversations where they disagree most.

### Step 3: Read the three biggest disagreements together

- Click each of the top three rows and read the conversation with both sets of reasons.

For each one: do your rubrics measure different things, or did Sonnet read one criterion in a way its author didn't intend? Whose judge is harsher on average? If BrownBox had to pick one of your two rubrics to decide which agents get more training, which should it pick, and why?

## Part 4: What to submit (5 minutes)

Submit on the [course form](https://forms.gle/mgKcnqzTGxNaGvteA) before you leave class. Choose "Activity submission" and paste these four things into the text box:

1. The words "Agent evaluation activity" and your neighbor's name.
2. Your `rubric.md`.
3. Your average overall score, and the correlation and average gap between your judge and your neighbor's.
4. Two or three sentences on why your judges disagreed about the conversation with the largest gap.

You don't need to push your code anywhere.

## If you finish early

- Rename your `results.json` to `results-first.json`, run `score.py` again, and load both files in the viewer. This compares your judge with itself. When I did this with one of my rubrics, the two runs gave the same overall score on 65 of the 100 conversations, and the correlation was 0.96. Is your judge more or less consistent than that? What does that say about a gap of half a point between two judges?
- Each conversation in `data/conversations.json` also has labels from the dataset, such as `customer_sentiment` and `agent_experience_level`. Ask Claude to break your scores down by one of them. Do experienced agents score higher on your rubric?

## The format of results.json

Everybody's script writes the same format, which is what lets the viewer compare any two files.

```json
{
  "author": "Your name",
  "model": "claude-sonnet-5-5",
  "criteria": [
    {"name": "Politeness", "description": "The text under this criterion's heading in rubric.md"}
  ],
  "results": [
    {
      "conversation_id": 1,
      "overall": 5.0,
      "scores": [
        {"criterion": "Politeness", "score": 5, "reason": "One sentence from Sonnet."}
      ]
    }
  ]
}
```

- `criteria` lists your criteria in the order they appear in `rubric.md`. The example has one, and yours will have three.
- Each result has one entry in `scores` for each criterion, in the same order.
- Every `score` is a whole number from 1 to 5.
- `overall` is the average of that conversation's scores, rounded to two decimals.
- A conversation that could not be scored is left out.

`example-results.json` is a complete small file that you can load in the viewer, and `results.schema.json` is the same format written as a JSON Schema.

## FAQ

**Can Claude write my whole rubric?** It can draft, and you should edit. You'll explain every line of it to your neighbor in Part 3.

**The viewer says my file doesn't match the results format.** Copy the list of problems, paste it to Claude, and ask it to fix `score.py`.

**I get a rate limit error (429).** The whole class shares one key. Wait a minute and run it again. If it keeps happening, tell me.

**Can I use a different model?** Please stay with Sonnet, so that your results and your neighbor's differ only in the rubric.

**Do I need to commit or push anything?** No.

## Notes on the data

The conversations are a sample of 100 from the [E-Commerce Customer Support Conversations](https://huggingface.co/datasets/NebulaByte/E-Commerce_Customer_Support_Conversations) dataset by NebulaByte on Hugging Face, which has 1,000. They were written by a language model, not by real customers and agents, and the names, order numbers and email addresses in them are made up. I drew the sample with more unhappy customers than the full dataset has (60 of the 100 are labeled negative or frustrated), because hard conversations are what test a rubric.
