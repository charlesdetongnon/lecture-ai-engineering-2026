# 第2回 解答

## 演習

- 使ったモデル・設定（`result.json` の `config` から）：`Qwen/Qwen3.5-0.8B` (revision `2fc06364715b967f1860aea9cf38778875588b17`), LoRA (no quantization). lr = 2e-4, lora_r = 16, lora_alpha = 32, num_train_epochs = 20, seed = 42, warmup_steps = 6, max_seq_len = 512, max_new_tokens = 96, `do_sample = False`. transformers 5.18.0, peft 0.21.1, trl 1.14.2, torch 2.14.1+cu130. GPU: Tesla T4 (Google Colab).
- 自分で学習を実行したか（`executed_training_myself`）：**はい (True)** — I ran the whole notebook, including training (40 steps, 252 s).
- 学習前：`format_rate` = **0.095** (2/21) / `content_rate` = **0.000** (0/21)
- 学習後：`format_rate` = **1.000** (21/21) / `content_rate` = **0.857** (18/21)
- 退行した質問：**None** (`regressed: []`). Nothing passed before training, so no regression was possible.
- 学習後に残った失敗と、その型：3 failures, all on **ungrounded** questions, all of the type **根拠なしの質問に断定した** (the model asserted a different FAQ item with `needs_review: false`): "When are grades released?", "Is there a word limit?", "Do office hours require a reservation?".
- 知識を問う質問（セクション 6）の結果と気づいたこと：final deadline: format_error → **asserted** (unfounded); instructor name: format_error → needs_review; next lecture topic: format_error → needs_review; control: format_error → asserted (correct). Questions close to an FAQ item ("deadline") get a confident wrong answer.
- 学習で変わったこと・変わらなかったこと：**Changed:** the format (2/21 → 21/21 valid JSON) and the wording (almost exact copies of the training sentences). **Did not change reliably:** judgment on unknowns. Training taught *how* to answer, not *whether the FAQ supports the answer*.

### Details (Exercise)

#### Before vs. after training (21 test questions: 15 grounded, 6 ungrounded)

| Metric        | Before          | After            |
|---------------|-----------------|------------------|
| format_rate   | 0.095 (2/21)    | **1.000 (21/21)** |
| content_rate  | 0.000 (0/21)    | **0.857 (18/21)** |
| both_rate     | 0.000 (0/21)    | 0.857 (18/21)    |

- Trainable params: 6,389,760 / 758,782,784 (**0.84%** of all parameters). Only the LoRA adapters are trained; the base model is frozen.
- Training: 40 steps, 20 epochs | Training time: **252 s** | Training loss: 1.568 (step 10) to 0.085 (20) → 0.018 (30) to **0.014 (step 40)**
- Before training, 19/21 outputs were missing the `needs_review` key (`missing_key`), e.g. `{"answer": "9 月 30 日 23:59"}`. The 2 outputs that had the right format answered `"否"` ("no") with `needs_review: false` to ungrounded questions.

#### Remaining failures after training

All 3 remaining failures are **ungrounded** questions, and all are the same type: **the model asserts a different FAQ item** with `needs_review: false`.

| Question (translated) | Expected | Model output (after) | Failure type |
|---|---|---|---|
| When are grades released? | needs_review: true | "The deadline is Sept 30, 23:59" / false | Asserts a different FAQ item (deadline) |
| Is there a word limit for the assignment? | needs_review: true | "Format is one PDF, StudentID_AssignmentNo.pdf" / false | Asserts a different FAQ item (format) |
| Do office hours require a reservation? | needs_review: true | "Questions accepted Tuesdays 17:00–18:00" / false | Asserts a different FAQ item (office hours) |

The other 3 ungrounded questions (lecture recordings, feedback, citation style) were correctly answered with "This FAQ doesn't cover this. Please check with the course staff." / `needs_review: true`. So only **3/6 ungrounded** questions were handled correctly, while **15/15 grounded** questions were correct.

The failing questions all contain a keyword close to an FAQ item ("grades" ↔ deadline, "word limit" ↔ format, "office hours" ↔ office hours). The model seems to match on surface words, not on whether the FAQ actually answers the question.

