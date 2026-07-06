# Aloqa Onboarding Documentation

This folder is a beginner-friendly guide to the Aloqa project.

It is written for a new project manager who understands software in general, but does not know this codebase yet.

Do not read this like a code reference. Read it like an onboarding book.

## How to read these documents

Start here:

1. `01-executive-summary.md`
2. `02-system-architecture.md`
3. `07-features.md`
4. `10-request-flows.md`

Then read the deeper chapters:

5. `03-frontend.md`
6. `04-backend.md`
7. `05-database.md`
8. `06-api.md`
9. `08-authentication.md`
10. `09-realtime.md`

Use these when planning work:

11. `11-technology-stack.md`
12. `12-project-structure.md`
13. `13-code-quality.md`
14. `14-technical-debt.md`
15. `15-project-manager-guide.md`
16. `16-glossary.md`

## What this documentation tries to do

It answers these questions:

- What is Aloqa?
- What does a user do in Aloqa?
- What happens after the user clicks a button?
- Which frontend screen is involved?
- Which backend service is involved?
- Which database tables are involved?
- What can break if we change this?
- How expensive is a change likely to be?
- Where should an engineer look in the code?

## The two main codebases

Aloqa is stored under:

```text
/Users/mahmud/Projects/aloqa
```

Inside that folder there are two main projects:

```text
aloqa/
  aloqa-frontend/   -> what users see and click
  aloqa-backend/    -> what stores data and enforces rules
  docs/project-audit/ -> these onboarding documents
```

Think of it like a company office:

```text
Users enter through the front lobby
        |
        v
Frontend shows screens and buttons
        |
        v
Backend departments do the work
        |
        v
Database warehouse stores records
```

## Important note about confidence

This documentation is based on source-code inspection. It explains what the code and configuration show.

Some things cannot be proven from code alone, such as:

- which nginx file is really active in production
- how much traffic production receives
- whether all migrations ran successfully in production
- whether every feature is fully tested by QA
- whether dashboards and alerts exist outside the repo

When something cannot be proven, the documents say so directly.

## What you should remember

- Aloqa has two main codebases: frontend and backend.
- The frontend is what users see.
- The backend is where business rules and stored data live.
- These documents are written in onboarding order, not code order.
- Start with the big picture before reading technical details.
- Every chapter ends with a short memory section.
- File paths are included so engineers can verify claims later.
