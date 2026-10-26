# AI Pitch Deck

Turn a startup idea into a visual pitch deck. AI Pitch Deck is a full-stack application that combines AI-generated slide content, custom illustrations, and background workflows in a clean web experience.

Describe your business, start generation, and browse the finished presentation in an interactive slide viewer. Decks and their slides are stored in PostgreSQL, with images hosted on ImageKit.

## Features

- **Idea-to-deck generation** — Create a structured business presentation from a project description.
- **Investor-focused storytelling** — Generate slides covering the title, problem, solution, market, product, business model, and funding ask.
- **Custom illustrations** — Generate an image for each slide and upload it to ImageKit.
- **Structured AI output** — Validate deck titles, slide content, and illustration prompts with Zod schemas.
- **Content guardrails** — Validate idea length and review generated content for quality and suitability.
- **Background processing** — Run AI generation, image uploads, and database persistence through named Inngest steps.
- **Generation status** — Track pending, generating, complete, and failed states with automatic status polling.
- **Deck library** — Browse saved presentations with titles, descriptions, slide counts, and creation dates.
- **Interactive viewer** — Navigate illustrated slides with carousel controls and a slide counter.
- **Responsive interface** — Built with Tailwind CSS, shadcn/ui, Base UI, and Lucide icons.

## Technology Stack

| Layer | Technology |
| --- | --- |
| Application | Next.js 16 App Router, React 19, TypeScript |
| Styling and components | Tailwind CSS 4, shadcn/ui, Base UI |
| AI orchestration | OpenAI Agents SDK |
| Slide writing and review | `gpt-4.1-mini` |
| Image generation | `gpt-image-1-mini`, 1024 × 1024 images |
| Background workflows | Inngest |
| Database | PostgreSQL, Prisma ORM 7, PostgreSQL driver adapter |
| Image storage and delivery | ImageKit |
| Validation | Zod |
| Package manager | pnpm 10.28.2 |

## How It Works

1. Submit a project idea containing at least 20 characters.
2. The application stores a pending deck and sends a `deck/generate` event to Inngest.
3. The workflow marks the deck as generating and runs the pitch deck agent.
4. The agent writes the slides, validates the structured output, and applies content quality guardrails.
5. For each slide, the workflow generates an illustration, uploads it to ImageKit, and saves the slide to PostgreSQL.
6. The deck is marked complete and becomes available in the slide viewer.

The agent targets six to seven slides. The viewer checks for updates every three seconds while generation is in progress.

## Architecture

See [ARCHITECTURE.md](ARCHITECTURE.md) for the frontend and backend architecture, system diagrams, generation workflow, data model, and deployment boundaries.

## Getting Started

### Prerequisites

- Node.js 22.12+ within the Node.js 22 release line, or Node.js 24+.
- pnpm 10.28.2.
- A PostgreSQL database.
- An OpenAI API key with access to the configured text and image models.
- An ImageKit account and private API key.

Run the following commands from the project root.

### 1. Install dependencies

```bash
pnpm install
```

### 2. Configure environment variables

Create a `.env` file in the project root:

```dotenv
DATABASE_URL="postgresql://USER:PASSWORD@HOST:5432/DATABASE"
OPENAI_API_KEY="your-openai-api-key"
IMAGEKIT_PRIVATE_KEY="your-imagekit-private-key"
USE_PLACEHOLDER_IMAGES=false
```

| Variable | Purpose |
| --- | --- |
| `DATABASE_URL` | PostgreSQL connection string used by Prisma and the application. |
| `OPENAI_API_KEY` | Authenticates slide writing, content review, and image generation. |
| `IMAGEKIT_PRIVATE_KEY` | Authenticates image uploads to the `/pitch-decks` folder. |
| `USE_PLACEHOLDER_IMAGES` | Optional development setting. Use `true` for stock images in place of AI illustrations; defaults to AI generation when omitted. |
| `INNGEST_EVENT_KEY` | Event publishing key for the deployed Inngest environment. |
| `INNGEST_SIGNING_KEY` | Signing key for verifying requests from Inngest in a deployed environment. |

