# Adding CDK Blocks to an existing project

The scaffolder detects the project by what is in the directory. Run it from the project root.

## A project with a `package.json` (Vite, Next.js, SvelteKit, Astro, an API, ...)

```bash
npx --yes @aws-blocks/create-blocks-app . --template <template> --yes
```

| The project | `--template` | The Blocks dev server serves by default |
|---|---|---|
| has a frontend (Vite, SvelteKit, Astro, ...) | `bare` | the API and the project's own frontend at http://localhost:3000 |
| is a Next.js app | `nextjs` | the API and the Next.js app at http://localhost:3000 |
| has no frontend (an Express API, a worker, a CLI) | `backend` | the API alone at http://localhost:3001 |

- `--yes` is required. Without it the command prints `Proceed? (y/N)` and, in a shell without a terminal, exits with status 0 having changed nothing.
- Without `--template` you get the `default` template's sample backend (sign-in, a todo model, realtime), which you would have to delete. With the three above, `aws-blocks/index.ts` holds only a `greet()` sample (`nextjs`: two sample methods) to replace.
- Wait for the scaffold command to exit successfully. npm may print install output; `✓ AWS Blocks backend added to your project!` is followed by next steps.

What it adds:

| Change | What it is |
|---|---|
| `aws-blocks/` | The backend workspace. `aws-blocks/index.ts` defines the API |
| `cdk.json` | CDK configuration |
| `package.json` | The `aws-blocks` workspace, `@aws-blocks/blocks`, and scripts to run, stop and deploy the backend. If your code imports `zod`, add it as a direct dependency if missing (`npm install zod` with npm); the 0.4.0 overlay does not add it, even with `bare`. |
| `.gitignore` | Entries for local data, generated files and secrets |

The overlay preserves host scripts and adds no tests. Before running a task, check the actual Blocks script in the app-root `package.json`: a name like `dev` or `docs` may still belong to the host. Scaffolders that support aliases print the chosen names (for example, `blocks:docs` on a collision). Run those names with the project's package manager.

## Working in it

The overlay does not add an `AGENTS.md`. Follow any project instructions already present. For the Blocks backend:

- **API.** The backend exports namespaces: `export const api = new ApiNamespace(scope, 'api', (context) => ({ async greet(name: string) { ... } }))`. Clients call `<export name>.<method>()`, so a request for `orders.create()` means `export const orders = new ApiNamespace(scope, 'orders', ...)`.
- **Clients.** Frontend code imports the backend with its types: `import { api } from 'aws-blocks'`, then `await api.greet('x')`. There is no client to generate and no URL to write. A Node script or test does the same while the dev server runs: a file in the project root such as `check.mts` (`.mts`, so that top-level `await` works) with `const { api } = await import('aws-blocks')`, run with `npx tsx check.mts`.
- **Errors.** `throw new ApiError('Order not found', 404, { name: 'OrderNotFound' })`, with `ApiError` from `@aws-blocks/blocks`, reaches the client with its status, message and name. Anything else thrown reaches it as an unnamed 500 `Internal error`. In the client the call throws an `ApiError` (`e.status`, `e.name`, `e.message`); client code imports `ApiError` and `isBlocksError` from `@aws-blocks/blocks/client` and matches the name: `isBlocksError(e, 'OrderNotFound')`.
- **Running.** From the app root, run the Blocks dev-server task (for example, `npm run dev:server` if defined for Blocks). It runs local mocks and, for frontend templates, proxies the frontend dev server. It stays in the foreground and streams logs. Wait until `http://localhost:<port>/.blocks-sandbox/config.json` answers. To stop it, use the corresponding stop task (for example, `npm run dev:stop`), not a process-name kill.
- **Vite 502.** Check the frontend log first. If Vite is listening on `[::1]` but the proxy connects to `127.0.0.1`, add `--host 127.0.0.1` to the existing Vite command. The 0.4.0 `bare` overlay sets it in `aws-blocks/scripts/server.ts`; for versions configured through `package.json`, use `blocks.dev.frontendCommand` there.
- **Calling it by hand.** One URL serves every namespace, as JSON-RPC: `POST /aws-blocks/api`. The namespace goes in `method`, never in the path: `orders.create()` is `"method":"orders.create"` at the same URL, and `/aws-blocks/orders` does not exist.

  ```bash
  curl -s -X POST http://localhost:<port>/aws-blocks/api -H 'Content-Type: application/json' \
    -d '{"jsonrpc":"2.0","method":"orders.create","params":[{"item":"x"}],"id":1}'
  ```

  The default answer is HTTP 200 with a `result`, or with an `error` such as `{"code":404,"message":"Order not found","data":{"name":"OrderNotFound"}}`. For `ApiNamespace` options, read `node_modules/@aws-blocks/blocks/docs/core/README.md`. Blocks 0.9.0 does not support `httpErrorStatus`; use it only if the installed version documents it.
- **Data.** Store data with Building Blocks, never in local files or memory. Use the [installed docs](../SKILL.md#look-up-blocks-docs) for the block catalog, guide and API lookups.
- **Sign-in.** Every API method is public until it calls `await auth.requireAuth(context)`.

If the project typechecks with `tsc`, follow the note the scaffolder prints about `aws-blocks/tsconfig.json`.

## An Amplify Gen 2 project (`amplify/backend.ts`)

```bash
npx --yes @aws-blocks/create-blocks-app . --yes
```

The scaffolder adds `aws-blocks/` and `amplify/blocks.ts`, imports the latter from `amplify/backend.ts`, and updates `amplify.yml`. Amplify keeps its auth and data; the Blocks API is served beside them. Use the Blocks scripts in the app's `package.json` for local development, stopping the server and client generation (normally `blocks:dev`, `blocks:stop` and `blocks:generate-client`). For deployment, use the integrated Amplify sandbox task named in the next steps after confirming its script, not a standalone Blocks sandbox task.
