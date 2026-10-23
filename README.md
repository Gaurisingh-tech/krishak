# KRISHAK

KRISHAK is a browser-based agricultural decision-support prototype for farmers in India. It combines a farm profile, starter crop notes, live weather, and an AI provider to generate recommendations. It is a hackathon/project prototype, not an official agricultural advisory.

## Features

- Responsive farm dashboard and editable farm profile.
- English and Hindi interface and AI responses.
- Agricultural Q&A using local Ollama, Google Gemini, or Anthropic.
- Weather lookup and forecasts from Open-Meteo.
- Optional wholesale mandi prices from Agmarknet through data.gov.in.
- Browser speech input/output and crop-photo questions (vision-capable provider required).
- Farm profile and language preference stored in the browser's local storage.

These capabilities describe functionality present in the code; this review has not validated browser compatibility, AI-provider responses, or live external-service behavior end to end.

The starter crop notes are in [`server/knowledge.json`](server/knowledge.json). The server returns recommendations in a structured format and includes context/source information where available.

## Requirements

- Node.js 20.6 or later (the server uses Node's `--env-file` support).
- A supported browser with internet access.
- An AI provider to receive AI recommendations:
  - Ollama: install Ollama and pull `qwen2.5:3b` (no API key).
  - Gemini: `GEMINI_API_KEY`
  - Anthropic: `ANTHROPIC_API_KEY`
- Optional: a data.gov.in API key for mandi prices (`DATA_GOV_API_KEY`).

The server uses Node.js built-in modules and declares no npm package dependencies. AI, weather, and optional mandi data require network access. No API key is required for Open-Meteo weather.

## Setup on macOS

From Terminal, go to the project folder, then create a local environment file from the template:

```sh
cd /path/to/KRISHAK-Farmer
cp server/.env.example server/.env
```

Open `server/.env` in a text editor. For the local Ollama model, select:

```dotenv
AI_PROVIDER=ollama
OLLAMA_MODEL=qwen2.5:3b
```

Install Ollama if needed, then download the model once:

```sh
ollama pull qwen2.5:3b
```

Keep Ollama running while using KRISHAK. The server calls Ollama only at `http://127.0.0.1:11434`; do not expose Ollama's API port on a public interface. Ollama selection does not require Gemini or Anthropic keys.

To use a cloud provider instead, set `AI_PROVIDER=gemini` and `GEMINI_API_KEY`, or `AI_PROVIDER=anthropic` and `ANTHROPIC_API_KEY`. When `AI_PROVIDER` is omitted, the existing key-based selection remains: Anthropic is selected first if its key is present, otherwise Gemini. For mandi prices, optionally set `DATA_GOV_API_KEY`. Keep real keys only in `server/.env`; never put them in source files or commit them. The `.env.example` file is a placeholder template and should not contain credentials.

## Run

```sh
cd /path/to/KRISHAK-Farmer/server
npm start
```

Open <http://localhost:8787> in your browser. Keep the terminal running while using the app; press **Ctrl+C** to stop the server. Do not open `index.html` directly when using API-backed features.

For Ollama, ensure the Ollama app/service is running and the configured model is installed. The server reports a clear error if Ollama is unreachable, the model is missing, a request times out, or the response is invalid. The `qwen2.5:3b` model is text-only: crop-photo analysis is unavailable with Ollama. Photo questions remain supported when using a vision-capable Gemini or Anthropic model. Mandi prices remain unavailable unless a valid optional data.gov.in key is configured. `PORT` can be set in `server/.env` to choose a different KRISHAK server port.

## Deploy to Vercel

The project root contains `index.html`, `vercel.json`, and the `api/` serverless entry point. Import the repository into Vercel with the project root as the Root Directory. No frontend build command or output directory is needed; Vercel serves `index.html` as a static file and routes `/api/*` to the Node.js function.

In the Vercel project settings, add environment variables for a hosted AI provider, for example `AI_PROVIDER=gemini` and `GEMINI_API_KEY`, or `AI_PROVIDER=anthropic` and `ANTHROPIC_API_KEY`. Add `DATA_GOV_API_KEY` if live mandi prices are required. Never add secrets to source files or commit `server/.env`; enter deployment secrets only in Vercel's environment-variable settings. Redeploy after changing environment variables.

Ollama remains supported for local development with `AI_PROVIDER=ollama` and `OLLAMA_MODEL=qwen2.5:3b`. A Vercel function cannot reach Ollama running on your Mac via `127.0.0.1`, so do not select Ollama for a Vercel deployment. Do not expose Ollama's local API port publicly to work around this limitation. Vercel function execution and request-size limits also apply; the configured function timeout is 60 seconds.

## Environment variables

| Variable | Required | Purpose |
| --- | --- | --- |
| `AI_PROVIDER` | No | Selects `ollama`, `gemini`, or `anthropic`. If omitted, existing API-key-based selection applies. |
| `OLLAMA_MODEL` | No | Ollama model name; defaults to `qwen2.5:3b`. Ollama is accessed only at `127.0.0.1:11434`. |
| `GEMINI_API_KEY` | When Gemini selected | Google Gemini API key. |
| `GEMINI_MODEL` | No | Optional Gemini model override. |
| `ANTHROPIC_API_KEY` | When Anthropic selected | Anthropic API key. If `AI_PROVIDER` is omitted and both keys are set, the server selects Anthropic. |
| `ANTHROPIC_MODEL` | No | Anthropic model override; the template provides a default model name. |
| `DATA_GOV_API_KEY` | No | Enables mandi-price requests through data.gov.in. |
| `PORT` | No | Local HTTP server port; defaults to `8787`. |

## Data handling and privacy

- The farm profile and selected language are saved in the current browser's local storage. The prefilled profile is fictional demo data, as confirmed by the project owner.
- When asking a question, the browser sends the profile, question, and selected language to the same-origin KRISHAK `/api` endpoint. If a photo is attached, it is sent to the configured AI provider for analysis. With Ollama, photo requests are rejected because `qwen2.5:3b` is text-only.
- The server requests weather using the entered location from Open-Meteo. Mandi requests send the selected state, crop, and district to data.gov.in when that feature is enabled.
- Review the relevant service and AI-provider terms and privacy policies before sending personal or sensitive information. Do not use real farmer information in a public demo unless you have appropriate permission.
- The app is intended for local use and is not documented or hardened as a public multi-user service.

## Limitations and safety

- AI-generated recommendations can be inaccurate or incomplete. Verify consequential decisions with a local Krishi Vigyan Kendra (KVK), agriculture officer, product label, or other qualified source.
- A crop photo cannot establish a certain diagnosis. Image quality, crop/model support, and internet availability affect results.
- The crop knowledge base contains general starter notes, not an official or comprehensive advisory. Check regional and crop-specific guidance.
- Weather and mandi information can be unavailable, delayed, or approximate. Mandi prices are wholesale reports and may not match the price a farmer receives; they are not MSP.
- Speech recognition and playback depend on browser capabilities and installed voices.

## Testing and validation

There is currently no automated test suite or test script in `server/package.json`. To check the server JavaScript syntax with Node.js:

```sh
node --check server/server.js
```

For a local Ollama smoke test, start Ollama, confirm the model is installed, start the KRISHAK server, and check its health endpoint:

```sh
ollama list
curl http://localhost:8787/api/health
```

The health response should show `"aiConfigured":true`, `"aiCheck":{"ok":true,...}`, and model `qwen2.5:3b`. Then submit a text question in the web app. Also attach a test photo to confirm the app reports that this text-only model cannot analyze images. If Ollama is stopped while KRISHAK remains running, its health check and recommendation requests should report Ollama as unavailable; selecting an uninstalled model should report that the model is missing. Do not expose Ollama's `11434` API port to the network.

For deployment, use Vercel's preview deployment to verify `/api/health`, text recommendations, and any configured mandi requests before promoting to production. Configure provider secrets in Vercel, not in the repository.

The syntax check and health request do not verify every recommendation or external service. Gemini and Anthropic can be tested separately with their respective credentials. Weather and optional mandi data still rely on their existing external services; avoid real personal data during manual testing.

## Contributing

Contributions should explain the problem being addressed and include steps to verify the change. Keep credentials, private data, and generated logs out of commits; use the placeholder `server/.env.example` for configuration documentation. Review agricultural claims and cite reliable sources when updating crop guidance.

The project code is licensed under the MIT License; see [`LICENSE`](LICENSE). This does not replace or override terms that apply to third-party services, fonts, datasets, or content. Review the provenance and reuse rights of all such materials before redistributing them or contributing changes.

## Third-party services and materials

- Google Fonts are loaded from Google Fonts at runtime; review applicable font licenses and attribution.
- Weather data is requested from Open-Meteo.
- Optional mandi data is requested from Agmarknet through data.gov.in.
- The origin and redistribution terms for the agricultural text in `server/knowledge.json` have not been established in the repository.

Review current service and data terms, attribution requirements, and the source/license of all included content before public distribution.
