# Next.js Instrumentation & pageExtensions Issue Reproduction

This repository demonstrates a regression in `next@canary` regarding `instrumentation` hooks and custom `pageExtensions`.

## Expected Behavior
The `instrumentation` hook should be loaded in the `nodejs` runtime.

## Actual Behavior (in `canary`)
1. Adding `src/instrumentation.api.ts` causes all static assets (`_next/static/...`) to return `404 (Not Found)`.
2. The `register()` function is never executed (no logs in server console).
3. `server.js` becomes unstable.

## Reproduction Steps
1. `npm install`
2. `npm run build`
3. `npm run standalone`
4. Access `http://localhost:3000` and check browser console.

## Technical Investigation
During the build process, the `instrumentation` hook is incorrectly assigned to the `edge` runtime. This is confirmed by inspecting `.next/server/instrumentation/middleware-manifest.json`:

```json
{
  "instrumentation": {
    "files": [
      "server/edge/chunks/_1rqv1up._.js",
      "server/edge/chunks/turbopack-node_modules_next_dist_esm_build_templates_edge-wrapper_02kynl8.js"
    ],
    "name": "instrumentation"
  }
}
```

This confirms that the build engine is forcing the instrumentation hook into the edge runtime, despite it being intended for the nodejs runtime.