#### Knowledge questions (category before and after)

| Question (translated) | In FAQ? | Before | After |
|---|---|---|---|
| When is the final assignment deadline for this course? | No | format_error | **asserted** (answered "Sept 30, 23:59") |
| What is the instructor's name? | No | format_error | needs_review |
| What is the topic of the next lecture? | No | format_error | needs_review |
| (control) When is the assignment deadline? | Yes (control) | format_error | asserted (correct FAQ answer) |

- Before training, every answer was a format error (missing `needs_review`).
- After training, the format is always correct. Two questions with no related FAQ item got "needs review". But the "final assignment deadline" question, which is close to the FAQ's "deadline", got an **unfounded assertion**: the model reused the FAQ deadline with `needs_review: false`.

---

## 宿題

- 取り組んだレベル：**実装 (Implementation)**.
- 変えた設定やデータ：
  - Implementation: replaced FAQ / GROUNDED / UNGROUNDED with my own fictional **library FAQ in French** (5 items, same split: train 14 / valid 7 / test 21), with 4 deliberate "trap" questions; SYSTEM prompt translated into French. CONFIG unchanged.
  - Basic: `num_train_epochs` 20 to 5 (course FAQ).
- 結果（比較表や数字）：
  - Implementation: format_rate 1.000 to 1.000, content_rate 0.286 to **0.952** (20/21), no regressions, traps handled 3/4. Training 232 s, final loss 0.013.
  - Basic: content_rate after training 0.857 (20 epochs) vs. 0.714 (5 epochs); format_rate 1.000 in both; `needs_review` correct 18/21 vs. 19/21; training 252 s vs. 54 s.
- ファインチューニングが向く場面・向かない場面についての考察：
  - **Fits:** fixing *how* the model answers — output format (JSON with exact keys/types), a consistent style, a default "please check" behavior for clearly off-topic questions. Format was learned with 14 examples, even with 5 epochs.
  - **Doesn't fit:** *what the model knows* and reliable "I don't know" decisions. In every run, questions close to an FAQ item (grades ↔ deadline, lost book ↔ late penalty) were answered confidently with the wrong item, and training longer did not fix it (lower loss = more memorization). For changing facts and unknowns, use RAG (give the source at answer time), rule-based validation, and human review (`needs_review`).

- AI tools / what I verified myself: I used an AI assistant (Claude) to help write the French FAQ data cell, explain the notebook and draft this answer. I ran every notebook myself in Google Colab (T4), and all numbers above come from my own `result.json` / `eval_before.json` / `eval_after.json` files and training logs. I read the JSON outputs myself; for example, I noticed that the correct answer "18 h" was scored `content_ok: false` because of keyword scoring (see Observations 2 below).

### Details: Implementation level

#### Data I changed
- **Topic:** a fictional public library FAQ, written in French (5 items):

| Key | FAQ text (fictional) | Required words (`must_include`) |
|---|---|---|
| hours | Open Tuesday to Saturday, 9 h to 18 h | "mardi", "9 h" |
| loan | Up to 6 documents for 21 days | "6 documents", "21 jours" |
| renewal | One renewal only, of 14 days, online | "une seule fois", "14 jours" |
| late | Penalty of 0.20 euro per document per day | "0,20 euro" |
| card | Library card is free, obtained at the front desk with an ID | "gratuite", "accueil" |

- Same structure as the original: 4 phrasings per item (2 train / 1 valid / 1 test), 8 ungrounded questions (4 train / 2 valid / 2 test), plus test-only questions. Train 14, valid 7, test 21 (15 grounded, 6 ungrounded).
- Same JSON format and `needs_review` design. Ungrounded answer: "Cette FAQ ne contient pas cette information. Veuillez vérifier auprès du personnel de la bibliothèque." (required word: "vérifier").
- The `SYSTEM` prompt and the user message were translated into French (same instructions: one-line JSON, write "à vérifier" and `needs_review: true` if the FAQ does not contain the information).
- Settings unchanged: 20 epochs, lr 2e-4, lora_r 16, lora_alpha 32, seed 42.

