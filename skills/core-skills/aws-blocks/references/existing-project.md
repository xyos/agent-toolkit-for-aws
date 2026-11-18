# Adding CDK Blocks to an existing project

The scaffolder detects the project by what is in the directory. Run it from the project root.

## A project with a `package.json` (Vite, Next.js, SvelteKit, Astro, an API, ...)

```bash
npx --yes @aws-blocks/create-blocks-app . --template <template> --yes
```

| The project | `--template` | `npm run dev:server` then serves |
|---|---|---|
| has a frontend (Vite, SvelteKit, Astro, ...) | `bare` | the API and the project's own frontend at http://localhost:3000 |
| is a Next.js app | `nextjs` | the API and the Next.js app at http://localhost:3000 |
| has no frontend (an Express API, a worker, a CLI) | `backend` | the API alone at http://localhost:3001 |

- `--yes` is required. Without it the command prints `Proceed? (y/N)` and, in a shell without a terminal, exits with status 0 having changed nothing.
- Without `--template` you get the `default` template's sample backend (sign-in, a todo model, realtime), which you would have to delete. With the three above, `aws-blocks/index.ts` holds only a `greet()` sample (`nextjs`: two sample methods) to replace.
- It prints nothing while it installs (30 to 90 seconds) and ends with `✓ AWS Blocks backend added to your project!`. If your shell hands the command back still running, read the project's files, then wait as for a new app: every check on the install is one call that waits 60 seconds (in Codex, an empty `write_stdin` with `yield_time_ms: 60000`), never a short poll. `npx --no -- cdk-blocks` works only after that line.

What it adds:

| Change | What it is |
|---|---|
| `aws-blocks/` | The backend workspace. `aws-blocks/index.ts` defines the API |
| `cdk.json` | CDK configuration |
| `package.json` | The `aws-blocks` workspace, `@aws-blocks/blocks`, and the scripts `dev:server`, `dev:stop`, `sandbox`, `sandbox:destroy`, `deploy`, `destroy`. `bare` also adds `zod`; with the others, run `npm install zod` if your code imports it |
| `.gitignore` | Entries for local data, generated files and secrets |

It does not change the project's own source or its `dev` and `build` scripts, and it adds no tests.

## Working in it

The scaffolder writes no `AGENTS.md` into an existing project, so there is none to look for. What a new app's would tell you:

- **API.** The backend exports namespaces: `export const api = new ApiNamespace(scope, 'api', (context) => ({ async greet(name: string) { ... } }))`. Clients call `<export name>.<method>()`, so a request for `orders.create()` means `export const orders = new ApiNamespace(scope, 'orders', ...)`.
- **Clients.** Frontend code imports the backend with its types: `import { api } from 'aws-blocks'`, then `await api.greet('x')`. There is no client to generate and no URL to write. A Node script or test does the same while the dev server runs: a file in the project root such as `check.mts` (`.mts`, so that top-level `await` works) with `const { api } = await import('aws-blocks')`, run with `npx tsx check.mts`.
- **Errors.** `throw new ApiError('Order not found', 404, { name: 'OrderNotFound' })`, with `ApiError` from `@aws-blocks/blocks`, reaches the client with its status, message and name. Anything else thrown reaches it as an unnamed 500 `Internal error`. In the client the call throws an `ApiError` (`e.status`, `e.name`, `e.message`); client code imports `ApiError` and `isBlocksError` from `@aws-blocks/blocks/client` and matches the name: `isBlocksError(e, 'OrderNotFound')`.
- **Running.** `npm run dev:server` runs the backend with local mocks and no AWS account, and proxies the project's frontend dev server, which it starts itself. It stays in the foreground: start it in the background with its output in a file (`npm run dev:server > dev.log 2>&1 &`), wait until `http://localhost:<port>/.blocks-sandbox/config.json` answers, and stop it with `npm run dev:stop`. If `/` still answers `502` ten seconds later in a Vite project, Vite is listening on `[::1]` while the proxy connects to `127.0.0.1`: set `"blocks": { "dev": { "frontendCommand": "npx vite --host 127.0.0.1 --port 3100 --strictPort" } }` in `package.json`. The project's own scripts are unchanged. To check that its own `dev` script still starts, give it a time limit, `timeout 10 npm run dev`: it prints its start line, ends by itself (exit 124) and leaves nothing running. Never stop a server with `pkill -f` or `killall node`: they match your own shell too.
- **Calling it by hand.** One URL serves every namespace, as JSON-RPC: `POST /aws-blocks/api`. The namespace goes in `method`, never in the path: `orders.create()` is `"method":"orders.create"` at the same URL, and `/aws-blocks/orders` does not exist.

  ```bash
  curl -s -X POST http://localhost:<port>/aws-blocks/api -H 'Content-Type: application/json' \
    -d '{"jsonrpc":"2.0","method":"orders.create","params":[{"item":"x"}],"id":1}'
  ```

  The answer is HTTP 200 with a `result`, or with an `error` such as `{"code":404,"message":"Order not found","data":{"name":"OrderNotFound"}}`. For a caller that reads status codes, `new ApiNamespace(scope, 'orders', (context) => ({ ... }), { httpErrorStatus: true })` answers a 4xx `ApiError` with that HTTP status; the body is the same and the typed client behaves the same (`npx --no -- cdk-blocks docs httpErrorStatus` prints the option).
- **Data.** Store data with Building Blocks, never in local files or memory. `npx --no -- cdk-blocks docs` lists them, `npx --no -- cdk-blocks docs <Name>...` prints a block or one of its exports, options or methods (`docs DistributedTable`, `docs KVStore.get`), `--section <topic>` prints one part of a long doc, and `npx --no -- cdk-blocks docs guide` prints the framework guide (sign-in, choosing a data block, testing, security).
- **Sign-in.** Every API method is public until it calls `await auth.requireAuth(context)`.

If the project typechecks with `tsc`, follow the note the scaffolder prints about `aws-blocks/tsconfig.json`.

## An Amplify Gen 2 project (`amplify/backend.ts`)

```bash
npx --yes @aws-blocks/create-blocks-app . --yes
```

The scaffolder wires Blocks into the Amplify backend: it adds `aws-blocks/` and `amplify/blocks.ts`, imports that file from `amplify/backend.ts`, adds `amplify.yml` settings and the scripts `blocks:dev` (the Blocks backend locally), `blocks:stop` and `blocks:generate-client`. Amplify keeps its auth and data; the Blocks API is served beside them. `npm run sandbox` stays Amplify's sandbox and now deploys both. Read what the scaffolder prints for the next steps.
