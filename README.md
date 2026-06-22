# TakeMeter: Evaluating r/ussoccer World Cup Discourse 

**Demo Video Walkthrough:** `https://www.loom.com/share/a124aa51258141c2ac68b2af51abd765`

TakeMeter is a fine-tuned NLP classification model designed to evaluate the substantive quality of user discourse inside the `r/ussoccer` subreddit during the 2026 FIFA World Cup.

---

## 1. Community Choice & Rationale

* **Target Community:** The `r/ussoccer` subreddit (specifically the daily World Cup open discussion match threads).
* **Why this community fits:** Because the 2026 World Cup is taking place on home soil, the daily open threads act as a chaotic, high-volume linguistic melting pot. Die-hard tactical nerds, highly emotional doomers, celebratory casuals, and international newcomers all collide in a single text box. This creates an extremely wide, messy spectrum of take quality—making it the ultimate stress test for a fine-tuned DistilBERT classifier.

---

## 2. Label Taxonomy

To measure discourse substance while maintaining **>90% community coverage**, we utilize a mutually exclusive 4-label taxonomy:

### `tactical_analysis` (Class 0)
* **Definition:** The post makes a structured, objective argument regarding squad logistics, lineups, or tournament strategy backed up by specific, verifiable constraints (e.g., player fitness, yellow card math, travel schedules, or opponent data).
* **Example 1:** `"So, imo no need to play Puli or anyone on a yellow. Adams, A Rob, Balogun, and Richards."`
* **Example 2:** `"Paraguay went from 17% possession in the first half to 25% by the end of the game. That means they had about 33% in the second half, nearly doubling their possession after going down to 10 men."`

### `hot_take` (Class 1)
* **Definition:** A bold, sweeping, or confrontational claim stated with high conviction but zero verifiable evidence; it relies on emotional assertion or managerial litigation rather than tactical proof.
* **Example 1:** `"Is anybody willing to admit that the last handful of years have been wasted by completely ridiculous managers that have do idea what they are doing?"`
* **Example 2:** `"Turkey getting eliminated after 2 games when their players said such cocky things would be cinema"`

### `hype_and_vibes` (Class 2)
* **Definition:** Pure, ungrounded emotional expression, relief, match banter, or celebration where the user is just reacting in the moment rather than trying to persuade anyone of a point.
* **Example 1:** `"YEAH YOU CAN TELL EVERYBODY YEAH YOU CAN TELL EVERYBODYYYYYY GO AHEAD AND TELL EVERYBODY I’M THE MAN I’M THE MAN I’M THE MANNNNN YES I AM YES I AM YES I AMMMM"`
* **Example 2:** `"Yeah its refreshing to have one game where its world cup but no stress at all. Just go out and have fun."`

### `fandom_inquiry` (Class 3)
* **Definition:** Genuine, neutral questions from casual fans, outsiders, or sub regulars seeking objective clarification on tournament mechanics, scheduling, or soccer history.
* **Example 1:** `"Not an American, but from an outsiders perspective is there any other sporting event that has such nationwide support as the world cup?"`
* **Example 2:** `"Wipes after the group stage, no?"`

---

## 3. Data & Annotation Process

### Collection & Distribution
203 raw comments were manually harvested from the *r/ussoccer World Cup Daily Discussion Thread*. Following automated syntax scrubbing and the removal of one unparseable row, the locked dataset of **202 rows** yielded the following healthy distribution:

* **`hype_and_vibes`:** 88 rows (43.5%)
* **`hot_take`:** 46 rows (22.7%)
* **`tactical_analysis`:** 43 rows (21.2%)
* **`fandom_inquiry`:** 25 rows (12.6%)

The dataset was split **70 / 15 / 15** into `Train` (141 rows), `Validation` (30 rows), and `Test` (31 rows). 

### 3 Genuinely Difficult Annotation Edge Cases

1. **The Loaded Grievance:** *"Why are all the us games on the west coast? Same thing happened in 1994 save Detroit. Makes no sense at all when most of the population lives on the east cost"*
   * *The Dilemma:* It uses the explicit syntax of a `fandom_inquiry`, but carries the underlying animus and sweeping generalization of a `hot_take`.
   * *The Decision:* **`hot_take`**. *Rule:* If an inquiry contains an accusatory premise or asserts a subjective complaint as its foundation, it forfeits its neutral inquiry status.
