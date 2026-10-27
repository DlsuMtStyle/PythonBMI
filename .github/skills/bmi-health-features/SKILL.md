---
name: bmi-health-features
description: "Implement and validate BMI health features in this Flask app, including chart-based result display, Taiwan military-service standard judgment, and configurable target BMI. Use when adding, modifying, testing, or reviewing these BMI features."
argument-hint: "Describe the BMI feature to implement or review"
user-invocable: true
---

# BMI Health Features

## Purpose

Extend the PythonBMI Flask application with user-facing health features while
preserving the existing BMI calculation, Taiwan adult classification, history
records, accessibility, input validation, and responsive layout.

## Current application context

- The application entry point is `app.py`.
- The main form and result display are in `templates/index.html`.
- Styling is in `static/style.css`.
- Existing automated tests are in `tests/test_app.py` and use `unittest`.
- BMI is calculated as kilograms divided by height in meters squared.
- Existing Taiwan adult classifications are:
  - Below 18.5: 體重過輕
  - 18.5 to below 24: 健康體位
  - 24 to below 27: 體重過重
  - 27 and above: 肥胖
- SQLite is used for persisted BMI records. Do not commit files under
  `instance/`.
- BMI results are rounded to one decimal place before display and
  classification.

## Features

### 1. Chart-based result display

As a BMI calculation user, I want to view my result as a chart so that I can
understand my BMI value and health range more intuitively.

Acceptance criteria:

- Show a chart after a valid BMI calculation.
- Clearly mark the calculated BMI and the corresponding health range.
- Render correctly on desktop and mobile layouts.
- Do not show a misleading chart when inputs are missing or invalid.
- Include accessible text or an equivalent semantic representation so the
  result is understandable without relying on color or graphics.

Prefer a dependency-free HTML/CSS/SVG implementation unless the task
explicitly requires a chart library. Keep chart values derived from the same
rounded BMI and classification logic used by the result panel.

### 2. Military-service standard judgment

As a BMI calculation user, I want the system to judge whether my BMI meets the
configured military-service standard so that I can quickly understand the
result.

Acceptance criteria:

- Show the judgment after valid height and weight inputs have produced a BMI.
- Clearly show whether the result meets the military-service standard and show
  the BMI used for the judgment.
- Apply the project-approved BMI ranges consistently, including boundary
  values. Do not invent or silently change official thresholds; if the
  thresholds are not specified in the task, request clarification or make the
  configured source explicit.
- Do not show a misleading judgment for missing or invalid inputs; show an
  actionable validation message instead.
- Present the judgment as informational guidance, not a medical or legal
  determination, and preserve the existing disclaimer style.

Keep the judgment in a named Python helper or configuration structure so its
boundary behavior can be unit tested independently from the route.

### 3. Configurable target BMI

As a user who wants to improve my health, I want to set a target BMI so that I
can compare my current BMI with my goal and track progress.

Acceptance criteria:

- Allow the user to enter and save a valid target BMI.
- Display the current BMI, target BMI, and their difference or progress.
- Recalculate the displayed progress when the target changes.
- Reject blank, malformed, non-finite, non-positive, or out-of-range target
  values with a clear validation message; never persist an invalid value.
- Restore the saved target when the user returns to the page.
- Scope persisted targets to the correct user identity and use parameterized
  SQL queries.

Define and document the accepted target range before implementation. Preserve
existing records and use a safe schema migration or additive table/column
change rather than destroying the SQLite database.

## Implementation procedure

1. Inspect the relevant route, template, stylesheet, database setup, and tests
   before editing.
2. Confirm any domain-specific threshold that is not already defined in the
   repository, especially military-service standards and target BMI bounds.
3. Keep parsing and validation in reusable Python helpers. Reject values that
   are non-numeric, non-finite, or not positive.
4. Add or update the smallest necessary database schema while preserving
   existing data and parameterizing all user-controlled values.
5. Pass explicit feature data to the template rather than duplicating BMI
   calculations in Jinja or JavaScript.
6. Preserve the existing visual language, responsive behavior, semantic HTML,
   escaped user input, and accessible error states.
7. Add unit tests for normal cases, lower/upper boundaries, invalid values,
   persistence, and the no-result state. Add route/template assertions for
   the visible labels and accessible text.
8. Run the focused tests first:

   ```bash
   python -m unittest discover -s tests
   ```

9. Review the diff for unrelated changes and verify that no generated
   database, credentials, or deployment secrets were added.

## Review checklist

- Existing BMI categories and history behavior remain unchanged.
- A valid calculation is the only path that renders chart and judgment data.
- Every acceptance criterion is covered by an automated test or a documented
  manual verification.
- Boundary values are tested explicitly rather than inferred from nearby values.
- Invalid input produces visible errors and never creates a false success state.
- The result is informative and avoids claiming a definitive medical or legal
  conclusion.
