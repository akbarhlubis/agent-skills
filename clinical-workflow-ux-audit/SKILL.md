---
name: clinical-workflow-ux-audit
description: Audit and improve UX for legacy clinical, hospital, SIMRS, inventory, pharmacy, billing, verification, registration, and patient workflow features. Use when users complain about repetitive input, confusing forms, unsafe actions, stale tables after save, date/time fatigue, paper-to-digital workflow mismatch, batch actions, modal context, table grouping, filters, pagination, previews, or UI polish in healthcare operational systems.
---

# Clinical Workflow UX Audit

Use this skill to audit workflow UX before changing legacy clinical or hospital features. Prioritize reducing user fatigue while preserving data correctness, auditability, and existing operational habits.

## Core Posture

- Treat UX as workflow design, not only visual polish.
- Learn the existing feature before proposing changes: controller, Blade, JavaScript, routes, services, and relevant database tables.
- Compare the digital flow with the real manual or paper flow when the user provides one.
- Keep improvements incremental and reversible in legacy systems.
- Preserve manual fallback paths unless the user explicitly wants them removed.
- For rsud-tgms or similar projects, read `AGENT_RULES.md` before editing project files.

## Audit Workflow

1. Map the current journey from the user's first click to saved data.
2. Identify the operator's repeated work: repeated dates, repeated times, duplicated drug rows, repeated confirmation, repeated search, or refresh dependency.
3. Identify missing context: patient name, no rawat, medicine name, unit, old value vs new value, current stock, source request, or status.
4. Identify risky actions: cancel, approve, reject, mutate stock, update price, delete rows, rollback, or batch update.
5. Check whether the UI updates after save without manual refresh.
6. Check table usability: default filters, date range, search, grouping, pagination, row density, sticky context, and export needs.
7. Check validation and empty states: missing required data, null relations, invalid dates, zero quantities, unchecked arrays, and disabled experimental inputs.
8. Classify findings into quick wins, medium changes, and high-risk changes.
9. Propose the smallest implementation that removes the main friction.
10. Verify with a realistic scenario and mention what was not tested.

## Healthcare UX Heuristics

- Default to today's data for operational lists, but make date filters obvious.
- For repeated schedule input, prefer checklist grids, presets, copy-forward, or batch apply over repeated manual datetime input.
- For verification flows, use one primary action to open a decision modal, then show approve/reject choices with clear confirmation.
- For destructive or irreversible actions, show what will change and require explicit confirmation.
- For batch actions, show a preview count and the exact grouped scope, such as no rawat instead of only patient name.
- For price or stock changes, always show old value, new value, difference, and calculation mode.
- For dense medical tables, group by the operator's decision unit: no rawat, patient, invoice, request number, medicine, or room depending on the task.
- Keep table actions compact and consistent with the existing project style.
- Do not hide critical business state behind color only; include text labels.
- Use plain operational language familiar to hospital users.

## Output Format

When auditing only, respond with:

- Current flow
- Main pain points
- Recommended flow
- Implementation plan
- Risks and validation

When implementing, first summarize the chosen change briefly, then edit the smallest safe set of files. After implementation, report changed files and verification.

## Common Fix Patterns

- Replace repeated datetime inputs with a date range plus time-slot checklist.
- Replace multiple row buttons with one action that opens a contextual modal.
- Add preview panels for old vs new price, stock, or status before saving.
- Add default today filters and explicit search controls for long lists.
- Group high-volume rows by no rawat, patient, invoice, or request, based on how users decide.
- Add optimistic table refresh or DataTable reload after successful AJAX actions.
- Add batch selection only when the selected scope is unambiguous and previewed.
- Mark unavailable experimental methods as read-only with clear info text.

## Red Flags

- The user must type the same date or time many times.
- A save action can happen accidentally via Enter without confirmation.
- The user must refresh the page to see the result of save or verification.
- The modal title does not mention the patient, medicine, no rawat, or request being changed.
- A table shows all historical data by default without a date filter.
- Approve and reject buttons sit directly in a crowded table without context.
- A batch action does not preview what rows will be affected.
- Price, stock, or billing changes do not show before and after values.
