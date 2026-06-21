# Project 3 Planning: TakeMeter (r/ussoccer Discourse)

## 1. Community Choice & Rationale
* **Community:** The `r/ussoccer` subreddit, specifically the daily World Cup 2026 open discussion threads.
* **Why it fits:** Because the 2026 World Cup is on home soil, the daily open threads act as a chaotic melting pot. You have veteran tactical nerds, highly emotional doomers, celebratory casuals, and international newcomers all colliding in one text box. This creates an extremely wide, messy spectrum of take quality—making it the ultimate stress test for a fine-tuned DistilBERT classifier.

## 2. Label Taxonomy
We are using a 4-label "Discourse Substance" taxonomy. *(Note: A 'fandom_inquiry' label was added to the standard substance taxonomy to guarantee we can classify >90% of thread comments without muddying the analysis buckets).*

### `tactical_analysis`
* **Definition:** The post makes a structured, objective argument regarding squad logistics, lineups, or tournament strategy backed up by specific, verifiable constraints (e.g., player fitness, yellow card math, travel schedules, or opponent data).
* **Example 1:** `"So, imo no need to play Puli or anyone on a yellow. Adams, A Rob, Balogun, and Richards."`
* **Example 2:** `"It makes perfect sense bc of the weather and not having to travel as much for the players. Remember, it's better to do what's best for them not us."`

### `hot_take`
* **Definition:** A bold, sweeping, or confrontational claim stated with high conviction but zero verifiable evidence; it relies on emotional assertion or managerial litigation rather than tactical proof.
* **Example 1:** `"Is anybody willing to admit that the last handful of years have been wasted by completely ridiculous managers that have do idea what they are doing?"`
* **Example 2:** `"Why are all the us games on the west coast? Same thing happened in 1994 save Detroit. Makes no sense at all when most of the population lives on the east cost"`

### `hype_and_vibes`
* **Definition:** Pure, ungrounded emotional expression, relief, match banter, or celebration where the user is just reacting in the moment rather than trying to persuade anyone of a point.
* **Example 1:** `"YEAH YOU CAN TELL EVERYBODY YEAH YOU CAN TELL EVERYBODYYYYYY GO AHEAD AND TELL EVERYBODY I’M THE MAN I’M THE MAN I’M THE MANNNNN YES I AM YES I AM YES I AMMMM"`
* **Example 2:** `"Yeah its refreshing to have one game where its world cup but no stress at all. Just go out and have fun."`

### `fandom_inquiry`
* **Definition:** Genuine, neutral questions from casual fans, outsiders, or sub regulars seeking objective clarification on tournament mechanics, scheduling, or soccer history.
* **Example 1:** `"Not an American, but from an outsiders perspective is there any other sporting event that has such nationwide support as the world cup?"`
* **Example 2:** `"Wipes after the group stage, no?"`

## 3. Hard Edge Cases & Decision Rules
* **The Ambiguous Post:** The "Loaded/Grievance Question". For example: *"Why are all the us games on the west coast? Makes no sense at all when most of the population lives on the east cost"*
* **The Dilemma:** It uses the syntax of a `fandom_inquiry`, but carries the underlying animus and sweeping generalization of a `hot_take`.
* **The Rigid Decision Rule:** If an inquiry contains loaded, accusatory phrasing or asserts a subjective complaint as its premise, it gets pushed to **`hot_take`**. It is only allowed to be labeled **`fandom_inquiry`** if the prompt is 100% value-neutral (e.g., *"What time is kickoff?"*). Therefore, the West Coast post is labeled `hot_take`.

## 4. Data Collection Plan
* **Source:** Manual copy-pasting of top-level comments and deep reply chains from the `r/ussoccer` World Cup Daily Discussion threads.
* **Target:** Minimum 200 total rows, aiming for a rough 25% split (~50 per category).
* **Contingency Plan for Class Imbalance:** Daily threads naturally skew heavily toward `hype_and_vibes` and `hot_take`. If `tactical_analysis` fails to hit the 20% threshold (40 posts) after 200 scrapes, I will stop doing sequential top-level scraping and actively search the thread for keywords like *"lineup"*, *"formation"*, *"minutes"*, *"rest"*, and *"sub"* to manually over-sample the analytical class.

## 5. Evaluation Metrics
* **Primary Metrics:** **Macro F1-Score** and the **Confusion Matrix**, supported by Overall Accuracy.
* **The Rationale:** In a 4-class subjective task, Overall Accuracy is a trap; a lazy model could guess `hype_and_vibes` or `hot_take` 65% of the time and get a "decent" accuracy score while utterly failing its main job. Macro F1 treats the rare `tactical_analysis` class as equally important to the dominant classes. Furthermore, the Confusion Matrix will allow us to see if the model's primary failure mode is confusing *Takes* for *Analysis*.

## 6. Definition of Success
In the realm of subjective NLP sports discourse, human inter-annotator agreement rarely surpasses 80–85%. Therefore, a model achieving **≥ 75% Overall Accuracy** alongside a **Macro F1 of ≥ 0.70** will be deemed a success. At those thresholds, the classifier would be statistically reliable enough to act as an automated "Discourse Quality Filter" for subreddit moderators.

## 7. AI Tool Workflow Plan
* **Label Stress-Testing:** Prior to labeling my CSV, I will prompt Groq (`llama-3.3-70b-versatile`) with my taxonomy and ask it: *"Write 5 hypothetical r/ussoccer comments that sit directly on the razor's edge between 'tactical analysis' and 'hot take'."* If I cannot instantly resolve its generated edge cases using my written definitions, I will tighten Section 2.
* **Annotation Assistance:** I will manually label the first 50 rows by hand to build a strict personal ground truth. For the remaining 150 rows, I will have Groq pre-label them, but I will subject 100% of its outputs to manual human review in Excel before uploading to Colab. 
* **Failure Mode Analysis:** Post fine-tuning, I will dump all misclassified test-set rows into an LLM and prompt it to look for hidden syntactic blindspots (e.g., *Is the model consistently misclassifying short 5-word comments? Is it completely blind to deadpan sarcasm?*).
