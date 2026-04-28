# Next.js Reproduction: `getTypeScriptConfiguration` pathsBasePath bug

Reproduces: `getTypeScriptConfiguration` deletes `baseUrl` for TypeScript 6 compatibility but does not set `pathsBasePath`, causing TypeScript to resolve rewritten path aliases from the wrong base directory in monorepos.

## Structure

```
tsconfig.base.json          ← defines baseUrl + paths at repo root
apps/
  my-app/
    next.config.ts
    package.json
    tsconfig.json           ← extends ../../tsconfig.base.json (no paths of its own)
    src/
      app/
        layout.tsx
        page.tsx            ← imports from @scope/shared-lib/foo
libs/
  shared-lib/
    src/
      foo.ts
      index.ts
```

## Steps to Reproduce

```bash
npm install
cd apps/my-app
npx next build
```

## Expected

Build completes. `@scope/shared-lib/foo` resolves correctly.

## Actual

TypeScript check fails:

```
Type error: Cannot find module '@scope/shared-lib/foo' or its corresponding type declarations.
```

## Root Cause

In `getTypeScriptConfiguration.js`, the TypeScript 6 block deletes `baseUrl` and rewrites `paths` but does not update `pathsBasePath`. TypeScript then resolves the rewritten `../../libs/...` paths from the repo root (where `pathsBasePath` points — the directory of `tsconfig.base.json`) rather than from `apps/my-app`.

## Fix

After `delete result.options.baseUrl`, add:

```js
result.options.pathsBasePath = path.dirname(tsConfigPath);
```
