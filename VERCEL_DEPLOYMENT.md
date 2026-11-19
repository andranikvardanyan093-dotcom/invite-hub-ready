# Vercel deployment

This archive is arranged with the Vite application files at the project root. Do not select an additional `invitehub/` subdirectory as the Vercel Root Directory.

## Vercel project settings

- Framework Preset: **Vite**
- Root Directory: **`.` (project root)**
- Install Command: **`npm ci`**
- Build Command: **`npm run build`**
- Output Directory: **`dist`**
- Node.js: **22.x or newer (22.12.0 minimum)**

The entry file is `index.html`, and it imports `src/main.jsx`. Both are placed at the same project root expected by Vite. The Vercel Root Directory must be `.`; do not point it to `src` or a parent folder.

## Environment variables

For a production Supabase connection, add these in Vercel → Project → Settings → Environment Variables:

- `VITE_SUPABASE_URL`
- `VITE_SUPABASE_ANON_KEY`
- `VITE_APP_URL` (your deployed site URL)

Do not put Supabase service-role keys or private OpenAI API keys in `VITE_` browser variables. Keep server-side secrets in the appropriate Supabase Edge Function environment instead.
