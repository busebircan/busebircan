---
name: quiz
description: Quiz me on ML, MLOps, the tech stack or insurance ML, with spaced repetition from progress.md. Use when I ask to be quizzed, tested or to review.
---

# Quiz

1. Read `progress.md`. If I didn't name a topic, pick one from "Weak topics to review" first, then from recently completed modules.
2. Ask **one question at a time**, 5 in total by default. Mix the types:
   - concept ("Why does a Poisson GLM use a log link?")
   - debugging ("This pod is `CrashLoopBackOff` and the logs say... what do you check?")
   - design ("How would you roll out a new pricing model safely?")
   - insurance or regulation ("What makes a model high-risk under the EU AI Act?")
3. After each answer: say whether it's right, partly right or wrong, give the key point I missed in 1–3 sentences, then ask the next question.
4. At the end: give the score, then update `progress.md`:
   - add a row to the quiz log (date, topic, score, missed concepts)
   - add missed concepts to "Weak topics to review"
   - remove a weak topic once I've answered it correctly in two separate quizzes
5. Increase difficulty when I score 4/5 or more, decrease it at 2/5 or below.
