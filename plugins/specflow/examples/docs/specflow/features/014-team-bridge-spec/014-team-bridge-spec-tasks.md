---
feature: 014-team-bridge-spec
status: shipped
created: 2026-05-06
shipped: v2.4.0
templateVersion: v2.3
prd: ./014-team-bridge-spec-prd.md
---

# 014-team-bridge-spec — task plan

## Coverage matrix

| Requirement | Tasks |
|---|---|
| R1 — doctrine doc | T-1 |
| R2 — severity mapping | T-1 |
| R3 — field mapping | T-1 |
| R4 — dedup rules | T-1 |
| R5 — auto-promotion contract | T-1 |
| R6 — when-to-invoke trigger | T-1 |
| R7 — worked example | T-2 |

| AC | Tasks |
|---|---|
| AC-1 through AC-6 | T-1 |
| AC-7 | T-2 |
| AC-8 | T-3 |

---

## T-1 — Author `templates/admin/team-review-bridge.md` (full doctrine)

- **Anchors:** R1, R2, R3, R4, R5, R6.
- **Acceptance:** Doctrine doc exists with all six required sections (rationale, severity mapping, field mapping, dedup rules, auto-promotion, when to invoke). Cites `develop/SKILL.md:475-490` for auto-promotion contract.
- **Lane:** Green.
- **Files:** `plugins/specflow/templates/admin/team-review-bridge.md` (new).
- **Estimate:** 25 minutes.

---

## T-2 — Build worked example folder + sample translation

- **Anchors:** R7, AC-7.
- **Acceptance:** Folder exists with PRD, interview, tasks file, Gate 2 manifest closed `passed`. Tasks file contains a sample `team-review` finding (markdown shape) translated to a Gate-5 manifest entry (JSON shape) using the bridge mapping.
- **Lane:** Green.
- **Files:** New folder.
- **Estimate:** 20 minutes.

### Sample translation (lives in this tasks file as the worked example)

**Input — `team-review` finding (markdown):**

```markdown
### [High] Token refresh path mishandles 401

**Location**: `app/lib/auth.ts:88`
**Dimension**: Security
**Severity**: High

**Evidence**:
The 401 branch returns the original error to the caller without invalidating the refresh token. A subsequent retry will use the same expired token and re-fail with 401, producing an infinite loop.

**Impact**:
Authentication failure cascades into a tight loop, exhausting rate limits and producing user-visible flicker.

**Recommended Fix**:
On 401, invalidate the refresh token before returning the error. Add a unit test asserting the cache is cleared on 401.
```

**Output — Gate-5 manifest finding (JSON):**

```json
{
  "reviewer": "team-review/security",
  "concern": "external-tiebreaker",
  "round": 3,
  "gate": "develop-gate5",
  "feature": "020-sprint-skill",
  "findings": [
    {
      "id": "tr-r3-f1",
      "severity": "block",
      "evidence": "Location: app/lib/auth.ts:88 — The 401 branch returns the original error to the caller without invalidating the refresh token. A subsequent retry will use the same expired token and re-fail with 401, producing an infinite loop. [dimension: Security]",
      "claim": "If unaddressed, authentication failure cascades into a tight loop, exhausting rate limits and producing user-visible flicker.",
      "proposed_change": "On 401, invalidate the refresh token before returning the error. Add a unit test asserting the cache is cleared on 401."
    }
  ]
}
```

**Severity decision:** `team-review` `High` → Gate-5 `block` because the finding cites a load-bearing R (R-N: "auth flow handles 401 deterministically"). If the same finding cited only code-quality concerns with no R/AC trace, it would downgrade to `concern`.

**Dedup decision:** `app/lib/auth.ts:88` is also cited by Gate-5 `surgical-reviewer` Round 1 finding `sr-r1-f2`. The Round-3 sharpen merges them: surgical-reviewer's finding survives as primary; the team-review entry is recorded with evidence appending `(also raised by surgical-reviewer sr-r1-f2)`.

---

## T-3 — Add v2.3 → v2.4 MIGRATIONS entry slice

- **Anchors:** AC-8.
- **Acceptance:** `MIGRATIONS.md` v2.3 → v2.4 entry references the bridge doctrine doc.
- **Lane:** Green.
- **Files:** `plugins/specflow/MIGRATIONS.md` (entry slice; consolidated at sprint close).
- **Estimate:** 5 minutes.
