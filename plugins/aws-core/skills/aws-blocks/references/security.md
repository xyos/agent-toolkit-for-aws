# Before a production deploy

The list that matches the installed version is in the app: `npx --no -- cdk-blocks docs guide --section security`. Read it before the first production deploy. These points hold for every version:

- API methods are unauthenticated until they call `await auth.requireAuth(context)`. The local mock does not flag a method left public.
- Keep API keys and credentials in the `Secret` block (`npx --no -- cdk-blocks docs Secret`), never in source or in plaintext environment variables.
- Attach a schema to every store that accepts user data. The RPC layer checks the shape of a call, not the rules of your domain.
- Building Blocks grant least-privilege access to their own resources. Do not add `*` IAM policies, and do not change `blockPublicAccess` on a `FileBucket`: serve public files through CloudFront.
- List the allowed browser origins for production (`defaults.allowedOrigins` on the stack) instead of a wildcard.
- WAF and API Gateway throttling are not included by default. Add them with CDK for a public app.
- `Logger` does not redact. Never pass credentials, tokens or secrets to it.
- Deploy with least-privilege credentials.
