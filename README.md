# stack-base-python-uv

A Python base project template using `uv` as the package manager, following the stackable-specs methodology.

## Included Specs

| Spec | Layer | Why it's essential |
| ---- | ----- | ------------------ |
| `language/python.md` | language | Python language conventions, typing, idioms |
| `platform/uv.md` | platform | uv package manager workflow and constraints |
| `practices/madr.md` | practices | ADR format and lifecycle |
| `practices/bdr.md` | practices | Behavior record format and lifecycle |
| `practices/conventional-commits.md` | practices | Commit message contract |
| `practices/tdd.md` | practices | Red-green-refactor discipline |
| `practices/git.md` | practices | Branch and merge workflow |
| `quality/unit-testing.md` | quality | Unit-test scope and naming rules |
| `security/dependency-management.md` | security | Dependency policy |
| `delivery/docker.md` | delivery | Container image conventions |
| `delivery/github-actions.md` | delivery | CI/CD pipeline conventions |

## Repository layout

```
.
├── .github/
│   └── workflows/          # CI/CD pipelines
├── docs/
│   ├── adr/                # Architectural Decision Records (MADR format)
│   ├── bdr/                # Behavior Decision Records
│   └── specs/              # Specs from stackable-specs
│       ├── delivery/
│       ├── language/
│       ├── platform/
│       ├── practices/
│       ├── quality/
│       └── security/
├── src/                    # Application source code
├── tests/                  # Automated tests
├── verify/                 # Smoke / post-deploy verification scripts
├── .gitignore
├── pyproject.toml          # Project metadata (PEP 621)
├── uv.lock                 # Locked dependencies
├── .python-version         # Pinned Python version
└── README.md
```

## Getting started

This project was initialized with `uv init` and follows the stackable-specs layered specification model.

1. Install dependencies: `uv sync`
2. Run the application: `uv run main.py`
3. Run tests: `uv run pytest`

## References

- [stackable-specs/specs](https://github.com/stackable-specs/specs) — Source of the specification files