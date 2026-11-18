# Before a production deploy

Before the first production deploy, read the installed version's security checklist. In Blocks 0.9.0, read **Security Considerations** in `node_modules/@aws-blocks/blocks/docs/README.md`. If the app defines a Blocks docs task that supports sections, run it from the app root: `npm run docs -- guide --section security` (substitute the actual task name). To capture Markdown rather than JSON on a version with output selection, use `npm run -s docs -- guide --section security --output text`.

- API methods are unauthenticated until they call `await auth.requireAuth(context)`. The local mock does not flag a method left public.
- Keep API keys and credentials in the `Secret` block, never in source or plaintext environment variables. See `node_modules/@aws-blocks/blocks/docs/bb-secret/README.md`, or `npm run docs Secret` if the app defines that Blocks task.
- Attach a schema to every store that accepts user data. The RPC layer checks the shape of a call, not the rules of your domain.
- Building Blocks grant least-privilege access to their own resources. Do not add `*` IAM policies, and do not change `blockPublicAccess` on a `FileBucket`: serve public files through CloudFront.
- List the allowed browser origins for production (`defaults.allowedOrigins` on the stack) instead of a wildcard.
- WAF and API Gateway throttling are not included by default. Add them with CDK for a public app.
- `Logger` does not redact. Never pass credentials, tokens or secrets to it.
- Deploy with least-privilege credentials.
