# Chef Claude — AI Recipe Suggester (React + Vite)

A small React app that turns a list of ingredients into a full recipe suggestion using Anthropic’s Claude 3 model. You add what you have in your kitchen, press a button, and the app asks Claude for a recipe. The response is returned as Markdown and rendered nicely in the UI.

> **Heads‑up (security):** This project calls the Anthropic API **from the browser** (`dangerouslyAllowBrowser: true`). That’s fine for local learning, but **do not ship your real API key to production**. For any deploy, move the API call to a backend (server or serverless function) and keep your key private.

---

## Features

- Add ingredients through a small form; they render in a live‑updating list.
- “Get a recipe” button appears when there are **4+ ingredients** (see `IngredientsList.jsx`).
- Requests a recipe from **Claude 3 Haiku** with a kitchen‑friendly system prompt.
- Smooth‑scrolls to the suggestion area after a recipe arrives.
- Renders recipe output via **Markdown** using `react-markdown`.
- Basic accessibility: important sections use `aria-live="polite"` so screen readers catch updates.

---

## Tech Stack

- **React 18** (Vite)
- **@anthropic-ai/sdk**
- **react-markdown**
- Plain CSS

---

## Project Structure

```
.
├── public/
├── src/
│   ├── assets/images/chef-claude-icon.png
│   ├── components/
│   │   ├── ClaudeRecipe.jsx
│   │   ├── Header.jsx
│   │   ├── IngredientsList.jsx
│   │   └── Main.jsx
│   ├── ai.js
│   ├── App.jsx
│   ├── index.css
│   └── index.jsx
├── .env.local         # your Anthropic API key (Vite env var)
├── index.html
├── package.json
└── vite.config.js
```

---

## Getting Started

### Prerequisites

- **Node.js 18+** (or 20+ recommended)
- An **Anthropic API key**

### Install & Run

```bash
# Install deps
npm install

# Provide your Anthropic key in Vite's env file
# (create the file if it doesn't exist)
echo "VITE_ANTHROPIC_API_KEY=sk-your-key" > .env.local

# Start dev server
npm run dev

# Build for production
npm run build

# Preview the production build locally
npm run preview
```

> **Vite env note:** In Vite you read env vars with `import.meta.env`, not `process.env`. This project expects `VITE_ANTHROPIC_API_KEY` (the `VITE_` prefix is required for exposure to the client at build time).

---

## How It Works

- **`src/ai.js`** sets up the Anthropic client with:

  - Model: `claude-3-haiku-20240307`
  - A concise `SYSTEM_PROMPT`
  - `getRecipeFromChefClaude(ingredientsArr)` which formats the user ingredients and makes a `messages.create` call.

- **UI flow**
  1. `Main.jsx` holds local state for `ingredients` and `recipe`.
  2. Adding an ingredient appends it to the list.
  3. When there are 4+ ingredients, `IngredientsList.jsx` shows the “Get a recipe” button.
  4. Clicking the button calls `getRecipeFromChefClaude`, stores the result, and smooth‑scrolls to the recipe.
  5. `ClaudeRecipe.jsx` renders the returned Markdown using `react-markdown`.

---

## Configuration & Customization

- **Ingredients threshold:** In `IngredientsList.jsx`, change `props.ingredients.length > 3` to adjust when the button appears.
- **Model choice:** In `ai.js`, change `model: "claude-3-haiku-20240307"` to another Claude model if desired.
- **Max tokens:** Tune `max_tokens` in `ai.js` depending on recipe length you want.
- **Prompt style:** Edit `SYSTEM_PROMPT` to guide tone/format of the recipe.

---

## Production Safety (Important)

This learning demo calls the Anthropic API from the browser (`dangerouslyAllowBrowser: true`). For any real deployment:

1. **Create a backend endpoint** (Node/Express, Cloudflare Worker, Netlify/ Vercel Function, etc.).
2. Move the Anthropic client + API call into that backend.
3. Call your backend from the React app, not Anthropic directly.
4. Keep your API key only on the server; never commit it to git.

Minimal example (Express‑style pseudo‑code):

```js
// server.js
import express from "express";
import Anthropic from "@anthropic-ai/sdk";

const app = express();
app.use(express.json());

const anthropic = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

app.post("/api/recipe", async (req, res) => {
  const { ingredients } = req.body;
  const msg = await anthropic.messages.create({
    model: "claude-3-haiku-20240307",
    max_tokens: 1024,
    system: "You suggest recipes...",
    messages: [{ role: "user", content: `I have ${ingredients.join(", ")}` }],
  });
  res.json({ recipe: msg.content[0].text });
});

app.listen(3000);
```

Then in React, `fetch("/api/recipe")` instead of calling Anthropic directly.

---

## Known Issues / Troubleshooting

- **`App.jsx` content**  
  If `App.jsx` accidentally contains `ReactDOM.createRoot(...).render(<App />)` (copy of `index.jsx`), replace it with a proper component:

  ```jsx
  // src/App.jsx
  import Header from "./components/Header";
  import Main from "./components/Main";

  export default function App() {
    return (
      <>
        <Header />
        <Main />
      </>
    );
  }
  ```

- **`process is not defined` error**  
  Use `import.meta.env.VITE_ANTHROPIC_API_KEY` (Vite) instead of `process.env.ANTHROPIC_API_KEY`.

- **401/403 from API**  
  Check that your key is valid, the env var name is correct, and that it’s available at runtime.

- **429 (rate limit)**  
  You’re hitting Anthropic rate limits. Wait, or reduce call frequency / model size.

---

## Scripts

- `npm run dev` — start the Vite dev server
- `npm run build` — build production bundle
- `npm run preview` — preview production build

---

## License & Use

This project is intended for educational/demo use. If you plan to deploy, add a proper license and move API calls to a backend to protect secrets.
