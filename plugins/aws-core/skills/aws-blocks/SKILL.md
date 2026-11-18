---
name: aws-blocks
description: Sets up CDK Blocks (formerly AWS Blocks) apps - scaffolds a new app, picks a starter template, or adds a Blocks backend to an existing project. Use only when the request names CDK Blocks, AWS Blocks, cdk-blocks or the @aws-blocks packages.
---

# CDK Blocks (formerly AWS Blocks)

CDK Blocks is a TypeScript framework for full-stack apps on AWS: the backend lives in one `aws-blocks/` folder built from Building Blocks (`KVStore`, `DistributedTable`, `Auth`, `FileBucket`, ...), runs locally without an AWS account and deploys with the CDK.

The packages are published as `@aws-blocks/*`. Run app tasks from the directory containing the app's `package.json`: use `<package manager> run <task>`, with the project's package manager and a task defined in `scripts`. Examples below use npm.

## Where to go

| The project | Do this |
|---|---|
| already has an `aws-blocks/` folder | Follow the app's `AGENTS.md` if present. For feature work, use the [docs for the installed version](#look-up-blocks-docs); don't scaffold again. |
| is an empty directory (a new app) | [Create an app](#create-an-app) |
| exists without Blocks (Vite, Next.js, an API, Amplify Gen 2, ...) | [references/existing-project.md](references/existing-project.md) |

## Create an app

One command scaffolds the app and installs its dependencies. With `--yes` it never prompts:

```bash
npx --yes @aws-blocks/create-blocks-app <directory> --template <template> --yes
```

`<directory>` is `.` for the current directory, which must be empty (`.git` and editor settings are fine).

Wait for the scaffolder to exit successfully before running app tasks. npm may print progress during `Installing dependencies...`; the scaffolder prints `✓ Blocks app created!` followed by next steps. If the install is still running, you can read the generated files while you wait.

Choose a template listed by the scaffolder you are using (`npx --yes @aws-blocks/create-blocks-app --help`). These are fresh-app choices in `@aws-blocks/create-blocks-app` 0.4.0:

| Template | What you get |
|---|---|
| `bare` | Vite frontend and one `greet()` API method. The smallest app; start here when the features are yours to write |
| `backend` | Backend only (API and CDK app), no frontend |
| `api-only` | Backend only, with a public health check and sign-in-gated CRUD as a sample, for mobile, CLI or third-party clients |
| `react` | Vite + React 19 todo app with sign-in, `DistributedTable` and realtime |
| `nextjs` | Next.js 16 App Router; Server and Client Components call the backend |
| `sql` | PostgreSQL backend (`Database`: migrations, foreign keys, transactions) |
| `auth-cognito` | Vite frontend with Cognito passwordless email sign-in |
| `demo` | Larger sample with sign-in, `KVStore` and a priority-sorted todo list |
| `default` | Used without `--template`: Vite + lit-html with sign-in, `DistributedTable` and realtime |

Use `auth`, `agent`, `cron` or `observability` only if the scaffolder lists it; 0.4.0 does not. If your code imports `zod`, check `package.json` and add it as a direct dependency if missing (`npm install zod` with npm).

## After scaffolding

1. Read the generated `AGENTS.md` in full before writing code. Then read `package.json`, `aws-blocks/index.ts` and `test/e2e.test.ts` for the tasks, API and tests you will extend.
2. For a local check, run the template's test and build tasks if defined (typically `npm run test:e2e` and `npm run build`). The scaffolded e2e test starts a local server if needed. Start the app's `dev` task separately only if you need a running server.

## Look up Blocks docs

With `@aws-blocks/blocks` 0.9.0, or whenever the app has no Blocks docs task, read `node_modules/@aws-blocks/blocks/docs/README.md` for the guide and catalog. Follow it to `docs/<block-folder>/README.md` or `API.md` in the same package. No CLI installation is needed.

If the installed version supports docs tasks, check their names in the app's `package.json`. Use the examples below only for the corresponding Blocks scripts; an existing project can assign a different name such as `blocks:docs` to preserve its own `docs` script.

```bash
npm run docs Auth.protect
npm run docs:api KVStore
npm run docs:search "partition key"
npm run docs -- KVStore --section query
```

Use positional names and search terms. With npm, put Blocks flags after `--`. For versions that select output by stdout, structured tasks (including docs) produce text in a terminal and JSON when piped or captured:

```bash
npm run -s docs Auth.protect | jq '{name, kind}'
npm run -s docs -- KVStore --output text
```

`-s` suppresses npm's banner, not Blocks output. The second command requests Markdown even when an agent captures stdout. Streaming tasks such as `dev` write logs, not a JSON document.

## Deploying and going to production

For a deployment, follow the installed app's `AGENTS.md` and confirm the task in `package.json`. For example, `npm run sandbox:deploy` performs a one-shot sandbox deployment in apps that define that task. Deployment needs AWS credentials; local development does not.

Before a production deploy, read [references/security.md](references/security.md).
