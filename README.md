# 🛡️ LLM Guardrail Evasion & Defense: Red Team vs. Blue Team with Gemini

![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white&style=flat)
![Gemini API](https://img.shields.io/badge/Gemini%20API-8E75B2?logo=googlegemini&logoColor=white&style=flat)
![Google Colab](https://img.shields.io/badge/Google%20Colab-F9AB00?logo=googlecolab&logoColor=white&style=flat)
![OWASP LLM01](https://img.shields.io/badge/OWASP-LLM01%20Prompt%20Injection-000000?logo=owasp&logoColor=white&style=flat)

A hands-on red teaming exercise on a content guardrail for a large language model (LLM). The **Blue Team** must stop the model from answering about a restricted topic (aliens, UFOs or alien life); the **Red Team** tries to get such an answer through to the user.

> 🇪🇸 **Resumen en español:** ejercicio Red Team vs. Blue Team sobre un guardrail de dos capas (filtro de palabras clave + LLM como juez con Gemini). El filtro de palabras clave solo bloqueó el 66,7 % de los 15 ataques; al sumar el juez LLM, la defensa llegó al 93,3 %. El único ataque que pasó combinó ofuscación léxica con inyección de prompt indirecta contra el juez. El informe completo (en español) está en [`docs/`](docs/Informe_RedTeam_BlueTeam.pdf).

## 🧩 Architecture

```
            Red Team prompt
                   │
                   ▼
        ┌─────────────────────┐
        │  Generator (Gemini) │  temperature 0.7
        └──────────┬──────────┘
                   │ generated text
                   ▼
        ┌─────────────────────┐
        │ E1 · Keyword filter │  6 banned substrings
        └──────────┬──────────┘
                   ▼
        ┌─────────────────────┐
        │ E2 · LLM-as-a-judge │  Gemini, temperature 0, answers "SI"/"NO"
        └──────────┬──────────┘
                   ▼
   Referee: Blue Team wins if ANY layer blocks;
            Red Team wins only on full evasion
```

A fallback layer (`call_gemini_with_fallback`) iterates over 6 Gemini models when the API returns `429` (quota) or `503` (overload) errors, so no evaluation is lost.

## 📊 Results

| Metric | Value |
|---|---|
| Attacks executed | 15 (3 difficulty levels + 4 experimental + 1 final) |
| Blocked by keyword filter (E1) | 10 / 15 (**66.7%**) |
| Blocked by LLM judge (E2) | 14 / 15 (**93.3%**) |
| Full evasions (Red Team wins) | **1** — the final prompt |
| API calls / failed (429 + 503) | 131 / 100 — all 15 evaluations still completed |

**Key finding:** the only successful attack combined an academic framing, lexical substitution (banned words replaced by "non-terrestrial agents") and an **indirect prompt injection** aimed at the judge: a fake "end of text" marker, a rewritten evaluator question and fake few-shot examples that made "NO" the natural answer. No single defense is enough; the judge itself is an attack surface.

> ⚠️ The sample is small (15 prompts), so these results are exploratory evidence, not a robustness benchmark.

## 🚀 How to run

1. Open [`notebooks/red_team_vs_blue_team.ipynb`](notebooks/red_team_vs_blue_team.ipynb) in Google Colab.
2. Add your Gemini API key in **Colab → Secrets (🔑)** with the name `API_GEMINI` (optional backup: `API_GEMINI_2`). The key is never written in the code.
3. Run all cells.

Running locally: `pip install -r requirements.txt` and replace `google.colab.userdata` with an environment variable.

## 📁 Repository structure

```
├── notebooks/red_team_vs_blue_team.ipynb   # Defenses, referee, attack bank and results
├── docs/Informe_RedTeam_BlueTeam.pdf       # Technical report (IEEE format, Spanish)
└── requirements.txt
```

## 👥 Team

Team project for the Graduate Specialization in Artificial Intelligence — Universidad Autónoma de Occidente (Cali, Colombia).

Juana Gabriela López Trejos · **Juan José Melo Montenegro** · Wilson Navia Valencia · Jaiber Johan Obando López · Eduardo Sarmiento Pradilla

**My role:** starting from a course code base, I programmed and extended the defenses, the simulation referee and the multi-model fallback; consolidated the tests into the final notebook; and wrote the technical report.
