# ai-coding-good-practices

**Example.** Build `orders-report`, a Python CLI that reads an orders CSV and writes daily paid-order counts and totals. The engineer owns the requirements, reviews the diff, and approves the release.

## 1. Define the contract

Specify input columns (`order_id,order_date,status,amount_cents`), output columns (`order_date,paid_order_count,paid_amount_cents`), and failure behavior. Count only paid orders, sort by date, reject invalid rows and duplicate IDs, and preserve the previous report if a run fails before publication. Use integer cents and synthetic fixtures.

> Read the contract and repo conventions. List assumptions that affect outputs or errors. Propose small changes and say how each will be verified. Return the plan before editing.

**Avoid** leaving rounding, duplicate handling, or overwrite behavior for the agent to decide.

## 2. Organize the project

The starter archive includes the files shown below. Add the report module and tests as you implement the contract.

```text
orders-report/
├── README.md
├── CLAUDE.md
├── pyproject.toml
├── .gitignore
├── src/
│   └── orders_report/
│       ├── __init__.py
│       └── cli.py
└── tests/
    └── fixtures/
        ├── orders.csv
        └── expected_report.csv
```

Keep argument parsing and exit codes in `cli.py`. Put validation and aggregation in `src/orders_report/report.py` so they can be tested without launching the CLI. Add `tests/test_report.py` for business rules and `tests/test_cli.py` for the installed command. Declare dependencies and the entry point in `pyproject.toml`. The README should explain how to install, run, and troubleshoot the tool.

**Avoid** hard-coded local paths, import-time file writes, and dependencies that the standard library already covers.

## 3. Implement in small changes

Start with parsing and aggregation. Then add the CLI and safe output publication. For a local destination with one writer, write a temporary file alongside the output and replace the destination only after writing succeeds. Reject paths that point to the input file.

> Implement parsing and aggregation using the standard library. Check the output against the supplied expected CSV. Limit the diff to this behavior and its tests. Flag any change to the contract before implementing it.

**Avoid** mixing unrelated refactors into the patch, silently skipping rows, or truncating the current report before validation finishes.

## 4. Verify before merging

Install in a virtual environment. After implementing the report and tests, run the same checks locally and in CI.

```bash
python -m pip install -e ".[dev]"
git diff --check
ruff check .
ruff format --check .
python -m pytest -q
orders-report --input tests/fixtures/orders.csv --output report.csv
```

Compare the actual CSV rows to `tests/fixtures/expected_report.csv`. Test a late invalid row with an existing destination. Check for a nonzero exit, a clear error, and unchanged destination bytes. Also cover duplicate IDs, pending-only input, and input/output aliases.

> Run the checks. Report each command, its result, and anything you could not verify. If a check fails, find the cause and preserve assertions that express the contract.

**Avoid** accepting “tests passed” without reviewing the assertions, computing expected totals with the implementation under test, or weakening checks to make CI green.

## 5. Review and release

Have a teammate review the diff and verification results. If the script becomes a scheduled job, name an owner, reconcile the first run, and document how to rerun it or restore the previous output.

**Avoid** putting real customer records or credentials into unapproved, or giving the agent production access for development.

## good prompts vs. bad prompts

| Task | Bad prompt | Good prompt |
| --- | --- | --- |
| 1. Plan the change | “Build the whole report.” | “Read the README and fixtures. Propose the smallest changes for parsing, aggregation, and the CLI. List open decisions and one check per change. Wait for review before editing.” |
| 2. Define behavior | “Handle bad data.” | “Reject invalid dates, duplicate IDs, and negative or fractional cents, including on pending rows. Exit nonzero and identify the failing record and field in stderr.” |
| 3. Organize the code | “Clean up the project structure.” | “Keep argument parsing in `cli.py` and validation and aggregation in `report.py`. Put tests under `tests/` and preserve the existing entry point.” |
| 4. Implement a feature | “Make the CSV report work.” | “Use Python’s `csv` module and integer cents. Count paid orders by date and emit rows in ascending date order. Compare against the supplied expected CSV.” |
| 5. Protect existing files | “Make the file writes safe.” | “Write a temporary file beside the output and publish it only after a successful write. Test that a late invalid row leaves the existing report byte-for-byte unchanged.” |
| 6. Write meaningful tests | “Add tests and get coverage up.” | “Use the synthetic fixture and hand-calculated totals. Assert exact output rows, then add tests for duplicate IDs, pending-only input, and input and output pointing to the same file.” |
| 7. Debug a regression | “Fix the wrong total.” | “Reproduce the incorrect November 2 total using the fixture, where the expected total is 1,525 cents. Add a failing regression test, explain the cause, and make the smallest fix.” |
| 8. Refactor safely | “Refactor everything that looks messy.” | “Simplify date validation in `report.py`. Preserve accepted inputs and error behavior. Run the targeted tests before and after, and keep unrelated files out of the diff.” |
| 9. Verify the change | “Make sure it works.” | “Run Ruff and pytest, then run the installed CLI on the fixture and compare its parsed output with the expected CSV. Report the command results and checks you could not run.” |
| 10. Review the PR | “Looks good, ship it.” | “Review the diff against the contract. Focus on miscounted orders, partial writes, and accidental overwrites. Cite the file and triggering input for each finding, and leave the merge decision to the reviewer.” |

