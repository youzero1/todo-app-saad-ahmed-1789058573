---
status: pending
title: Personal Todo App with Due Dates & Local Persistence
---

## Scope

A single-user personal task app. Tasks live only on the device via `localStorage`. Emphasis on due dates: tasks are grouped and badged as Overdue / Today / Tomorrow / Upcoming / No date, with optional in-app browser notifications while the tab is open. No accounts, no server, no sync.

## Steps

1. **Scaffold the app shell and styling baseline.**
   Create `src/styles/global.css` containing exactly `@import "tailwindcss";`, and `src/main.tsx` importing it once, creating the TanStack Router instance from `src/routeTree.gen.ts` and rendering `RouterProvider` into `#root`. Ensure `vite.config.ts` registers the `@tailwindcss/vite` plugin and `@tanstack/router-plugin/vite`, and that `@/` resolves to `src/` in both `vite.config.ts` and `tsconfig.json`.
   *Outcome:* dev server boots, Tailwind classes apply, generated route tree is picked up.

2. **Define the task data model.**
   Add `src/types/task.ts` exporting a `Task` type: `id` (string), `title` (string), `notes` (optional string), `dueDate` (optional ISO date string, date-only, no time zone ambiguity), `completed` (boolean), `completedAt` (optional ISO string), `createdAt` (ISO string). Also export a `TaskDraft` type for creation input and a `DueBucket` union (`"overdue" | "today" | "tomorrow" | "upcoming" | "none"`).
   *Outcome:* one shared source of truth for task shape; no component defines its own inline task type.

3. **Build the persistence layer.**
   Add `src/lib/storage.ts` with a single versioned storage key (e.g. `todo.tasks.v1`), a `loadTasks()` that safely parses and validates the stored array (returning `[]` on missing/corrupt data), and a `saveTasks()` that serializes. Guard all access in try/catch so private-browsing or quota errors never crash the app.
   *Outcome:* reading/writing tasks is isolated from React and fails soft.

4. **Build due-date helpers.**
   Add `src/lib/dates.ts` with helpers to: get today's date-only string, compare a due date to today, derive the `DueBucket` for a task, produce a human label (`"Overdue by 2 days"`, `"Today"`, `"Tomorrow"`, `"Fri, 14 Mar"`), and sort tasks by due date with undated tasks last. All comparisons must be date-only in local time to avoid off-by-one issues.
   *Outcome:* every due-date string shown in the UI comes from this module.

5. **Create the tasks hook.**
   Add `src/hooks/useTasks.ts` that loads tasks once on mount, keeps them in state, and persists to storage on every change. Expose `tasks`, `addTask`, `toggleComplete`, `updateTask`, `deleteTask`, `clearCompleted`, and derived `groups` (tasks bucketed by `DueBucket`, each sorted) plus counts for overdue and due-today.
   *Outcome:* a single hook owns all task state and mutations; components stay presentational.

6. **Set up routes.**
   Add `src/routes/__root.tsx` as the app shell: centered max-width column, page header with the app name and a live "X overdue · Y due today" summary, `Outlet`, and a small footer note that data is stored on this device. Add `src/routes/index.tsx` for the main list view. Add `src/routes/completed.tsx` for a completed-tasks archive with a "Clear completed" action, linked from the header.
   *Outcome:* two working URLs (`/`, `/completed`) rendering inside a shared shell.

7. **Build the add-task input.**
   Add `src/components/AddTaskForm.tsx`: a prominent single-line title input with a compact native date input for the due date and quick-set chips (Today / Tomorrow / Next week / No date). Submitting on Enter adds the task and clears the title while keeping focus for fast entry. Reject empty/whitespace titles.
   *Outcome:* a task can be added with or without a due date in one keystroke flow.

8. **Build the task item.**
   Add `src/components/TaskItem.tsx`: checkbox toggle with a completed strikethrough state, inline title editing on click (Enter saves, Escape cancels), an editable due date, a due badge colored by bucket (overdue red, today amber, otherwise neutral), and a delete action. Ensure keyboard focus states and accessible labels on all controls.
   *Outcome:* every task operation is reachable from the row itself.

9. **Build the grouped list and empty states.**
   Add `src/components/TaskGroup.tsx` (section heading + count + task rows) and `src/components/EmptyState.tsx`. Wire `src/routes/index.tsx` to render groups in order Overdue → Today → Tomorrow → Upcoming → No date, hiding empty groups, and showing the empty state when no active tasks exist. Completed tasks do not appear here.
   *Outcome:* the main view reads as a calm, prioritized day plan.

10. **Add optional in-app reminders.**
    Add `src/hooks/useDueReminders.ts`: on user opt-in only, request `Notification` permission, then on mount and on a low-frequency interval notify once per task per day for tasks that are overdue or due today. Track already-notified ids in `localStorage` under a separate key. Add a small toggle control in `src/routes/__root.tsx` (or a `Reminders` component) that no-ops gracefully when the API is unsupported or denied.
    *Outcome:* nudges appear while the app is open; the app works identically with reminders off.

11. **Polish styling and responsiveness.**
    Apply a restrained Tailwind v4 palette (neutral background, one accent color, red/amber reserved for due states), generous spacing, `rounded-lg` cards, subtle borders over shadows, and comfortable touch targets. Verify single-column layout at 375px width and the centered column at desktop widths; respect `prefers-reduced-motion` for any transitions.
    *Outcome:* clean and legible on phone and desktop.

12. **Verification pass.**
    Confirm: add/edit/complete/delete persist across a full page reload; a task dated yesterday shows as Overdue; a task dated today shows Today; deleting the storage key yields the empty state without errors; `/completed` lists completed tasks and "Clear completed" empties it; no TypeScript errors and no `src/routeTree.gen.ts` edits.
    *Outcome:* all acceptance criteria pass on a fresh browser profile.

## Future ideas / out of scope

Multiple lists or projects, subtasks, labels and priorities, search and filtering, drag-to-reorder, completion stats, recurring tasks, cross-device sync.
