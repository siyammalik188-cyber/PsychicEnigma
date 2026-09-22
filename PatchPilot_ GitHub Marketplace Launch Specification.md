# PatchPilot: GitHub Marketplace Launch Specification — Risk Remediation

## Persona-test correction

The original synthetic example must not claim that `fastapi==0.115.0` conflicts with `pydantic==1.10.15`. FastAPI 0.115.0 declares a compatible Pydantic range that includes 1.10.15:

```text
fastapi 0.115.0 depends on pydantic!=1.8,!=1.8.1,<3.0.0,>=1.7.4
```

Therefore, the original fixture is not evidence of a dependency-resolution failure.

### Corrected fixture

Use a genuinely incompatible requirement in the synthetic input:

```text
fastapi==0.115.0
pydantic==1.10.15
pydantic-settings==2.6.1
```

`pydantic-settings==2.6.1` requires Pydantic 2.x, while the application explicitly pins Pydantic 1.10.15. The expected first failure should therefore be based on the resolver output that names this conflict, not on a fabricated FastAPI/Pydantic conflict.

### Corrected expected diagnosis

```text
🔴 ROOT CAUSE: requirements.txt pins pydantic==1.10.15 while pydantic-settings==2.6.1 requires Pydantic 2.x, so pip cannot resolve the dependency set.

📊 CONFIDENCE: High

📋 EVIDENCE: The first real failure is the pip ResolutionImpossible message naming the incompatible Pydantic requirements. The cleanup error is cascading noise because it occurs after pip exits with code 1.

📁 AFFECTED FILES: requirements.txt; .github/workflows/tests.yml

🔧 SUGGESTED FIX: Choose one supported dependency family. For Pydantic 2, remove the 1.10.15 pin and regenerate the lock or constraints file after selecting a compatible Pydantic 2 version. If the application must remain on Pydantic 1, remove or downgrade pydantic-settings to a release that supports Pydantic 1. Verify the selected pair against the project's Python versions before committing.

🧪 RECOMMENDED TEST: In a fresh virtual environment, run `python -m pip install --requirement requirements.txt` and then `python -m pip check` for every supported Python version.

🔁 RECURRING: Yes only if the same normalized resolver error is present in the cited historical runs; otherwise report recurrence as unknown.
```

## Required automated validation

Add a fixture test that fails if the example asserts a conflict for a compatible package pair:

```bash
python -m venv .venv-fixture
. .venv-fixture/bin/activate
python -m pip install --upgrade pip
python -m pip install --requirement requirements.txt
python -m pip check
```

The test fixture must include:

- the complete `requirements.txt` or constraints file;
- the CI Python version;
- the full resolver output, not a hand-written excerpt;
- the exact workflow step and commit diff;
- a clean-environment reproduction command.

## Evidence guardrail

PatchPilot must refuse to name a package conflict unless it can verify that the reported version constraints have an empty intersection or reproduce the resolver failure. If the evidence is incomplete, the diagnosis must explicitly say what is missing rather than inventing a replacement version.

## Merge-risk decision

After replacing the invalid fixture and adding the clean-install validation, merge risk is **MEDIUM** until the corrected resolver output is generated from the actual package files. It becomes **LOW** once the fixture passes the clean-environment install and the expected diagnosis matches the first resolver error.
