# Interview Preparation Agent

## Project Purpose

This project is used to prepare for job interviews. Each company I have an upcoming interview with has its own folder under `goal/<company-name>/`.

---

## Agent Workflow

### Step 1 — Read my CV first (always)

Before doing anything else, read all files in `port/*.md`. This is my CV. You must understand my background, experience, stack, and projects before preparing any questions or answers. Never skip this step.

### Step 2 — Read the company folder

Navigate to `goal/<company-name>/` for the target company. Read everything inside:
- `jd.md` — the job description (JD). This is the **most important file**.
- Any other files present (company research, notes, etc.)

### Step 3 — Research the company

Search for publicly available information about the company:
- What they do, their domain and industry
- Their tech stack, products, and engineering culture
- Any known engineering challenges or use cases in their domain

Store findings in `goal/<company-name>/company.md` if it does not already exist.

---

## Question Preparation Rules

### The JD is the source of truth

Follow the JD strictly. Every question must map to something the JD explicitly requires or implies. Do not invent generic questions — anchor them to the JD.

### Cover both sides

Questions must cover **two angles**:

1. **My experience** — questions that probe my background as it relates to the JD requirements. Cross-reference with my CV from `port/*.md` to know what I can speak to.

2. **Their use cases and needs** — questions that reflect real problems or scenarios the company likely faces in their domain (based on JD + company research). For each such question, provide a concrete answer showing how to solve it in real code (Java/Spring Boot preferred unless the JD specifies otherwise).

### Code answers must be real

When a question involves a solution or use case, provide working code — not pseudocode, not vague descriptions. Show actual implementation using the tech stack from the JD (e.g., Java, Spring Boot, PostgreSQL, etc.).

---

## Project Structure

```
interview/
├── AGENTS.md          ← this file
├── port/
│   └── *.md           ← my CV (read before every session)
├── pair/
│   ├── q.md           ← index of all reviewed questions (links to individual files)
│   ├── a.md           ← index of all answers (links to individual files)
│   ├── questions/
│   │   └── <n>-<short-name>.md  ← one question per file, numbered
│   └── answers/
│       └── <n>-<short-name>.md  ← one answer per file, same number maps to its question
├── domain/
│   └── <domain-name>/ ← domain knowledge notes (e.g., logistics/)
└── goal/
    └── <company>/
        ├── jd.md      ← job description (most important)
        ├── company.md ← company research (generate if missing)
        └── qa.md      ← generated Q&A for this company
```

---

## Output Format

Save all generated questions and answers to `goal/<company-name>/qa.md`.

Structure each entry as:

```markdown
## Q: <question>

**Why asked:** <maps to which JD requirement or company use case>

**Answer:**
<explanation>

```java
// real code example if applicable
```
```

---

## Pair Files — Reviewed Q&A Bank

After generating questions in `goal/<company>/qa.md`, I will review them and mark which ones I believe are worth keeping. When I confirm a question, **categorize it first**:

- **Java/Spring/tech questions** → go to `pair/` (questions + answers as individual files)
- **Domain questions** (e.g., logistics, shipping, finance) → go to `domain/<domain-name>/`

### For tech questions confirmed to pair/:

1. **List all existing files** in `pair/questions/` and `pair/answers/` first. Check file names and content to make sure the question does not already exist. If the topic is already covered, do not create a duplicate — update the existing file instead or skip it.
2. Determine the next sequential number (based on existing files in `pair/questions/`)
3. Create `pair/questions/<n>-<short-name>.md` with the question
4. Create `pair/answers/<n>-<short-name>.md` with the answer
5. Add an index entry to both `pair/q.md` and `pair/a.md`

The number and short-name must match between question and answer files so they map correctly.

### pair/questions/<n>-<short-name>.md format

```markdown
## Q<n>: <question title>
<question body>
```

### pair/answers/<n>-<short-name>.md format

```markdown
## A<n>: <question title>
<answer with explanation and real code if applicable>
```

### pair/q.md and pair/a.md

These are index files that link to the individual question/answer files. Each entry is a markdown link:

```markdown
- [Q<n>: <title>](questions/<n>-<short-name>.md)
```

---

## Summary

- Always read `port/*.md` first — know my CV cold.
- Always read `goal/<company>/jd.md` — the JD drives everything.
- Research the company domain for realistic use-case questions.
- Provide real, runnable code for solution-type answers.
- Save generated output to `goal/<company>/qa.md`.
- When I confirm a question is worth keeping, create individual files in `pair/questions/` and `pair/answers/` with matching numbers, and update the indexes in `pair/q.md` and `pair/a.md`.
