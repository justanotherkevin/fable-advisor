# Chat-Controlled Todo List — Design

## Overview

A browser todo app where the todo list is edited entirely through a natural-language command bar. The user never clicks a checkbox directly; instead they type instructions like "add buy milk" or "mark laundry done," and a DeepSeek-backed backend interprets the command, updates the list, and the checkboxes reflect the result. There is no chat reply thread — the only feedback is the list itself updating, or a toast when a command can't be carried out.

## Architecture

Two processes:

- **Frontend** — Vite + React + TypeScript. A single page with a command-bar input, a read-only todo list (checkbox + text per item), and a toast notification area.
- **Backend** — Node + Express + TypeScript. Exposes `GET /todos` and `POST /command`. Holds the DeepSeek API key; the frontend never talks to DeepSeek directly.

## Data Model

Stored in `todos.json` on the backend:

```ts
type TodoItem = {
  id: string;   // backend-generated (crypto.randomUUID())
  text: string;
  done: boolean;
};
```

IDs are never typed by the user. The model is given the full current list (id, text, done) as context on every request, and must reference items by `id` when calling a tool — this avoids relying on fuzzy text matching to figure out which item a command targets.

## Tools Exposed to DeepSeek

- `add_item(text: string)`
- `complete_item(id: string)`
- `uncomplete_item(id: string)`
- `remove_item(id: string)`

## Request Flow

1. User types a command and submits. The input clears immediately (command-bar behavior — no message history is shown).
2. Frontend calls `POST /command` with `{ message }`.
3. Backend reads `todos.json`, sends a system prompt (tool definitions + current list) plus the user's message to DeepSeek.
4. Backend inspects the response:
   - **Exactly one valid tool call**, referencing an existing `id` where applicable → apply it, write `todos.json`, respond `{ success: true, todos }` with the full updated list.
   - **No tool call, an invalid/ambiguous `id`, or a DeepSeek/network error** → respond `{ success: false }`. Nothing is written.
5. Frontend:
   - `success: true` → replace local `todos` state with the returned list; checkboxes update accordingly.
   - `success: false` → show a toast ("Couldn't do that, try again.") and leave the list unchanged.

The system prompt explicitly instructs DeepSeek to only call a tool when it is confident about a single matching item, and to call nothing otherwise. This keeps "no suggestions, just fail" behavior inside the model/backend contract rather than requiring disambiguation logic in the UI.

## Statelessness

Each submitted command is processed independently. The backend does not retain conversation history across requests — only the current todo list is passed as context. References like "it" or "that one" that depend on earlier commands are expected to fail (no matching id resolvable) and surface the toast.

## Components

**Frontend:**
- `App` — holds `todos` state, fetches the initial list via `GET /todos` on mount, renders `CommandBar` and `TodoList`.
- `CommandBar` — input + submit; calls `POST /command`; clears itself on submit; triggers the toast on `success: false`.
- `TodoList` / `TodoItem` — renders checkbox (`checked = done`, non-interactive) + text per item.
- `Toast` — transient message component, shown only on failure.

**Backend:**
- `GET /todos` — returns the current list, for initial page load.
- `POST /command` — implements the request flow above.
- `todoStore.ts` — reads/writes `todos.json`; exposes `addItem`, `completeItem`, `uncompleteItem`, `removeItem`.
- `deepseek.ts` — wraps the DeepSeek chat completions call and tool definitions.

## Testing

- **Backend:** basic tests for each `todoStore` operation (add, complete, uncomplete, remove), including the not-found case returning a failure rather than throwing.
- **Frontend/manual:** run the app and exercise the golden path — add an item via chat, complete it, uncomplete it, remove it — plus an unrecognized/ambiguous command to confirm the toast fires and the list is left unchanged.

## Out of Scope

- Multi-user support / auth.
- Conversation history or LLM chat replies.
- Manual checkbox interaction (checkboxes are read-only, driven only by chat commands).
