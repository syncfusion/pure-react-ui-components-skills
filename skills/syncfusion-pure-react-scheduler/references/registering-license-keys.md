# Registering Syncfusion React License Keys

Register your Syncfusion license key in your React application to remove runtime licensing notices.

## Prerequisites

Before registering, you must:
1. Have a valid Syncfusion license (trial, paid, or community)
2. Generate a license key from your Syncfusion account
3. Have access to your application entry point

## Registration Methods

**Choose ONE registration approach for your project.** Do not mix multiple methods.

Select the approach that best fits your workflow:

1. **Code Registration** (recommended for most applications): Register directly in your application code during startup
2. **CLI Activation** (for build processes or CI/CD): Register via command-line interface without code changes

Once you choose an approach, implement only that method. Stop after implementation is complete.

## Code Registration (Recommended for Most Cases)

Register the license key in your app's existing entry point before rendering Syncfusion components.

**Entry points vary by framework:**
- Vite, React: `main.tsx` or `main.ts`
- Next.js: `pages/_app.tsx` or `app/layout.tsx`
- Create React App: `src/index.tsx` or `src/main.tsx`
- Remix: `root.tsx`
- React Native: `index.js`

Find and modify only your framework's entry point. Do not create separate licensing files or documentation.

### Example: Register in main.tsx

**File: main.tsx** (existing file - add these lines at the top)

```tsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import './styles/index.css'
import App from './App.tsx'
import { registerLicense } from '@syncfusion/react-base';

// Registering Syncfusion license key
registerLicense('Replace your generated license key here');

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <App />
  </StrictMode>,
)
```

### Using Environment Variables (Recommended for Security)

For security, load the key from an environment variable instead of hardcoding. Modify only your entry point:

**File: main.tsx** (existing file - add these lines at the top)

```tsx
import { registerLicense } from '@syncfusion/react-base';

const licenseKey = import.meta.env.VITE_SYNCFUSION_LICENSE;
if (!licenseKey) {
  console.warn('Syncfusion license key not found in environment variables');
}
registerLicense(licenseKey);
```

Your framework's existing `.env` file (development only):

```
VITE_SYNCFUSION_LICENSE=your-license-key-here
```

**Important**: Modify only your entry point and existing `.env` file. Do not create separate license documentation or config files.

## CLI Registration (No Code Changes)

**Use CLI Registration if:** You prefer to register without modifying application code, or you're deploying to CI/CD pipelines.

**Choose ONE approach below:**

### Approach 1: Register with License File (Development)

Create `syncfusion-license.txt` in project root and activate via CLI:

**Step 1: Create License File**

File: `syncfusion-license.txt` (project root)

```
your-license-key-here
```

**Step 2: Activate the License**

From your project root, run:

```bash
npx syncfusion-react-license activate
```

**Expected Output**

```
License message: (INFO) Syncfusion License imported successfully.
```

**Step 3: Clear Cache and Test**

Remove the cache directory and restart your application:

```bash
rm -r node_modules/.cache
npm run dev
```

Add to existing `.gitignore`:

```
syncfusion-license.txt
```

---

### Approach 2: Register with Environment Variable (CI/CD)

Use an environment variable for CLI activation in CI/CD pipelines or shared environments:

**Step 1: Set Environment Variable**

Set `SYNCFUSION_LICENSE` with your key:

**Windows (CMD)**

```cmd
setx SYNCFUSION_LICENSE "your-license-key-here"
```

**Linux/Mac (Bash)**

```bash
export SYNCFUSION_LICENSE="your-license-key-here"
```

**Step 2: Restart IDE/Terminal**

After setting the environment variable, restart your IDE or terminal for changes to take effect.

**Step 3: Activate the License**

From your project root:

```bash
npx syncfusion-react-license activate
```

**Expected Output**

```
License message: (INFO) Syncfusion License imported successfully.
```

**Step 4: Clear Cache and Test**

Remove the cache and restart your application:

```bash
rm -r node_modules/.cache
npm run dev
```

---

**Do not use both Approach 1 and Approach 2 in the same project. Choose one based on your deployment strategy.**

## Implementation Scope

**Choose ONE approach and implement only those changes:**

| Scenario | Approach | Modify |
|----------|----------|--------|
| Development, small teams | Code Registration | Entry point only |
| Production deployment | Code Registration + env vars | Entry point + .env |
| CI/CD pipelines | CLI + Environment Variable | System env only |
| Local dev without code changes | CLI + License File | `syncfusion-license.txt` + `.gitignore` |

**Modifications are minimal:**
- Entry point file (existing): Add 3-4 lines for registration
- Environment config (existing): Add license key variable
- `.gitignore` (existing): Add one line to exclude license file
- No new files beyond `syncfusion-license.txt` (if using CLI approach)
- No documentation changes needed (this skill's references are authoritative)

## Troubleshooting

**Still Seeing Licensing Notice**

- Verify the license key is correctly registered
- Confirm the key hasn't expired (for trial licenses)
- Ensure `registerLicense()` is called before any component rendering
- Clear browser cache and rebuild if using code registration
- Delete `node_modules/.cache` if using CLI registration

**License Key Not Found**

- Double-check the key string for typos
- Verify the key is for the correct Syncfusion version
- Confirm your license is active in your account

**Build or Activation Failures**

- Ensure `SYNCFUSION_LICENSE` environment variable is properly set
- Restart terminal/IDE after setting environment variables
- Verify file permissions for `syncfusion-license.txt`

## After Registration

**Verify Implementation**
- Run `npm run lint` / `tsc --noEmit` (no syntax errors in entry point)
- Run `npm run build` (build succeeds)
- Run `npm test` (existing tests still pass)
- Start app (`npm run dev`) and confirm no Syncfusion license warnings

**Key Security Rules**
- Never commit actual keys (`.gitignore` excludes license files)
- Use environment variables for CI/CD (stored in CI secret manager)
- Verify no keys appear in git history before pushing

**Common Issues**
- Build fails? → Check `@syncfusion/react-base` installed, import path correct
- License warning shows? → Ensure `registerLicense()` is first in entry point
- Tests fail? → Mock Syncfusion imports in test setup if needed