2. **The Passive-Aggressive "Stat":** *"Brazil scores 3 against Haiti 'ah yes they’re back.' US scores 3 against anyone 'they still need to play a good opponent'"*
   * *The Dilemma:* Cites real match scorelines (Analysis syntax) to construct a purely subjective victimhood narrative regarding international media bias (Hot Take intent).
   * *The Decision:* **`hot_take`**. *Rule:* Data used strictly as a decorative prop to complain about a double-standard does not constitute an objective tactical breakdown. 
3. **The Micro-Analysis:** *"A suspension doesn’t"* (Replying to: *"Wipes after the group stage, no?"*)
   * *The Dilemma:* It is only 3 words long, containing zero player names or formations. 
   * *The Decision:* **`tactical_analysis`**. *Rule:* Regardless of character count, an objective, highly specific statement of factual tournament disciplinary constraints provides verified utility to the community.

---

## 4. Fine-Tuning Approach & Hyperparameter Discovery

The model was fine-tuned over Hugging Face's `distilbert-base-uncased` via Google Colab's T4 GPU. 

### The Key Hyperparameter Decision
Due to the small size of the training split (141 rows), DistilBERT completed its default 3 requested epochs in just **26 total optimization steps** (batch size 16). Because the starter code declared `warmup_steps=50`, the linear scheduler kept the learning rate microscopic, shutting the training down before the optimizer ever woke up. This left the new classification head paralyzed in a flat `0.25` random probability distribution. 

To overcome this severe underfitting, **I dropped `warmup_steps` to `0`, raised the learning rate to `5e-5`, and extended the horizon to `8 epochs` (~72 total steps).** This instantly allowed the classification head to optimize, driving validation loss down from `1.20` to `0.86`.

---

## 5. Baseline LLM Comparison 

To establish a zero-shot baseline, Groq’s `llama-3.3-70b-versatile` was prompted at `temperature=0.0`. It was handed the plain-language definitions, the edge-case ambiguity rules, and the two exact examples for each class committed to the project spec. 

---

## 6. Evaluation Report

Both models were tested against the exact same locked **31-row holdout test set**:

### Overall Accuracy
* **Zero-Shot Baseline (Groq Llama-3.3-70B):** `0.677` (21 / 31 correct)
* **Fine-Tuned Model (DistilBERT):** `0.677` (21 / 31 correct)
* **Net Fine-Tuning Delta:** **`0.000`**

### Per-Class Performance Breakdown

| Label Class | DistilBERT Precision | DistilBERT Recall | DistilBERT F1 | Baseline (Groq) F1 | Support |
| :--- | :---: | :---: | :---: | :---: | :---: |
| `tactical_analysis` | 0.71 | 0.71 | **0.71** | 0.73 | 7 |
| `hot_take` | 0.43 | 0.43 | **0.43** | 0.53 | 7 |
| `hype_and_vibes` | 0.92 | 0.79 | **0.85** | 0.83 | 14 |
| `fandom_inquiry` | 0.40 | 0.67 | **0.50** | 0.29 | 3 |
| **Weighted Avg** | **0.71** | **0.68** | **0.69** | **0.69** | **31** |

### Fine-Tuned Model Confusion Matrix (Test Set)

| True \ Predicted | `tactical_analysis` | `hot_take` | `hype_and_vibes` | `fandom_inquiry` |
| :--- | :---: | :---: | :---: | :---: |
| **`tactical_analysis`** | **5** | 2 | 0 | 0 |
| **`hot_take`** | 2 | **3** | 1 | 1 |
| **`hype_and_vibes`** | 0 | 1 | **11** | 2 |
| **`fandom_inquiry`** | 0 | 1 | 0 | **2** |

***

### Deep-Dive Failure Mode Analysis (3 Wrong Predictions)

