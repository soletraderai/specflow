# Fixture — graph-invalid cases (one per failure mode)

Each case shows a malformed `Depends on:` graph and the verbatim `GRAPH-INVALID:` diagnostic synthesis emits before any `sprint-bucket: N` is written.

## Case 1 — self-loop

```
### T-1 — Add config knob
- Depends on: T-1
```

Diagnostic emitted to stderr:

```
GRAPH-INVALID: self-loop on T-1
```

## Case 2 — cycle

```
### T-1 — Auth setup
- Depends on: T-3

### T-2 — Token storage
- Depends on: T-1

### T-3 — Token refresh
- Depends on: T-2
```

Diagnostic:

```
GRAPH-INVALID: cycle: T-1 -> T-3 -> T-2 -> T-1
```

(Members listed in traversal order from the entry node `T-1`.)

## Case 3 — duplicate task ID

```
### T-1 — Add badge component
- Depends on: none

### T-1 — Wire badge to header
- Depends on: none
```

Diagnostic:

```
GRAPH-INVALID: duplicate task ID T-1
```

## Case 4 — duplicate dependency edge

```
### T-1 — Setup migration
- Depends on: none

### T-2 — Apply migration
- Depends on: T-1, T-1
```

Diagnostic:

```
GRAPH-INVALID: duplicate edge T-2 -> T-1
```

## Case 5 — dangling reference

```
### T-1 — Add config knob
- Depends on: T-99
```

(T-99 does not exist anywhere in the feature.)

Diagnostic:

```
GRAPH-INVALID: T-1.depends_on references unknown task T-99
```

## Audit signal

Every diagnostic is prefixed `GRAPH-INVALID:` followed by the subtype. Synthesis aborts before any `sprint-bucket: N` is written. The user is pointed to `specflow:scope-change` for legitimate dependency-graph edits.
