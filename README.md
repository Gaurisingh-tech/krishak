# KRISHAK

KRISHAK is a browser-based agricultural decision-support prototype for farmers in India. It combines a farm profile, starter crop notes, live weather, and an AI provider to generate recommendations. It is a hackathon/project prototype, not an official agricultural advisory.

## Features

- Responsive farm dashboard and editable farm profile.
- English and Hindi interface and AI responses.
- Agricultural Q&A using either Google Gemini or Anthropic.
- Weather lookup and forecasts from Open-Meteo.
- Optional wholesale mandi prices from Agmarknet through data.gov.in.
- Browser speech input/output and crop-photo questions (browser and model support vary).
- Farm profile and language preference stored in the browser's local storage.

These capabilities describe functionality present in the code; this review has not validated browser compatibility, AI-provider responses, or live external-service behavior end to end.

The starter crop notes are in [`server/knowledge.json`](server/knowledge.json). The server returns recommendations in a structured format and includes context/source information where available.

## Requirements

- Node.js 20.6 or later (the server uses Node's `--env-file` support).
- A supported browser with internet access.
- An API key for one AI provider to receive AI recommendations:
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

Open `server/.env` in a text editor. Set **one** AI provider key, for example:

```dotenv
GEMINI_API_KEY=your_gemini_key
```

Or use `ANTHROPIC_API_KEY` instead. For mandi prices, optionally set `DATA_GOV_API_KEY`. Keep real keys only in `server/.env`; never put them in source files or commit them. The `.env.example` file is a placeholder template and should not contain credentials.

## Run

```sh
cd /path/to/KRISHAK-Farmer/server
npm start
```

Open <http://localhost:8787> in your browser. Keep the terminal running while using the app; press **Ctrl+C** to stop the server. Do not open `index.html` directly when using API-backed features.

If the AI key is missing or rejected, AI recommendations will not work. Mandi prices remain unavailable unless a valid optional data.gov.in key is configured. `PORT` can be set in `server/.env` to choose a different local port.

## Environment variables

| Variable | Required | Purpose |
| --- | --- | --- |
| `GEMINI_API_KEY` | One AI key required | Google Gemini API key. |
| `GEMINI_MODEL` | No | Optional Gemini model override. |
| `ANTHROPIC_API_KEY` | One AI key required | Anthropic API key. If both provider keys are set, the server selects Anthropic. |
| `ANTHROPIC_MODEL` | No | Anthropic model override; the template provides a default model name. |
| `DATA_GOV_API_KEY` | No | Enables mandi-price requests through data.gov.in. |
| `PORT` | No | Local HTTP server port; defaults to `8787`. |

## Data handling and privacy

- The farm profile and selected language are saved in the current browser's local storage. The prefilled profile is fictional demo data, as confirmed by the project owner.
- When asking a question, the browser sends the profile, question, and selected language to the local KRISHAK server. If a photo is attached, it is sent to the configured AI provider for analysis.
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

This syntax check does not verify API keys, external services, or end-to-end browser behavior. Test those manually with valid credentials and network access, while avoiding real personal data.

## Contributing

Contributions should explain the problem being addressed and include steps to verify the change. Keep credentials, private data, and generated logs out of commits; use the placeholder `server/.env.example` for configuration documentation. Review agricultural claims and cite reliable sources when updating crop guidance.

The project code is licensed under the MIT License; see [`LICENSE`](LICENSE). This does not replace or override terms that apply to third-party services, fonts, datasets, or content. Review the provenance and reuse rights of all such materials before redistributing them or contributing changes.

## Third-party services and materials

- Google Fonts are loaded from Google Fonts at runtime; review applicable font licenses and attribution.
- Weather data is requested from Open-Meteo.
- Optional mandi data is requested from Agmarknet through data.gov.in.
- The origin and redistribution terms for the agricultural text in `server/knowledge.json` have not been established in the repository.

Review current service and data terms, attribution requirements, and the source/license of all included content before public distribution.
