---
description: Run the project-audit workflow against the given path or codebase
---

Activate the `project-audit` skill.

Procedure:
1. Check if the user already provided the service profile in their message.
2. If any fields are missing, ask for **all missing fields in one batch** (service_type, stage, expected_mau, data_sensitivity, etc.).
3. Match the development stage to section ranges (planning → 0; pre-launch → 0–10; etc.).
4. Apply the grade filter (🔴🟠🟡🔵⚪) based on the profile.
5. Mark each checked item with ✅ / ❌ / ⚠️ / ⏭️.
6. End with ❌ items re-sorted by risk: Security > Reliability > Data > Performance > rest.

Full checklist: `SPEC.md` at the plugin root.

Respond in the user's language.

Target path / codebase: $ARGUMENTS
