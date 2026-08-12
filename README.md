# agent-experiment
Using latest models to improve my AGENTS.md - here is the results

## The Setup

I took my AGENTS.md from Nostria, which is a fairly old file. I manually modified it first with a few tips from Theo (T3): https://youtu.be/e1snsuY4lTI

The harness I used was Visual Studio Code Agents. Default thinking and context for all models.

Using GitHib Copilot subscription.

### Prompt

```
Read my AGENTS.md file and see if there is any improvements that can be made to improve the quality of model output.

I don't rely on one specific model, I use most of the latest AI/LLM models that is available.

I'm the only developer on this project and I'm an extremely skilled developer, so I don't want the models to treat me like a novice.

I need quality, but I'm also after innovation and speed. Don't ask me questions, just update the file how you see fit.
```

### Results

[AGENTS.md (Original)](AGENTS.md)

[MAI-Code-1-Flash](MAI-Code-1-Flash.md)   
23.8 credits

[Grok 4.5](Grok-4.5.md)   
64.2 credits

[GPT-5.6 Terra](GPT-5.6-Terra.md)   
16.2 credits

[GPT-5.6 Sol](GPT-5.6-Sol.md)   
60.2 credits

[Kimi K3](Kimi-K3.md)   
50.9 credits

[Gemini 3.6 Flash](Gemini-3.6-Flash.md)
32.6 credits

[Opus 5](Opus-5.md)   
498.8 credits


## Conclusion

Opus: There is no doubt that Opus 5 spent the most time, it searched the codebase a lot more than the other models. It even compiled and ran the app, ran the test suites. It's absolutely wasting insane amount of tokens for a simple task. It even started modifying my codebase (3 unit tests it modified) and not just AGENTS.md. The end result in AGENTS.md, also contains a lot of details about tests and things not working. I don't want that in my AGENTS.md file. Example: `As of v4.1.67: **~407 failing / 1434 tests, 64 failing spec files.** Almost all are stale hand-rolled mocks`. This does absolutely not belong in AGENTS.md file.

Terra: Made too few changes, assumes too much that the current document is correct (which it is not).

Sol: Made many changes, very good improvements. Fixed some issues with testing framework, also includes some knowledge of generic components in the solution.

Gemini: Decent changes and useful improvements. Though a bit too generic and not enough knowledge of the code base.

Grok: Very good result, fairly verbose but it did an impressive job.

Kimi: Made too few changes, didn't fix issues and doesn't know the code base well.

MAI: Made a fully new top part of the file, very well written. Didn't doo too much modifications in the latter parts.


Not useable:

- Terra
- Kimi

Decent results:

- Grok
- Gemini

Good results:

- Sol
- MAI

Too much, not useable:

- Opus
