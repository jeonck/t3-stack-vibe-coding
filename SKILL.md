---
name: local-t3-stack-vibe-coding
description: This skill should be used when building or extending a full-stack TypeScript web app with the T3 Stack (Next.js, TypeScript, Tailwind CSS, tRPC, Prisma or Drizzle, NextAuth.js, Zod) — especially when an AI coding agent writes the code (vibe coding). Covers scaffolding with create-t3-app, the AI development rules (keep the T3 structure, validate inputs with Zod, pass the type check, never bypass errors with any), the end-to-end type-safety workflow, and when to use or skip T3. Use it when the user asks for T3 Stack development, a typesafe Next.js full-stack setup, or rules for AI-written code that must stay type-safe.
---

# T3 Stack Vibe Coding

## Purpose

Build full-stack TypeScript web apps where the AI writes most of the code, without the code tangling as features grow. The T3 Stack keeps one type definition flowing from the database through the server to the screen, so the type checker — not the human — catches mismatches the AI introduces.

## The stack

| Piece | Role |
|---|---|
| Next.js | Web screens + server functions (App Router or Pages Router) |
| TypeScript | Data-format checking in code, end to end |
| Tailwind CSS | Screen design |
| tRPC | Type-safe data connection between screen and server (no codegen, no REST schema drift) |
| Prisma or Drizzle | Database data management |
| NextAuth.js (Auth.js) | Login / authentication |
| Zod | Runtime validation of data inputs; also validates env vars via t3-env |

Source of truth: `t3-oss/create-t3-app` (MIT, ~29,000 stars). Every piece is optional — pick only what the service needs.

## Scaffold

Requirements: Node.js 20 or newer, one package manager (npm / yarn / pnpm / bun).

```bash
npm create t3-app@latest
```

Answer the interactive prompts (TypeScript, Tailwind, tRPC, auth, ORM, App Router, import alias `~/`). For a monorepo with a mobile app, use `create-t3-turbo` instead.

## AI development rules (give these to the coding agent, verbatim)

> Develop while maintaining the existing T3 Stack structure. Validate data inputs with Zod, and write it so it passes the type check. Don't bypass errors with the any type.

Korean original (from the source post):

> 기존 T3 Stack 구조를 유지하면서 개발해 줘. 데이터 입력값은 Zod로 검증하고, 타입 검사를 통과하도록 작성해 줘. any 타입으로 오류를 우회하지 마.

## Workflow

1. Scaffold with `create-t3-app`; commit the clean scaffold before any feature work.
2. Define the data model first (Prisma schema or Drizzle tables). The schema is the single source of truth for shapes.
3. Add tRPC procedures with Zod input schemas. Never accept unvalidated input at a procedure boundary.
4. Build screens against the tRPC client's inferred types. Do not redeclare the same shapes by hand on the client — a hand-written duplicate is where drift starts.
5. Run the type check (`npm run typecheck`) and the build after every feature. Treat a type error as the AI's signal to fix its own change, not as something to silence.
6. Never use `any` to bypass an error, and never add `// @ts-ignore` / `// @ts-expect-error` without a written reason in a comment. If the type is genuinely unknown, narrow it with Zod first, then type the result.

## Why this catches AI mistakes: two canonical examples

- Price type: server defines `price = 10000` (number); the screen sends `price = "10000"` (string). A person reads them as the same value; a program does not. With tRPC the server's type is inferred on the screen, so the mismatch fails the type check before anything runs.
- Rename: the database field changes from `name` to `username`, but the screen still uses `name`. Where types are connected end to end, that is a compile error the AI must fix — not a silent runtime bug a person has to hunt down.

## When to use it

- A service where features keep growing: signup, data storage, dashboard, orders, order history.
- Any project where AI writes most of the code: the more code is delegated, the more the project needs a structure that verifies the rules the code must follow.

## When NOT to use it

- A simple intro/landing page: Next.js + Tailwind CSS alone is enough.
- A codebase that already runs on a different stack: do not force a migration to T3.
- Type checking alone does not solve security or all errors. Auth rules, authorization checks, and input validation still need explicit design.

## Integration with sdlc-runner

Use this skill as the coding-rules input of the sdlc-runner pipeline: the plan step names T3 as the stack, the code step receives the AI development rules above, and the test/CI gate runs `typecheck` + build and stops the pipeline on failure. The skill does not replace the human approval gate.

## Related repos (verified)

- `t3-oss/create-t3-app` — the official CLI and stack definition.
- `t3-oss/create-t3-turbo` — T3 as a Turborepo monorepo (Expo React Native support).
- `nramkissoon/t3-stripe` — Stripe payment integration example on create-t3-app.
- `kaje94/menufic` — restaurant digital-menu generator built on the T3 Stack.

## CK context

- CK's existing TypeScript/React side projects (cti-sentinel, devops-tools/OpsForge, digitaltwin-hub) are Next.js + React + TypeScript frontend apps — the T3 frontend half matches his existing base. tRPC, Prisma/Drizzle, and NextAuth.js are new pieces for him; do not claim prior hands-on with them.
- Direct fit for the small-business direct-ordering SaaS direction: login, database, orders, and dashboard are exactly the T3 pieces. Payment goes through Stripe (see t3-stripe), not through the T3 Stack itself.
