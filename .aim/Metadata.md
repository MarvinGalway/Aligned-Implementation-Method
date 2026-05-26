# AIM Metadata Block — Full Specification

> **This file is a human reference document.** It is NOT loaded into the agent's context.
> The operative rules (fields, identifiers, validation) are in `.aim/PROTOCOL.md` under "Metadata Block".
> This file provides extended examples, language conventions, and KG consumption details
> for developers who need to look up the spec while writing code.

---

## Format

The Metadata block lives inside the docstring of an annotated artifact, after the prose summary and any rationale section. It uses a fixed structure:

```python
def issue_refresh_token(payload: RefreshRequest) -> TokenResponse:
    """
    Issue a new access token from a valid refresh token.

    Rationale:
        Refresh token rotation chosen over long-lived tokens to limit
        blast radius if a refresh token leaks.

    Metadata:
        phases: ["Phase_2_RefreshToken", "Phase_5_Optimizations"]
        requirements: ["REQ-03"]
        acceptance_criteria: ["AC-07", "AC-09"]
        contracts: ["RefreshRequest", "TokenResponse"]
        tests: ["tests/auth/test_refresh.py::test_rotation_invalidates_old"]
    """
```

The block is intentionally parseable by both deterministic parsers (Tree-sitter via GitNexus) and LLM-mediated extraction (Graphify). It uses YAML-like syntax inside the docstring — list values in square brackets, string values quoted.

---

## Fields

| Field | Type | Required | Purpose |
|---|---|---|---|
| `phases` | list of strings | yes | Phases in which this artifact was introduced or had meaningful behavioral change |
| `requirements` | list of strings | yes (≥1) | Upstream requirements this artifact contributes to |
| `acceptance_criteria` | list of strings | yes (≥1) | ACs this artifact directly satisfies or contributes to |
| `contracts` | list of strings | for service functions | DTOs / interfaces this artifact consumes or produces |
| `tests` | list of strings | for service functions | Test paths that prove this artifact's correctness |

### Identifier formats

- **Phases:** `"Phase_N_SlugName"` — numeric prefix for stable ordering, slug for human readability. Example: `"Phase_2_RefreshToken"`. The slug must match the phase title in `.ai/phase-N.md`.
- **Requirements:** `"REQ-NN"` — zero-padded two-digit numeric. Example: `"REQ-03"`.
- **Acceptance Criteria:** `"AC-NN"` — zero-padded two-digit numeric. Example: `"AC-07"`.
- **Contracts:** the exact class or type name as defined in the contracts layer.
- **Tests:** path relative to project root, with `::` separating file from test identifier when targeting a specific test.

### Phase list semantics

The `phases` list represents **meaningful contribution**, not mere modification. A phase entry is added when:
- The artifact was introduced in that phase, OR
- Its behavior (not formatting, not signature-preserving refactoring) was meaningfully changed in that phase

Trivial refactors do not add phase entries. This keeps queries like "what did Phase 5 actually change?" useful instead of noisy.

---

## Annotation targets

The Metadata block is required on:

1. **Public functions and methods** in the service / business logic layer
2. **DTOs, contracts, and interfaces** in the contracts layer
3. **Domain entities** in the domain layer
4. **Test functions or test classes** (with `tests` field omitted; the test itself proves the AC, so the `acceptance_criteria` list declares what it proves)
5. **Module headers** (top-of-file docstring summarizing phases and ACs covered by the module)

The Metadata block is **not** required on:

- Private helpers (they inherit context from their public caller)
- Generated code
- Pure plumbing (e.g., `__init__.py` re-exports)

---

## Language conventions

The Metadata block format is the same across languages; only the surrounding docstring syntax changes.

| Language | Container |
|---|---|
| Python | Triple-quoted docstring |
| TypeScript / JavaScript | JSDoc `/** ... */` block above the declaration |
| Java / Kotlin | Javadoc `/** ... */` block above the declaration |
| Go | Doc comment (`//` lines) immediately above the declaration |
| Rust | Doc comment (`///` lines) immediately above the declaration |
| C# | XML doc comment (`///`) above the declaration |

In `//`-style comment languages, the `Metadata:` block lives inside the doc comment, with each line prefixed by the comment marker. The structured content is identical.