## Claude Code skills worth using

| Skill or workflow | When to use it |
| --- | --- |
| [Superpowers](https://github.com/obra/superpowers) | Use `brainstorming` and `writing-plans` for ambiguous work. Use `test-driven-development` for regression fixes, `systematic-debugging` when failures are unexplained, and `verification-before-completion` before opening a PR. |
| [Karpathy Guidelines](https://github.com/multica-ai/andrej-karpathy-skills) | Use as a default for coding tasks. State assumptions, choose a simple design, keep the diff focused, and tie the work to checks you can run. |
| [Project-specific Claude Code skills](https://code.claude.com/docs/en/skills) | If the team repeatedly runs the same verification workflow, put it in `.claude/skills/verify-report/SKILL.md`. Keep stable repo conventions and common commands in `CLAUDE.md`. |
| **Matt Pocock’s `grill-with-docs`** | Before coding, resolve ambiguities such as what counts as a paid order and how duplicate IDs should be handled. Record the decisions against the project’s existing docs. [Skill collection](https://github.com/mattpocock/skills) |
| **OpenSpec `/openspec-propose`** | When changing the CSV schema or CLI contract, draft a proposal, requirements, design, and task list. Have a teammate review how existing callers will be affected before implementing. [OpenSpec](https://github.com/Fission-AI/OpenSpec) |
| **Seth Hobson’s `python-testing-patterns`** | Design pytest fixtures and assertions for exact totals, malformed rows, and file safety. Use real temporary files for CLI tests and mock only the boundary needed to trigger a specific failure. [Python Development plugin](https://github.com/wshobson/agents#quick-start) |
| **ECC (Everything Claude Code)** | Use `python-patterns` for Python conventions, `python-testing` for pytest, and `verification-loop` to repeat checks before handing off a change. [ECC](https://github.com/affaan-m/ECC) |
| **GitHub Spec Kit** | Use `/speckit-specify` and the plan and task steps for a larger feature, such as adding scheduled report delivery. Review the written requirements and plan before implementation. [Claude Code integration](https://github.github.io/spec-kit/integrations/claude.html) |
| **gstack** | Use `/plan-eng-review` to check data flow, failure behavior, and the test plan before coding. Use `/review` for the patch and `/qa` if the project adds a browser interface. [gstack](https://github.com/garrytan/gstack) |
| **Get Shit Done (GSD)** | Use its phase-based planning, implementation, review, and acceptance checks when a project spans multiple sessions or milestones. For example, verify the local CLI before starting work on scheduling. [GSD command reference](https://github.com/gsd-build/get-shit-done/blob/main/docs/COMMANDS.md) |
| **BMad Method** | Explore and document the product idea while the scope is still open. Once a change is agreed, `bmad-build` implements and reviews it. [BMad Method](https://docs.bmad-method.org/) |



### More skills for everyday engineering

| Skill | When to use it |
| --- | --- |
| `/verify` | Exercise the installed CLI with a synthetic CSV and confirm the observable output. Use it alongside pytest. |
| `/run-skill-generator` | Record setup and launch steps if the project needs repeated environment setup or relies on local services. |
| `/code-review` | Inspect a diff for bugs and regressions before teammate review. Ask for a concrete input or scenario for each finding. |
| `/simplify` | Look for duplicated logic and unnecessary abstractions after behavior is covered by tests. It can edit the code, so inspect the resulting diff. |
| `/debug` and `/doctor` | Use `/debug` for Claude Code session problems and `/doctor` for setup checks. For a bug in your Python code, use Superpowers’ `systematic-debugging`. |
| `/batch` | Split a large, repetitive migration into independent changes. Use a normal prompt for a change to one Python script. |
| Superpowers `requesting-code-review` and `receiving-code-review` | Check a patch against the agreed plan and verify review feedback before changing code. |

See the [Claude Code commands reference](https://code.claude.com/docs/en/commands) and the [Superpowers skill list](https://github.com/obra/superpowers#skills-library).



## References

- [Python Packaging User Guide on project configuration](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/) and [src versus flat layout](https://packaging.python.org/en/latest/discussions/src-layout-vs-flat-layout/)
- Python docs for [CSV](https://docs.python.org/3/library/csv.html), [argument parsing](https://docs.python.org/3/library/argparse.html), and [file replacement](https://docs.python.org/3/library/os.html#os.replace)
- [pytest integration practices](https://pytest.org/en/stable/explanation/goodpractices.html), [Ruff linting](https://docs.astral.sh/ruff/linter/), and [Ruff formatting](https://docs.astral.sh/ruff/formatter/)
- [GitHub’s guidance on reviewing AI-generated code](https://docs.github.com/en/copilot/tutorials/review-ai-generated-code) and [OWASP’s secure coding with AI guidance](https://cheatsheetseries.owasp.org/cheatsheets/Secure_Coding_with_AI_Cheat_Sheet.html)

- 
