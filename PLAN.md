# Cockpit MVP Plan

## Summary
- Build `Cockpit` as a macOS-first Electron desktop app using TypeScript, Vue 3, Tailwind CSS, PrimeVue, and Vitest on this currently empty repo.
- Use `electron-vite` for the app shell, `@agentclientprotocol/sdk` with `copilot --acp --stdio` for Copilot integration, and a file-backed local store under Electron `userData` for projects, threads, settings, transcripts, git metadata, and model preferences.
- Keep the UI simple but align with the screenshots’ core shell: project picker, thread sidebar, transcript, composer, top action bar, model selector, repo status, and commit/push controls.

## Requirements and user stories
- As a developer, I can add a local project folder, see it in Cockpit, and open its saved threads.
- As a developer, I can create a thread, send prompts, watch streaming output, stop a response, and retry after an error.
- As a developer, I can quit and relaunch Cockpit and still see my projects, thread titles, transcript, selected model, and last known status.
- As a developer, I can reopen a thread and continue it with preserved context from Cockpit’s stored summary and recent messages.
- As a developer, I can see whether Copilot CLI is installed and authenticated, and get guided install/login steps when it is not ready.
- As a developer, I can choose an available Copilot model for a thread from a UI dropdown, and that choice persists with the thread.
- As a developer, I can see the current git branch and dirty state for the active project, review changed files, enter a commit message, and run commit + push from the app.
- As a developer, I can approve or deny Copilot tool/trust requests from the GUI.
- As a QA tester, I can validate fresh-install, happy-path chat, persistence, model switching, permission prompts, git commit/push flow, and crash recovery with automated tests plus a manual smoke checklist.

## Implementation changes
- Scaffold one Electron app with Vue 3 Composition API, Pinia, PrimeVue styled mode, Tailwind CSS, `tailwindcss-primeui`, and Vitest; standardize on Node 22 LTS and `npm`.
- Use one main window with three regions: left project/thread sidebar, center transcript, bottom composer; add a compact top bar for project name, branch/status, model selector, `Open`, and `Commit & Push`.
- Keep all process and OS access in the Electron main process with `contextIsolation: true`, `sandbox: true`, and `nodeIntegration: false`.
- Define preload/API contracts for:
  - `system.getCliHealth`, `system.pickProjectDirectory`, `system.openProjectPath`
  - `projects.list/create/remove/select`
  - `threads.list/create/rename/delete/open/updateModel`
  - `chat.send/stop/retry/subscribe`
  - `git.getStatus`, `git.listChangedFiles`, `git.commitAndPush`
  - `settings.get/update`
- Persist:
  - `ProjectRecord { id, name, rootPath, createdAt, updatedAt, lastOpenedAt }`
  - `ThreadRecord { id, projectId, title, summary, modelId, createdAt, updatedAt, lastMessageAt, status }`
  - `MessageRecord { id, threadId, role, content, createdAt, kind, metadata }`
  - `CliHealth { installed, version, executablePath, state, error }`
  - `GitStatus { branch, upstream, ahead, behind, changedCount, untrackedCount, isClean }`
  - `PermissionRequest { id, threadId, kind, prompt, options }`
- Store metadata as JSON plus append-only `messages/<threadId>.jsonl`; avoid SQLite/native modules for MVP.
- Keep Cockpit persistence as the source of truth. On reopen, start a fresh ACP session and bootstrap it with the stored thread summary and recent exchanges; store native Copilot session ids only as optional metadata.
- Model selector behavior:
  - Per-thread model selection, shown in the top bar for the active thread.
  - Discover models dynamically from Copilot CLI by running a transient helper session that invokes `/models` and parsing the returned list; cache results per app launch and allow manual refresh.
  - Start ACP sessions with `--model <modelId>` for the active thread.
  - If the user changes the model on an existing idle thread, restart the runtime session for subsequent prompts while preserving the stored transcript.
  - If discovery fails, show the current/default model and disable switching until refresh succeeds.
- Git actions behavior:
  - Read repo state from the active project root only.
  - Show branch, ahead/behind, and changed-file counts in the top bar.
  - Expose a simple changed-files panel for review, but no stage/unstage, discard, or hunk UI in MVP.
  - `Commit & Push` performs `git add -A`, `git commit -m <message>`, then `git push` to the current upstream; if no upstream exists, show a guided error instead of inventing branch setup UX.
  - Disable commit/push while a Copilot run is active to avoid overlapping repo state changes.
- Onboarding/status:
  - Detect CLI with `copilot version`.
  - Detect auth/setup problems from startup and command failures.
  - Provide copyable install/login commands and optional executable-path override in Settings.

## Test and QA plan
- Unit tests with Vitest for store serialization, thread-title generation, transcript bootstrapping, CLI health parsing, model-discovery parsing, git-status normalization, and commit/push state reducers.
- Service tests with Vitest against fake ACP and git command fixtures for streaming responses, permission prompts, cancellation, model-change restarts, git failure handling, and recovery after process exit.
- Component tests with Vitest and Vue Test Utils for sidebar selection, transcript rendering, composer states, onboarding banners, model dropdown states, changed-files panel, and commit dialog validation.
- Manual QA stories:
  1. Fresh macOS machine without Copilot CLI shows install guidance and never crashes.
  2. Installed but logged-out CLI shows auth-required state and recovers after login.
  3. User creates a project, starts a thread, receives streamed output, and stops a run cleanly.
  4. User opens the model selector, sees CLI-discovered models, switches models, and the next prompt uses the new model.
  5. User sees branch and dirty state, reviews changed files, commits all changes, and pushes successfully.
  6. User attempts commit/push with no upstream or a rejected push and gets a recoverable error with no transcript loss.
  7. User approves and denies separate permission prompts and sees the correct transcript/status results.
  8. User quits Cockpit, relaunches it, reopens a thread, and continues from preserved transcript and model selection.
- MVP acceptance criteria: automated tests pass locally, the manual smoke checklist passes on macOS, persistence works across restart, model selection works from CLI-discovered options, and commit/push works on a repo with a configured upstream.

## Assumptions and defaults
- Locked choices: app name `Cockpit`, macOS-first, projects plus threads, ACP-first integration, guided CLI readiness, per-thread model selector, dynamic model discovery from CLI, and simple git status plus commit/push.
- Out of scope for MVP: staging UI, discard/reset actions, pull/fetch workflows, pull request management, attachments, cloud sync, auto-update, notarization, and multi-window support.
- GitHub’s docs confirm `--model` and `/models` support model selection, and ACP support is still public preview; the dynamic model discovery approach is an implementation inference based on those documented interactive commands.
- References: [Copilot CLI ACP server](https://docs.github.com/en/copilot/reference/acp-server), [CLI command reference](https://docs.github.com/en/copilot/reference/cli-command-reference), [About Copilot CLI](https://docs.github.com/en/copilot/concepts/agents/copilot-cli/about-copilot-cli), [CLI best practices](https://docs.github.com/en/copilot/how-tos/copilot-cli/cli-best-practices), [electron-vite](https://electron-vite.org/guide/), [PrimeVue Tailwind integration](https://primevue.org/tailwind/), [Vitest guide](https://vitest.dev/guide/).