---

## Example: full service function (Python)

```python
def issue_refresh_token(payload: RefreshRequest) -> TokenResponse:
    """
    Issue a new access token from a valid refresh token.

    Validates the refresh token's signature and expiration, then issues
    a new access token with rotated refresh token. The old refresh token
    is invalidated to prevent replay attacks.

    Rationale:
        Refresh token rotation chosen over long-lived tokens to limit
        blast radius if a refresh token leaks. Trade-off: every refresh
        is a write, increasing DB load — acceptable given expected
        refresh frequency from REQ-12 NFRs.

    Metadata:
        phases: ["Phase_2_RefreshToken", "Phase_5_Optimizations"]
        requirements: ["REQ-03", "REQ-12"]
        acceptance_criteria: ["AC-07", "AC-09"]
        contracts: ["RefreshRequest", "TokenResponse"]
        tests: [
            "tests/auth/test_refresh.py::test_rotation_invalidates_old",
            "tests/auth/test_refresh.py::test_expired_token_rejected"
        ]
    """
```

## Example: test function (Python)

```python
def test_rotation_invalidates_old():
    """
    Verify that issuing a new refresh token invalidates the previous one.

    Metadata:
        phases: ["Phase_2_RefreshToken"]
        requirements: ["REQ-03"]
        acceptance_criteria: ["AC-07"]
    """
```

## Example: DTO (Python)

```python
@dataclass
class RefreshRequest:
    """
    Payload for the refresh token endpoint.

    Metadata:
        phases: ["Phase_2_RefreshToken"]
        requirements: ["REQ-03"]
        acceptance_criteria: ["AC-07", "AC-09"]
    """
    refresh_token: str
```

## Example: TypeScript service function (JSDoc)

```typescript
/**
 * Issue a new access token from a valid refresh token.
 *
 * Rationale:
 *   Refresh token rotation limits blast radius if a token leaks.
 *
 * Metadata:
 *   phases: ["Phase_2_RefreshToken"]
 *   requirements: ["REQ-03"]
 *   acceptance_criteria: ["AC-07", "AC-09"]
 *   contracts: ["RefreshRequest", "TokenResponse"]
 *   tests: ["tests/auth/refresh.test.ts::rotates and invalidates old token"]
 */
export function issueRefreshToken(payload: RefreshRequest): TokenResponse {
```

## Example: Go service function

```go
// IssueRefreshToken issues a new access token from a valid refresh token.
//
// Rationale:
//   Refresh token rotation limits blast radius if a token leaks.
//
// Metadata:
//   phases: ["Phase_2_RefreshToken"]
//   requirements: ["REQ-03"]
//   acceptance_criteria: ["AC-07", "AC-09"]
//   contracts: ["RefreshRequest", "TokenResponse"]
//   tests: ["auth/refresh_test.go::TestRotationInvalidatesOld"]
func IssueRefreshToken(payload RefreshRequest) (TokenResponse, error) {
```

---

## How knowledge graph tools use the block

**GitNexus** parses the docstring via Tree-sitter, then a small post-processor extracts the `Metadata:` block as structured edges in the graph. Each list item becomes an edge from the function node to a node representing the phase, requirement, AC, contract, or test. Cypher queries can then answer:

```cypher
// Which functions cover AC-07?
MATCH (f:Function)-[:COVERS]->(ac:AC {id: "AC-07"}) RETURN f

// Which ACs have no test coverage?
MATCH (ac:AC) WHERE NOT (ac)<-[:PROVES]-(:Test) RETURN ac

// What does Phase 5 actually change?
MATCH (f:Function)-[:IN_PHASE]->(p:Phase {id: "Phase_5_Optimizations"}) RETURN f
```

**Graphify** picks up the Metadata block through its LLM-mediated extraction and produces concept-level edges with confidence tags. It also extracts the prose `Rationale:` section as `rationale_for` edges pointing to the concept the rationale explains — this is the value-add over GitNexus alone, useful when requirements include design rationale that needs to be queryable.

Both tools share the same anchor identifiers (`AC-07`, `Phase_2_RefreshToken`, `REQ-03`). Cross-graph reconciliation reduces to: do both graphs return the same set of functions when queried for the same identifier?