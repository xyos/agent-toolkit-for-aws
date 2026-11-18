---
name: aws-blocks
description: Sets up CDK Blocks (formerly AWS Blocks) apps - scaffolds a new app, picks a starter template, or adds a Blocks backend to an existing project. Use only when the request names CDK Blocks, AWS Blocks, cdk-blocks or the @aws-blocks packages.
---

# CDK Blocks (formerly AWS Blocks)

CDK Blocks is a TypeScript framework for full-stack apps on AWS: the backend lives in one `aws-blocks/` folder built from Building Blocks (`KVStore`, `DistributedTable`, `Auth`, `FileBucket`, ...), runs locally without an AWS account and deploys with the CDK.

The product was named AWS Blocks, and the packages are still published as `@aws-blocks/*`. The CLI inside an app is `cdk-blocks`.

## Where to go

| The project | Do this |
|---|---|
| has an `aws-blocks/` folder and an `AGENTS.md` | Follow that `AGENTS.md` and `npx --no -- cdk-blocks docs`. They match the installed version. Nothing below is needed for feature work. |
| is an empty directory (a new app) | [Create an app](#create-an-app) |
| exists without Blocks (Vite, Next.js, an API, Amplify Gen 2, ...) | [references/existing-project.md](references/existing-project.md) |

## Create an app

One command scaffolds the app and installs its dependencies. With `--yes` it never prompts:

```bash
npx --yes @aws-blocks/create-blocks-app <directory> --template <template> --yes
```

`<directory>` is `.` for the current directory, which must be empty (`.git` and editor settings are fine).

The command prints `Installing dependencies...` within a second and then nothing until it ends with `✓ Blocks app created!`, one to three minutes later. `AGENTS.md`, `aws-blocks/` and `test/` exist from that first second; `node_modules`, and with it `npx --no -- cdk-blocks`, only after the last line. If your shell hands the command back still running, read those files, then wait: every check on the install is one call that waits 60 seconds (in Codex, an empty `write_stdin` with `yield_time_ms: 60000`). A shorter poll, `ps` or `sleep` only adds calls.

Pick the template from what was asked for:

| Template | What you get |
|---|---|
| `bare` | Vite frontend and one `greet()` API method. The smallest app; start here when the features are yours to write |
| `backend` | Backend only (API and CDK app), no frontend |
| `api-only` | Backend only, with a public health check and sign-in-gated CRUD as a sample, for mobile, CLI or third-party clients |
| `react` | Vite + React 19 todo app with sign-in, `DistributedTable` and realtime |
| `nextjs` | Next.js 16 App Router; Server and Client Components call the backend |
| `sql` | PostgreSQL backend (`Database`: migrations, foreign keys, transactions) |
| `auth` | Tour of the `Auth` block: email sign-up with a code, groups, profile |
| `agent` | An AI `Agent` with a tool, and a canned model for local runs |
| `cron` | Backend only, scheduled `CronJob`s |
| `observability` | Backend only: `Logger`, `Metrics`, `Tracer`, a CloudWatch dashboard |
| `demo` | Larger sample: `Auth`, `KVStore` and `DistributedTable` |
| `default` | Used without `--template`: Vite + lit-html with sign-in, `DistributedTable` and realtime |

A later version may have other templates: the scaffolder's `--help` lists them. `backend`, `nextjs`, `sql`, `cron` and `observability` do not list `zod` in `package.json` (the other templates do): run `npm install zod` if your code imports it.

## After scaffolding

1. Read the app's `AGENTS.md` before writing code. It was created during this session, so it is not in your context yet, and it is the guide for this version: how APIs are defined and called, errors, sign-in, tests, deploying. It is about 270 lines (30 KB, 8,000 tokens): read it whole with your file-reading tool, or with an output limit that large. A plain `cat` is cut off. Then read `package.json`, `aws-blocks/index.ts` and `test/e2e.test.ts` in one call: the scripts, the API and the tests you will extend.
2. Look things up with the app's docs command, which `AGENTS.md` describes, rather than by reading `node_modules`: `npx --no -- cdk-blocks docs <Name>...` (several names fit in one call).
3. Run it: `npm run dev` (local mocks, no AWS), `npm run test:e2e`, `npm run build`. A test run starts the app's dev server when none is running, calls it and stops it: when it passes, `npm run dev` works, with no need to start it by hand as well. A build or a test run takes 10 to 30 seconds: when one comes back still running, check on it with one call that waits 30 seconds (in Codex, `write_stdin` with `yield_time_ms: 30000`), not every second.

An app whose `@aws-blocks/blocks` has no `docs` command keeps the same docs in `node_modules/@aws-blocks/blocks/docs/` (`README.md` is the guide, one folder per block).

## Deploying and going to production

The app's `AGENTS.md` has the deploy commands, how long they take and what they print. Deploying needs AWS credentials; `npm run dev` does not.

Before a production deploy, read [references/security.md](references/security.md).
