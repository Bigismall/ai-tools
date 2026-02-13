---
name: usestate-to-usereducer
description: "Refactor React TSX: if target component has >3 useState hooks, consolidate into a single useReducer and extract reducer to adjacent file."
argument-hint: "Run on an open .tsx file (best in Edit mode)."
agent: agent
---

# Goal
Refactor the currently open file `${fileBasename}`:
- If the target React component contains **more than 3** `useState` hooks, replace them with **one** `useReducer`.
- Create the reducer in a new adjacent file: `${fileDirname}/${fileBasenameNoExtension}.reducer.ts`.

Do not do this refactor if the condition is not met.

# Scope / Constraints
- React (no Next.js specifics).
- Keep behavior the same (no functional changes).
- No new dependencies.
- Keep Tailwind `className` strings unchanged (unless you must touch whitespace due to formatting).

# Pick the target component
Target the main exported component in this file using this priority:
1) `export const ${fileBasenameNoExtension} = (...) => ...`
2) otherwise: the first `export const <ComponentName> = (...) => ...` that returns JSX
3) otherwise: the component with the highest number of `useState` calls

# Trigger condition
Count `useState` hook usages inside the target component body:
- If `<= 3`: **do not change code**. Reply with: “Found X useState hooks in <ComponentName>; skipping.”
- If `> 3`: proceed.

# What to generate (new reducer file)
Create (or update if already exists) this file:
`${fileDirname}/${fileBasenameNoExtension}.reducer.ts`

It must contain **named exports only**:
- `export type SetStateAction<T> = T | ((prev: T) => T);`
- `export type State = { ... }` where keys match the old `useState` state variable names
- `export type Action = { [K in keyof State]: { type: K; payload: SetStateAction<State[K]> } }[keyof State];`
- `export const reducer = (state: State, action: Action): State => { ... }`

Reducer rules:
- Try to understand all possible component states, then name them as and introduce a single `State` type that captures all of them (instead of multiple independent `useState` variables).
- Pure function (no side effects).
- Switch on `action.type` to determine which field to update.
- Action payload should ALWAYS be a plain value, never function (even if the setter supported functional updates, the reducer action should be a plain value for simplicity).
- Each case should modify State object with a simple spread and update pattern (e.g., `return { ...state, foo: newValue }`).
- Return a new object with only the one field updated (no mutations).
- Use human-readable generic Types names (e.g., `State`, `Action`, `SetStateAction`) and a simple action shape

