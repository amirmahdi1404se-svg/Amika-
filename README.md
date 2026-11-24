# Amika AI

A cross-platform AI workspace starter with a React web client, Flutter Android app, and Cloudflare Workers API. The repository is structured for GitHub Actions APK builds.

## Repository layout
- `apps/web` — React + TypeScript web application.
- `apps/mobile` — Flutter Android application.
- `apps/api` — Cloudflare Worker API (authentication, chat, conversations, memory, research, image generation).
- `database/schema.sql` — Cloudflare D1 schema.
- `.github/workflows/android-apk.yml` — build APK on push or manual dispatch.

## 1. Configure Cloudflare API
Requirements: Node.js 20+, Cloudflare account, Wrangler.

```bash
cd apps/api
npm install
npx wrangler d1 create amika-db
```
Copy the returned `database_id` into `apps/api/wrangler.toml`, then:

```bash
npx wrangler d1 execute amika-db --remote --file ../../database/schema.sql
npx wrangler secret put OPENROUTER_API_KEY
npx wrangler secret put JWT_SECRET
# Optional integrations:
npx wrangler secret put TAVILY_API_KEY
npx wrangler secret put IMAGE_MODEL
npx wrangler deploy
```

Set `ALLOWED_ORIGINS` in `wrangler.toml` to a comma-separated list of exact production origins. For local development, copy `.dev.vars.example` to `.dev.vars` and fill it in. Never commit secrets.

## 2. Run web app
```bash
cd apps/web
npm install
cp .env.example .env
# Set VITE_API_BASE_URL to your Worker URL
npm run dev
```

## 3. Build Android APK with GitHub
1. Upload this repository to GitHub.
2. In GitHub → Settings → Secrets and variables → Actions, add repository secret `AMIKA_API_BASE_URL` with your deployed Worker URL (example `https://amika-api.your-subdomain.workers.dev`).
3. Open Actions → **Build Amika Android APK** → Run workflow, or push to `main`.
4. Download the `amika-ai-debug-apk` artifact from the completed workflow. For a public release, configure Android signing secrets and use a signed release build; do not distribute a debug APK as a production release.

## 4. Run Flutter locally
```bash
cd apps/mobile
flutter pub get
flutter run --dart-define=API_BASE_URL=https://YOUR-WORKER.workers.dev
```

## Implemented in this starter
- Email/password registration and login with PBKDF2 password hashing and signed session tokens.
- Authenticated conversation CRUD, message history, notebook notes, and user memory CRUD in D1.
- Multi-model OpenRouter chat with streamed responses and persistent chat history.
- Research endpoint using Tavily when configured.
- Image-generation endpoint using an OpenRouter image-capable model when configured.
- Web app with model picker, history, file text extraction (PDF/DOCX/TXT), voice input/output via browser APIs, research, image studio, and sandboxed HTML preview.
- Flutter app with login/register, chat, model picker, conversation history, file picker, speech input/output, and saved sessions.
- GitHub Actions workflow for Android APK artifacts.

## Important production checklist
Before opening the service publicly: set a strong random `JWT_SECRET`; restrict CORS; add Cloudflare Turnstile or equivalent signup protection; configure per-user rate limits and quotas; configure billing only if needed; review privacy/retention; set provider limits; add monitoring and backups; and test the selected image model. Research requires `TAVILY_API_KEY`; image generation requires an image-capable OpenRouter model and may depend on model/provider availability. Voice uses device/browser speech services and availability varies by platform. The GitHub workflow generates missing Android Gradle scaffolding before building.