Stock image mode still uses OpenAI for slide content and quality review, and ImageKit for uploads. Keep environment values on the server; `.env` files are excluded from version control.

### 3. Initialize the database

Generate the Prisma client and apply the included migrations:

```bash
pnpm exec prisma generate --config prisma7.config.ts
pnpm exec prisma migrate deploy --config prisma7.config.ts
```

The project uses `prisma7.config.ts` for Prisma configuration and generates its client into `lib/generated/prisma`.

### 4. Start the application

```bash
pnpm dev
```

In a second terminal, start the Inngest development server:

```bash
npx inngest-cli@latest dev -u http://localhost:3000/api/inngest
```

Open the application at [localhost:3000](http://localhost:3000). The Inngest development dashboard is available at [localhost:8288](http://localhost:8288).

## Using the Application

1. Open the home page and describe your startup, audience, problem, and proposed solution.
2. Select **Generate Pitch Deck** to start the background workflow.
3. Follow the generation status on the deck page.
4. Browse the finished slides using the carousel controls.
5. Visit **My Decks** to return to saved presentations.

Example idea:

> A B2B SaaS platform that uses AI to automate invoice reconciliation for mid-size companies, helping finance teams match payments, identify discrepancies, and close their books faster.

## Project Structure

```text
app/
  page.tsx                 Idea submission page
  decks/                   Deck library and presentation pages
  api/decks/               Deck request handlers
  api/inngest/             Inngest workflow endpoint
components/
  create-deck-form.tsx      Idea input and submission
  deck-viewer.tsx           Status polling and slide carousel
  deck-status-badge.tsx     Generation status display
  site-header.tsx           Application navigation
  ui/                      Shared interface components
inngest/
  client.ts                Inngest client
  functions/
    generate-deck.ts       Deck generation workflow
lib/
  agents/                  Pitch deck agent and guardrails
  schemas/                 Structured AI output schemas
  types/                   Deck and slide types
  db.ts                    Prisma database client
  openai.ts                Slide image generation
  imageKit.ts              Image uploads
prisma/
  schema.prisma            Deck and Slide models
  migrations/              Database migration history
prisma7.config.ts          Prisma CLI configuration
```

## Available Commands

| Command | Description |
| --- | --- |
| `pnpm dev` | Start the development server. |
| `pnpm build` | Create the production build. |
| `pnpm start` | Serve the production build. |
| `pnpm lint` | Run ESLint. |
| `pnpm exec prisma generate --config prisma7.config.ts` | Generate the Prisma client. |
| `pnpm exec prisma migrate deploy --config prisma7.config.ts` | Apply committed database migrations. |
| `npx inngest-cli@latest dev -u http://localhost:3000/api/inngest` | Start the local workflow server. |

## Production Deployment

Deploy the application to a hosting environment that supports Next.js server execution, alongside PostgreSQL, ImageKit, and an Inngest environment.

Configure `DATABASE_URL`, `OPENAI_API_KEY`, `IMAGEKIT_PRIVATE_KEY`, `INNGEST_EVENT_KEY`, and `INNGEST_SIGNING_KEY` in the hosting environment. Set `USE_PLACEHOLDER_IMAGES=false` to generate custom illustrations.

Generate the database client before building:

```bash
pnpm install --frozen-lockfile
pnpm exec prisma generate --config prisma7.config.ts
pnpm build
```

Apply the committed migrations as part of the release process:

```bash
pnpm exec prisma migrate deploy --config prisma7.config.ts
```

For a Node.js server deployment, start the application with:

```bash
pnpm start
```

Connect the deployed `/api/inngest` endpoint to the matching Inngest environment so it can register and execute the `deck/generate` workflow.
