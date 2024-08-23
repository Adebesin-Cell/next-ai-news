# Next AI News

A full-stack replica of HN using Next.js and AI-generated content.

## Design Notes

- **Framework**: Built with [Next.js 14](https://nextjs.org/) using the [App Router](https://nextjs.org/docs/app/building-your-application/routing) and [RSC](https://nextjs.org/docs/app/building-your-application/rendering/server-components) on the Node.js runtime.
  - **Rendering**: All pages are server-rendered and dynamic, with no data caching.
  - **Mutations**: Handled via [Server Actions](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations).
  - **Streaming**: Implemented [Streaming](https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming) for enhanced speed and concurrency.

- **Package Management**: Uses [pnpm](https://pnpm.io/installation) for managing packages.
- **Data Layer**: Utilizes [Drizzle ORM](https://orm.drizzle.team/docs/overview) and Zod for the data layer.
- **Authentication**: Implements [Next-Auth](https://next-auth.js.org/) from [Auth.js](https://authjs.dev/) for password authentication.
- **UI Generation**: Initial UIs generated with [Tailwind](https://tailwindcss.com/), [Shadcn UI](https://ui.shadcn.com/), and [Radix UI](https://www.radix-ui.com/) via [v0](https://v0.dev).
- **Development and Testing**: Entirely developed and tested using the new Next.js `--turbo` Rust compiler.
- **Search Highlights**: Uses [react-highlight-words](https://bvaughn.github.io/react-highlight-words/) for search highlights.
- **Pre-rendering**: [PPR](https://vercel.com/blog/partial-prerendering-with-next-js-creating-a-new-default-rendering-model) _(experimental)_ used to precompute page shells.
  - These pre-rendered shells are served statically from the edge.
  - Improves TTFB and speeds up CSS/fonts while origin streams.

- **Deployment**: Serverlessly deployed on Vercel's Edge Network with:
  - **AI Content Generation**: [Cron Jobs](https://vercel.com/guides/how-to-setup-cron-jobs-on-vercel) for AI content generation.
  - **SSR**: [Serverless Functions](https://vercel.com/docs/functions/serverless-functions) (Node.js) for SSR, deployed in `iad1` / `us-east-1`.
  - **Rate Limiting**: [KV](https://vercel.com/docs/storage/vercel-kv) (Upstash) for rate-limiting, deployed in `iad1` / `us-east-1`.
  - **Storage**: Core storage and search powered by [Postgres](https://vercel.com/docs/storage/vercel-postgres) (Neon) with [`pg_trgm`](https://www.postgresql.org/docs/current/pgtrgm.html), deployed in `iad1` / `us-east-1`.

### AI

- **LLM**: Uses [Mixtral](https://mistral.ai/) `mixtral-8x7b-32kseqlen` for AI-generated content.
- **Tools Support**: Finetuned with [Anyscale](https://www.anyscale.com/) for [Tools support](https://www.anyscale.com/blog/anyscale-endpoints-json-mode-and-function-calling-features).
- **Structured Generation**: Employs [openai-zod-functions](https://www.npmjs.com/package/openai-zod-functions) for structured and runtime-validated generation.

## Deployment

1. Ensure the Vercel project is connected to a Vercel Postgres (Neon) database.
2. Optionally, add a Vercel KV (Upstash) database for rate limiting.
3. Run `pnpm drizzle-kit push:pg`.
4. Update `metadataBase` in `app/layout.tsx` to match your target domain.

## Local Development

1. Run `vc env pull` to obtain a `.env.local` file with your database credentials.
2. Run `pnpm dev` to start developing.
3. For DB migrations with `drizzle-kit`:
   - Ensure `?sslmode=required` is added to the `POSTGRES_URL` env for development.
   - Run `pnpm drizzle-kit generate:pg` to generate migrations.
   - Run `pnpm drizzle-kit push:pg` to apply them.

## Performance

[PageSpeed report](https://pagespeed.web.dev/analysis/https-next-ai-news-vercel-app/x55es0m0ya?form_factor=mobile) for Emulated Moto G Power with Lighthouse 11.0.0, Slow 4G Throttling:

[![PageSpeed Score](https://h2rsi9anqnqbkvkf.public.blob.vercel-storage.com/perf-LAbwq5HsiimvbRrNSUV9JAGCATsBMs.png)](https://pagespeed.web.dev/analysis/https-next-ai-news-vercel-app/x55es0m0ya?form_factor=mobile)

<sup>💩 The SEO `98` score cannot be `100` without sacrificing stylistic fidelity to the original HN navigation</sup>

## Codebase Notes

- **Auth**: Initialized in `app/auth.tsx`.
- **Database**: Initialized in `app/db.tsx`.
- **Components**: Shared components are located in `./components` (exposed as `@/components`).
- **Custom Components**: Only one component was not reused from npm / shadcn ([`components/time-ago.tsx`](components/time-ago.tsx)).
  - This component takes a `now` prop with a timestamp and works well with server rendering.
- **Manual DB Migrations**:
  - `CREATE EXTENSION IF NOT EXISTS pg_trgm;` as part of #13.
  - `USING GIN (title gin_trgm_ops);` as part of #13.

## TODO

This project replicates HN with numerous features. However, there are some important gaps that the community could help fill:

- [ ] Inline comment replies
- [ ] "Forgot password" flow
- [ ] Voting and ranking
- [ ] "Next" and "Prev" comment links
- [ ] Comment toggling
- [ ] Flagging submissions and comments
- [ ] Improved search functionality
- [ ] Comment pagination
- [ ] More efficient comment data structures
- [ ] Optimistic comments with `useOptimistic`
- [ ] Local storage for comment and submission drafts
- [ ] Improved `/next` implementation after login
- [ ] Support for passkeys
- [ ] Basic admin panel
- [ ] User profiles

## License

MIT