Sample reducer structure:
```ts

export type SimpleContractModalKind =
  | 'CREATED'
  | 'SEE_FLAGS'
  | 'SHARED'
  | 'STOPPED'
  | 'SIGNED_CONSUMER'
  | 'SIGNED_BOTH'
  | 'SIGNED_BUSINESS'
  | 'LOADING'
  | 'EDIT';

export type SimpleContractModalType = Record<
  SimpleContractModalKind,
  { visible: boolean }
>;
export type SimpleContractModalState = {
  contract: SimpleContractListItem | null;
  modal: SimpleContractModalType;
};
export type SimpleContractModalActionType =
  | SimpleContractModalKind
  | 'INIT'
  | 'RESET';
export type SimpleContractModalAction = {
  type: SimpleContractModalActionType;
  payload?: {
    contract: SimpleContractListItem;
  };
};

export const SimpleContractModalDefaultValue: SimpleContractModalState = {
  contract: null,
  modal: {
    ITEM_LOADING: { visible: false },
    EDIT: { visible: false },
    CREATED: { visible: false },
    SEE_FLAGS: { visible: false },
    SHARED: { visible: false },
    STOPPED: { visible: false },
    SIGNED_CONSUMER: { visible: false },
    SIGNED_BOTH: { visible: false },
    SIGNED_BUSINESS: { visible: false },
  },
};

const setModalVisible = (
  kind: SimpleContractModalKind,
): SimpleContractModalState['modal'] => ({
  ...SimpleContractModalDefaultValue.modal,
  [kind]: { visible: true },
});

export const SimpleContractModalReducer = (
  state: SimpleContractModalState,
  action: SimpleContractModalAction,
): SimpleContractModalState => {
  switch (action.type) {
    case 'INIT':
      if (!action.payload?.contract) {
        throwContractIsRequired();
      }
      return {
        ...state,
        modal: setModalVisible(action.payload.contract.statusEnum),
        contract: action.payload.contract,
      };
    case 'RESET':
      return {
        ...state,
        modal: { ...SimpleContractModalDefaultValue.modal },
        contract: null,
      };
    case 'ITEM_LOADING':
      return {
        ...state,
        modal: setModalVisible('ITEM_LOADING'),
        contract: null,
      };
    case 'EDIT':
      if (!action.payload?.contract) {
        throwContractIsRequired();
      }
      return {
        ...state,
        modal: setModalVisible('EDIT'),
        contract: action.payload.contract,
      };

    case 'SIGNED_BOTH':
      return {
        ...state,
        modal: setModalVisible('SIGNED_BOTH'),
        contract: action.payload.contract,
      };

    case 'SIGNED_BUSINESS':
      return {
        ...state,
        modal: setModalVisible('SIGNED_BUSINESS'),
        contract: action.payload.contract,
      };
    case 'SIGNED_CONSUMER':
      return {
        ...state,
        modal: setModalVisible('SIGNED_CONSUMER'),
        contract: action.payload.contract,
      };
    case 'STOPPED':
      return {
        ...state,
        modal: setModalVisible('STOPPED'),
        contract: action.payload.contract,
      };

    // Unsupported contracts - we don't need to do anything
    case 'CREATED':
    case 'SEE_FLAGS':
    case 'SHARED':
      return { ...state };

    default:
      throw new Error(`Unknown action type: ${action.type}`);
  }
};
```


# How to refactor the component file
## 1) Imports
- Replace `useState` import usage with `useReducer`.
- If you create wrapper setters with stable identity (recommended), also import `useCallback`.
- Use `import type` for types (`State`, `SetStateAction`) from the reducer file.

## 2) Replace multiple useState with one useReducer
For each old state:
`const [foo, setFoo] = useState<...>(initialFoo);`

Build a single state object with keys `{ foo, ... }`.

### Initial state semantics
Preserve the original initializer behavior:
- If a `useState` used a lazy initializer (`useState(() => expr)`), keep that laziness (do not eagerly run `expr` on every render).
- If it used an eager initializer (`useState(expr)`), keep evaluation semantics as close as practical (don’t “magically” make eager expressions lazy unless clearly safe).

Preferred implementation:
- If at least one lazy initializer exists, use `useReducer(reducer, initArg, init)`:
  - `initArg` can carry either a direct value or a `() => value` for lazy ones
  - `init` resolves those into the final `State` once
- Otherwise, `useReducer(reducer, initialState)` is fine.

## 3) Keep variable names stable
After `useReducer`, keep the old state variable names available without rewriting the whole file, e.g.:
- `const { foo, bar, baz } = state;`

## 4) Replace setters
Goal: remove `setFoo` coming from `useState`, but keep the rest of the file readable and correct.

Preferred approach (minimal churn + preserves stable setter identity like `useState` setters):
- Recreate each setter name as a `useCallback` wrapper that dispatches:
- dispatch an action `{ type: 'foo', payload: payload }` where `payload` is value

Example pattern (repeat per state key):
- `const setFoo = useCallback((payload: SetStateAction<State['foo']>) => dispatch({ type: 'foo', payload }), [dispatch]);`

Then:
- Do **not** rewrite call sites that already use `setFoo(...)` unless necessary (they should keep working).

## 5) Cleanup
- Remove now-unused `useState` calls, imports, and any dead code created by the refactor.
- Ensure TypeScript types still compile.
- Ensure there are no unused vars/imports after changes.

# Output
- Apply edits to the current file and create/update the reducer file.
- End with a short summary (max 5 bullets): what changed (e.g., “consolidated 5 useState hooks into useReducer; extracted reducer to X.reducer.ts; preserved setState semantics via SetStateAction wrappers”).
