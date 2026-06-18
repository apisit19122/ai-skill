---
description: Rules for agent behavior in this all repository 
alwaysApply: true
---
# User Rules
## Language
- Always respond in Thai unless the user requests another language
- `console.log` / `console.warn` / `console.error` messages that the agent **adds or modifies** must be written in Thai
- Do not translate existing logs in the original code unless the user requests it

## Package Installation
- If the user does not specify a version explicitly, install the **latest compatible version** for the project stack (check `package.json`, lockfile, Node/engine fields)
- Use the project's package manager (e.g. `pnpm` in OSS-PORTAL)
- If a major upgrade may be breaking, notify the user before installing

## Summary When Done
After editing code, include a **Modified Files** section at the end of the response:
- Use markdown links that can open the file, not just filenames
- Format: `[filename](path/from/workspace-root)` or full path if multi-root
- If no files were changed, explicitly state that no files were modified

Example:
````text
## Modified Files
- [PromotionDetailModal.tsx](src/components/Promotion/PromotionDetailModal.tsx)
- [gip01IncomeProgram.ts](src/components/Promotion/PromotionItem/shared/gip01IncomeProgram.ts)
````

## TypeScript
- Do not use the `any` type in files the agent edits (`.ts`, `.tsx`)
- Use existing types/interfaces, `unknown` + narrowing, or generics instead
- Avoid `as any` and `@ts-ignore` unless the user explicitly accepts it as an exception
- Prefer `??` (nullish coalescing) over `||` or explicit `null`/`undefined` checks when the intent is to provide a default for `null` or `undefined` only
- Prefer `??=` for nullish assignment and `?.` for optional chaining where applicable
- Requires `strictNullChecks: true` in TSConfig for this to work correctly

## Complexity
- Cyclomatic complexity of functions the agent **writes or modifies** must not exceed **15**
- If logic is too complex, extract it into sub-functions or helpers instead of consolidating branches into a single function

## Code Quality
- Duplicated lines in files the agent **writes or modifies** must not exceed **3%**
- If similar logic appears multiple times **within the same file**, extract it into a local helper, utility function, or constant
- Do **not** touch or refactor files outside the current task scope, even if duplication exists elsewhere
- **Never use nested ternary operators** (ternary inside ternary) — SonarQube flags these as *"Extract this nested ternary operation into an independent statement"*
- Extract nested ternaries into a named helper function or separate `if`/`else` logic instead:
```ts
  // ❌ Bad — triggers SonarQube nested ternary warning
  wantsPet: state.wantsPet === undefined ? undefined : state.wantsPet ? 'true' : 'false'

  // ✅ Good — extract into a helper
  const toBooleanString = (val: boolean | undefined): string | undefined => {
    if (val === undefined) return undefined;
    return val ? 'true' : 'false';
  };
  wantsPet: toBooleanString(state.wantsPet)
```
- For simple null/undefined fallbacks, prefer `??` over ternary entirely
````
