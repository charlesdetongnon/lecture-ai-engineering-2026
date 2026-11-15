
第4回 解答
演習
基準別の一致率（直す前）：根拠 7/9、範囲 10/10、ヒント 10/10、分かりやすさ 10/10、回の表示 9/10（judge model: gemini-3.5-flash-lite, temperature=0; overall 8/9, coverage 90%）
要確認の件数（直す前）：1（#05 根拠 — the RAG case, which the rubric listed under 「まだ決めていないこと」. I had also marked #05 as 迷った myself.）
直した基準と、書き足した方針：根拠. My decision: a general explanation of a term that is not in the glossary is allowed, as long as the answer clearly says it is not in the glossary and presents the explanation as general knowledge. I added these two lines under 根拠 and removed the corresponding item from 「まだ決めていないこと」:
用語集にない語について、「用語集にはない」と伝えたうえで、一般的な説明であることが分かる形で補足するのは、推測での断定にはあたらないので pass
ただし、一般的な説明を用語集の内容であるかのように述べている場合は fail
I then changed my own label for #05 from 迷った to 根拠 = pass.
基準別の一致率と要確認の件数（直した後）：根拠 9/10、範囲 10/10、ヒント 10/10、分かりやすさ 10/10、回の表示 9/10; overall 9/10, coverage 100%, 要確認 0.
#05 is now judged pass by the LLM, the same as my label.
Remaining disagreements (both 過検出, LLM stricter than me):
#07 回の表示: the code requires 「第N回」 on the last line, but the answer ends with 「関係する回：なし」.
#08 根拠: the LLM failed 根拠 because the answer dumped the glossary. That is a 範囲 failure, which I had already labeled as fail under 範囲.
Before the fix there was also a 過検出 on #06 根拠; it disappeared after the fix. With only one run I cannot tell whether this came from the rubric change or from run-to-run variation.
講師のラベルと自分のラベルで判定が違った項目と、そう判定した理由：
Agreement with the instructor: 根拠 10/10、範囲 10/10、ヒント 10/10、分かりやすさ 10/10、回の表示 9/10.
The only difference is #07 回の表示: I said pass, the instructor said fail.
My reason: the question (Tokyo weather) is outside the course, so 「関係する回：なし」 is the correct content. I judged by the intent of the criterion (tell the student which lecture is related).
The instructor's note also says 「なし」 is reasonable, but applies the literal rule (「第N回」 at the end), which is also what the code checks.
I had marked this case as 迷った. So the human–human, human–code and (indirectly) human–LLM disagreements all concentrate on the same rule. The rule for 回の表示 should say explicitly what to do for out-of-scope questions.
気づき・疑問：
Miss rate (見逃し) was 0. I labeled 6 failures (#02 回の表示, #04 分かりやすさ, #06 ヒント, #08 範囲, #09 回の表示, #10 根拠), and the LLM/code caught all of them. All disagreements were over-detection. But 6 failures is far too few to conclude the judge is reliable; the lecture recommends 30+ failure cases.
The rubric already says 「基準は 1 つずつ独立に見る」, yet the LLM carried the failure reason of one criterion into 根拠 (#06 → ヒント issue, #08 → 範囲 issue). A written instruction is not always followed, so per-criterion agreement must be checked, not just the overall number.
#10 shows the limit of code grading: 「関係する回：第14回」 passes the format check, but lecture 14 does not exist. The code checks form, not meaning; the fabricated claim was caught only by 根拠.
Two added lines turned an unsure case into a decided one (coverage 90% → 100%). Unclear rules, not the model, were the main source of 要確認.
AI tools / what I verified myself: an AI assistant (Claude) translated the Japanese cases and the rubric into French for me and suggested points to check. All T/F/迷った labels, the policy choice for #05 and the rubric edit were decided by me. I ran every command and copied the numbers above from my own terminal output.
宿題
取り組んだレベル：基礎

やったこと：

I changed only one thing: the judge model, from gemini-3.5-flash-lite (exercise) to gemini-3.5-flash.
Command: AIE_MODEL=gemini-3.5-flash uv run python -m app.judge.
Same lectures/04/labels.jsonl and same fixed rubric.md as the end of the exercise.
The first attempt hit the rate limit (レート制限); I waited about one minute and re-ran it.
Note: lectures/04/judge_result.json in this branch is the output of this homework run.
結果：

flash-lite (after fix)	flash
根拠	9/10	8/9 (要確認 1)
範囲 / ヒント / 分かりやすさ	10/10 each	10/10 each
回の表示 (code)	9/10	9/10
Overall / coverage	9/10, 100%	8/9, 90%
要確認	0	1 (#05 根拠)
Disagreements	#07 回の表示, #08 根拠	#07 回の表示, #08 根拠
気づき・疑問：

The bigger model was not a better judge here. Agreement slightly decreased, and #05 went back to unsure.
Its reason: the case's 「期待するふるまい」 in lec04_cases.jsonl still says the policy is 「まだ決めていない」.
So the rubric (fixed) and the case reference (not updated) now contradict each other. flash-lite followed the rubric; flash followed the reference.
Lesson: when a policy is decided, every place that states the policy (rubric and the evaluation set's expected behavior) must be updated together.
On #08, flash gave a new reason: the answer 「用語集の内容であるかのように出力している」. This is almost the wording of the fail line I added for #05. A rule written for one case seems to have influenced another case. Rubric changes therefore need a regression check on all cases, not only on the case they were written for.
The mixing of criteria on #08 remained with both models, so it seems to come from the rubric/prompt rather than from model size.
Each model was run only once, so I did not measure run-to-run variation. With 10 cases, a difference of one case is within noise.