1. **Text:** `"It’s just predicted. They have no inside knowledge"`
   * **True Label:** `tactical_analysis` | **Predicted:** `hot_take` (Conf: 61%)
   * *Why it failed:* Directional boundary failure. The model saw short, blunt, skeptical syntax (*"no inside knowledge"*) and mapped it to the heuristic of a cynical doomer Hot Take, failing to capture the underlying semantic context (explaining how FotMob's API algorithmically constructs automated lineup previews).
2. **Text:** `"Most likely. The rule is to curb racist/homophobic shit"`
   * **True Label:** `tactical_analysis` | **Predicted:** `hot_take` (Conf: 72%)
   * *Why it failed:* Lexical over-indexing on profanity. The word *"shit"* acted as an overwhelming heuristic trigger routing the tensors to the `hot_take` bucket, completely overriding the sentence's objective explanation of IFAB's newly codified on-pitch abuse rules.
3. **Text:** `"He's 10000% right like it or not everyone needs time away from their significant other."`
   * **True Label:** `hot_take` | **Predicted:** `tactical_analysis` (Conf: 48%)
   * *Why it failed:* Blindness to hyperbole. The attention mechanism latched onto the pseudo-statistical token `"10000%"` and classified it as grounded math, missing the absolute absurdity of a relationship take inside a soccer thread.

***

### Sample Classifications Table

| Comment Text | True Label | Predicted Label | Confidence | Status |
| :--- | :---: | :---: | :---: | :---: |
| *"WE DID IT!!! WE WON THE GROUP!!!!!!"* | `hype_and_vibes` | `hype_and_vibes` | 94.2% | ✅ |
| *"Paraguay went from 17% possession... to 25% by the end"* | `tactical_analysis` | `tactical_analysis` | 88.4% | ✅ |
| *"Wipes after the group stage, no?"* | `fandom_inquiry` | `fandom_inquiry` | 81.1% | ✅ |
| *"God I hate him so much... turned into shit shows."* | `hot_take` | `hype_and_vibes` | 76.0% | ❌ |

* **Why the correct prediction for Row 2 makes sense:** The model's attention heads successfully ignored the team names and locked onto the delta calculation syntax (`"went from"`, `"% possession"`) to correctly identify the post as grounded numerical evidence.

---

## 7. Reflection: Intended vs. Learned Behavior

The 67.7% dead-heat tie between DistilBERT and Llama-3.3-70B reveals a profound NLP reality regarding subjective internet text. 

I *intended* for the model to capture the abstract semantic concept of **persuasive rigor** (whether a post proved its point). In practice, the model **overfit to localized surface heuristics**. It learned that `%` signs equal Analysis; that ALL CAPS equal Hype; that question marks equal Inquiries; and that swear words equal Hot Takes. When a user swore while delivering a brilliant tactical point (Failure #2), or used a percentage sign to deliver a trash opinion (Failure #3), DistilBERT's heuristic house of cards collapsed. 

Conversely, Groq's Llama-70B achieved its 67.7% score through sheer, generalized semantic intelligence—meaning a microscopic 66M-parameter encoder memorizing local subreddit heuristics was able to pull off an exact performance tie with a 70B-parameter enterprise supercomputer.

---

## 8. Spec Reflection (`planning.md`)

* **How the spec helped:** Establishing the strict *"Loaded/Grievance Question"* rule in Section 3 of my `planning.md` saved my fine-tuned model's test set. Without pre-defining that rule, standard LLM pre-labeling would have thrown every single sentence ending in a question mark into `fandom_inquiry`, polluting the class with passive-aggressive rants.
* **Where implementation diverged:** In Section 4 of `planning.md`, I established a contingency plan to abandon sequential scraping and actively search the Reddit DOM for keywords like *"lineup"* or *"sub"* if `tactical_analysis` fell below 20%. In practice, bulk-highlighting 200 sequential comments yielded a natural 21.2% analytical distribution, allowing me to bypass active keyword harvesting entirely.

---

## 9. AI Usage Disclosure

1. **Unstructured Text Chunking:** I utilized Groq's `llama-3.3-70b-versatile` to ingest a messy, unstructured wall of copy-pasted Reddit DOM text and parse it into 203 clean `"text","label"` CSV rows. I then conducted a 100% human audit of the CSV in Google Sheets, manually overriding the LLM's zero-shot pre-labels on roughly 15% of the dataset to establish my personal ground truth.
2. **Optimization Diagnosis:** When my fine-tuned model output `0.26` uniform confidences across the board, I pasted my `TrainingArguments` block into an LLM and asked it to calculate my step horizon. It successfully diagnosed that an 8.8-step epoch combined with `warmup_steps=50` was starving the optimizer of learning rate power, leading directly to my `warmup_steps=0` breakthrough.