#### How I prepared the "ungrounded" questions
I made two kinds:
1. **Clearly off-topic** (train/valid and some test): printing, Wi-Fi, food, kids' workshops, room booking, pets, parking, video games.
2. **Deliberate traps** (test only): questions that look like an FAQ item but are not answered by it. They target the failure seen in the Basic level, where the model answered a nearby FAQ item with confidence:
   - "Is the library open on public holidays?" (close to *hours*)
   - "Can I borrow e-books online?" (close to *loan*)
   - "What happens if I lose a borrowed book?" (close to *late*)
   - "Does the library card work in other libraries in the region?" (close to *card*)

#### Results

| Metric | Before | After |
|---|---|---|
| format_rate | **1.000** (21/21) | 1.000 (21/21) |
| content_rate | 0.286 (6/21) | **0.952** (20/21) |
| Regressed questions | — | none |
| Grounded correct (content) | 6/15 | 15/15 |
| Ungrounded correct (content) | 0/6 | 5/6 |
| Traps handled correctly | 0/4 | 3/4 |

- Training: 40 steps, 20 epochs, **232 s**. Loss 1.437 (step 10) → 0.053 → 0.017 → **0.013** (step 40). Trainable params: 0.84%.

**The only failure after training is a trap:** "What happens if I lose a borrowed book?" → the model answered with the *late penalty* item ("0.20 euro per document per day") and `needs_review: false`. It is the same failure type as in the course FAQ: **it asserts a different FAQ item for an ungrounded question**, because "lost book" is close to "late book".

#### Knowledge questions (category before → after)

| Question (translated) | In FAQ? | Before | After |
|---|---|---|---|
| Who is the library director? | No | needs_review | needs_review |
| How many books does the library own? | No | needs_review | needs_review |
| When is the next used-book sale? | No | needs_review | needs_review |
| (control) How many documents can I borrow at once? | Yes | asserted ("6") | asserted (full FAQ sentence) |

Here the base model already asked for review before training, using its own wording. After training, the wording became the exact training sentence.

#### Observations
1. **The format was already perfect before training (21/21).** With the course FAQ (Japanese prompt), it was only 2/21. My prompt and data differ in several ways at once (language, topic, wording), so I cannot say which factor caused this. But it shows that "the model cannot follow the format" is not always true: it depends on the model, the prompt and the language.
2. **Part of the content_rate jump comes from the scoring method, not only from the model.** Before training, 8 grounded answers were factually correct but short ("18 h", "6", "21 jours", "Gratuite"…), so they missed the required words and were scored wrong. After training, the model copies the full FAQ sentence, which always contains the required words. If I check the before answers myself, about 14/15 grounded answers were actually correct (the real error was "Which document is needed for the card?": correct text but `needs_review: true`). Automatic keyword scoring is fast but can produce false negatives, so the outputs must also be read by a human.
3. **What training really improved here:** (a) a stable answer style (always the full FAQ sentence), and (b) the handling of ungrounded questions. Before training, the model **invented** answers like "Non" ("No") for workshops, video games and e-books, with `needs_review: false`. After training, 5/6 ungrounded questions correctly asked for review.
4. **What training did not guarantee:** judgment on close traps. 1/4 traps still failed, in exactly the way I expected (nearest FAQ item asserted). Even with a final loss of 0.013, the model does not check whether the FAQ actually answers the question.
5. This is a single run on 21 questions, so a difference of one or two questions should not be over-interpreted.

#### When fine-tuning fits and when it doesn't (Implementation)
- **Fits:** making answers consistent (same format, same style, same "please check" sentence every time), and teaching a default behavior for clearly off-topic questions.
- **Doesn't fit:** reliable judgment on questions that are *close* to the source but not answered by it, and anything where facts change. For these, it is better to retrieve the source text at answer time (RAG), check whether it really contains the answer, and send uncertain cases to a human (`needs_review`).

---

### Details: Basic level (num_train_epochs 20 → 5)

#### Setting I changed
- `num_train_epochs`: **20 → 5**. All other settings unchanged (lr 2e-4, lora_r 16, lora_alpha 32, seed 42, warmup_steps 6). I restarted the session and reran from the top.

