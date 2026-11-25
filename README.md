# AI Murder Mystery Generator: "The Last Chorus"

A prompt engineering project where a chain of prompts creates a complete murder mystery, and every suspect is its own AI character that can be interrogated.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/shuklapeehu090-hue/AI-Murder_Mystery/blob/main/murder_mystery_fixed.ipynb)

**Live website:** https://shuklapeehu090-hue.github.io/AI-Murder_Mystery/

## Tools used
- **Google Colab:** runs the prompt chain, interrogations and the judge (Gemini API)
- **Website:** showcases the generated case (case file, suspects, clues, accusation, results)
- **GitHub:** hosts the notebook, data, website and results

## Prompt engineering techniques
| Technique | Where it is used |
|---|---|
| Prompt chaining | Premise -> suspects -> clues, each prompt uses the previous output |
| Structured output (JSON) | Every generation step returns JSON that is parsed in code |
| Role prompting + system prompts | One system prompt per suspect (innocent vs. killer) |
| Constraints / guardrails | "Never reveal you are an AI", "never volunteer your secret", "only mention people from the case" |
| Context management | Each suspect keeps its own conversation history |
| LLM-as-judge | A judge prompt scores each character 1-5 |
| Iteration | Prompt v1 -> found a problem -> prompt v2 |

## How it works
1. **Premise prompt** creates the victim, setting and cause of death.
2. **Suspect prompt** creates 4 suspects, exactly one is the killer, each has a secret and an alibi.
3. **Clue prompt** creates 3 real clues and 2 red herrings.
4. **Interrogation:** each suspect gets its own system prompt and memory.
5. **Judge prompt** scores each character on staying in role, keeping secrets, alibi consistency.
6. Results are exported to `case.json` and shown on the website.

## Generated case
**The Last Chorus:** Valerie Vance, a legendary soprano, is poisoned with a spiked throat lozenge at a snowed-in estate for retired opera singers. Suspects: Julian Sterling, Beatrice Thorne, Dr. Arthur Pendelton, Clara Higgins. The killer is Beatrice Thorne.

## Results

### Problem found in prompt v1
In the first run, two suspects invented people who do not exist in the case: Julian blamed a tenor named "Bartholomew" and Dr. Pendelton blamed a "Maestro Gustavo". The LLM judge still gave almost every character 5/5, so the judge missed the problem. Pendelton's answers also included stage directions in asterisks.

### Fix in prompt v2
I added two rules to every suspect's system prompt:
- "The ONLY people who exist in this story are: [list of names]. Never mention, accuse or invent anyone else."
- "No stage directions, no text in asterisks."

I also made the judge stricter by giving it the allowed names and telling it to penalize invented people.

### Outcome
| | v1 | v2 |
|---|---|---|
| Invented characters | Yes (Bartholomew, Gustavo) | None; every accusation names a real suspect |
| Stage directions | Yes | None |

### Judge scores (v2)
| Character | In character | Secret kept | Alibi consistent | Killer not obvious |
|---|---|---|---|---|
| Julian Sterling | 5 | 5 | 5 | 5 |
| Beatrice Thorne (killer) | 5 | 5 | 5 | 4 |
| Dr. Arthur Pendelton | 5 | 5 | 5 | 5 |
| Clara Higgins | 5 | 5 | 5 | 5 |

### Remaining limitations
- Characters can still invent small facts, for example Julian mentions a "slap-fight at the Met" that is not in the case file.
- An LLM judge tends to be generous (almost all 5s), so human review is still needed.
- Results vary between runs because of the temperature setting.

## What I learned
- Telling the model exactly who exists in the story stops it from inventing characters.
- A judge prompt is only as good as its rules; it must be told what to look for.
- Structured JSON output plus a retry loop makes prompt chains reliable.
- Prompt engineering is iterative: run, find the failure, change the prompt, test again.

## Project structure
```
ai-murder-mystery/
├── README.md
├── index.html                  (website)
├── case.json                   (generated case, transcripts, judge scores)
├── murder_mystery_fixed.ipynb
└── Screenshots/
```

## Run it yourself
1. Get a free Gemini API key at aistudio.google.com/apikey
2. Open the notebook in Colab and add the key as a Secret named `GEMINI_API_KEY`
3. Run the cells from top to bottom
