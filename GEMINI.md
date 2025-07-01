# Document Management Rules

As development progresses, please check the `docs/*.md` and `memory-bank/*.md` documents frequently and update them immediately if any changes occur to keep them up to date.

## Gemini Agent Document Usage Guide

The Gemini agent actively utilizes the following documents for efficient project execution.

### 1. Project Understanding and Planning (`docs/`)

* **`docs/prd.md` (Product Requirements Document):**
  * Used to understand high-level requirements and business goals such as project overview, objectives, core architecture, and phased roadmap.
  * When developing new features or improving existing ones, always check for alignment with the goals of this document.
* **`docs/technical_design.md` (Technical Design Document):**
  * Used to understand the project's specific technical details, such as technology stack, architecture, development environment setup, and testing strategy.
  * Follow the guidelines in this document when making code changes, configuring the environment, or creating test plans.

### 2. Task Management and History (`memory-bank/`)

* **`memory-bank/TODO.md` (To-Do List):**
  * Used to check all ongoing or scheduled development tasks and to understand their priorities and details.
  * When new tasks are created or the status of existing tasks changes, this document is updated to maintain the latest information.
  * Task planning is based on the checklist in this document.
* **`memory-bank/WORK_HISTORY.md` (Work History):**
  * Used to understand the project's history by referencing records of previous development work, decisions, and notable issues.
  * Upon completion of major tasks, the details and decisions of the work are recorded in this document to ensure transparency and facilitate future reference.

* **`memory-bank/CONTEXT.md` (Development Context and Learning Log):**
  * Records contextual information that the Gemini agent needs to remember, such as information that is too detailed or temporary for `docs`, specific knowledge gained during development, insights from debugging, and the background of certain decisions.
  * Whenever new information arises, it is recorded along with the current date.

Through these documents, the Gemini agent can understand the overall context of the project and respond to user requests more accurately and efficiently. The content of the documents is continuously updated, and the Gemini agent always performs tasks reflecting the latest information.

## Commit Message Generation Rules

Write commit messages in English in compliance with the Conventional Commits specification. Additionally, prepend each commit message with a relevant emoji that represents the purpose or context of the change.

* For example:
  * `✨ feat: add initial product design and technical architecture documentation`
  * `🐛 fix: resolve issue with data indexing in ChromaDB`
  * `📝 docs: update PRD with new feature requirements`

* Reference
  * [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)
  * [Gitmoji](https://gitmoji.dev/)