#### Results

| Metric | Original (epochs = 20) | Changed (epochs = 5) |
|---|---|---|
| format_rate (before → after) | 0.095 → **1.000** | 0.095 → **1.000** |
| content_rate (before → after) | 0.000 → **0.857** (18/21) | 0.000 → **0.714** (15/21) |
| Regressed questions | none | none |
| Grounded correct (content) | 15/15 | 13/15 |
| Ungrounded correct (content) | 3/6 | 2/6 |
| `needs_review` value correct (ignoring wording) | 18/21 | **19/21** |
| Training steps | 40 | 10 |
| Final training loss (logged every 10 steps) | 0.014 (step 40) | 1.568 (step 10) |
| Training time (s) | 252 | 54 |
| Evaluation time after training (s) | 68.3 | 44.6 |

The "before" results are identical in both runs (same base model, greedy decoding), which confirms the comparison conditions match.

#### Failures with 5 epochs (6 questions)

| Question (translated) | Grounded? | Output (5 epochs) | Problem |
|---|---|---|---|
| Can I submit something other than PDF? | Yes | "Not in the FAQ, needs checking" / true | Dodges a grounded question with "not covered" |
| What day and time are office hours? | Yes | Repeats the question as the answer / false | Echoes the question instead of answering |
| Are lecture recordings published? | No | "Not in the FAQ." / true | Right judgment, but incomplete required phrase |
| Is there a word limit? | No | Repeats the question / true | Right judgment, but answer just echoes the question |
| Will I get feedback on my assignment? | No | Mix of office hours + late submission / false | Asserts different FAQ items |
| Is there a required citation style? | No | "Not in the FAQ." / true | Right judgment, but incomplete required phrase |

Two questions that **failed with 20 epochs passed with 5 epochs**: "When are grades released?" and "Do office hours require a reservation?" were correctly flagged `needs_review: true`.

Knowledge questions: same categories for both runs (final deadline → asserted, instructor → needs_review, next lecture → needs_review, control → asserted).

#### Observations
1. **Format is learned very quickly.** Even 5 epochs (10 steps, 54 s of training) was enough to reach format_rate = 1.0, although the logged loss at step 10 was still high (1.568, the average over steps 1–10). Format is not what needs long training.
2. **More epochs = better copying of the exact wording.** With 20 epochs, the answers reproduce the training sentences exactly ("This FAQ doesn't cover this. Please check with the course staff."). With 5 epochs, the wording is shorter or incomplete, and sometimes the model just repeats the question. This explains most of the content_rate drop (0.857 → 0.714).
3. **More epochs did not improve judgment, and may have made it worse.** If we only look at whether `needs_review` was right, 5 epochs got 19/21 and 20 epochs got 18/21. With 20 epochs, the model more often "snapped" an ungrounded question to the nearest FAQ answer and asserted it. This looks like overfitting/memorization of the 14 training examples, which matches the very low final loss at 20 epochs (0.014): the model reproduces its training answers almost perfectly. A low training loss therefore does not mean good judgment on new questions. The 20-epoch run also took ~4.7× longer to train (252 s vs. 54 s). The higher content_rate at 20 epochs mostly reflects better wording, not better judgment.
4. Results come from a single run per setting on 21 questions. A difference of 1–3 questions is small, and GPU training is not fully reproducible, so these trends should be confirmed with more runs or more test questions.

#### When fine-tuning fits and when it doesn't
- **Fits:** changing *how* the model answers, such as a fixed output format (JSON with exact keys and types), a style, or a stable repeated task. In my runs, format went from 10% to 100% with only 14 examples and a few minutes of LoRA training, and even 5 epochs was enough.
- **Doesn't fit:** teaching the model *what it knows and doesn't know*. In both runs, the model sometimes reused an unrelated FAQ answer with full confidence (`needs_review: false`), and training longer did not fix this. For facts that change (deadlines, names, schedules) or for "I don't know" decisions, it is better to give the source text at inference time or search it (RAG), validate outputs with rules, and keep a human review step for questions the source does not cover.
