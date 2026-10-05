# Run the OSPS Baseline scanner first

Every Security Slam skill starts with a fresh local run of the OSPS Baseline scanner, so the report starts from the scanner's verdicts and inference covers only what the scanner cannot see. The run takes a few seconds per repository. It is the same `openssf/github-repo` plugin the CI options in the `mechanizer` skill run, at the same version, so a local result is what the next CI run will report.

Do this before any other step, every time the skill runs. A result from an earlier session is stale: settings and files change between runs.

## 1. Make sure the tools are installed

```bash
pvtr version
pvtr install openssf/github-repo
```

- If `pvtr` is missing, show the user the install command and run it once they agree: `brew install privateerproj/tap/pvtr`, or the install script in the [pvtr README](https://github.com/privateerproj/pvtr#readme), which puts the binary in `~/.privateer/bin`. Installing a CLI changes their machine, so ask first.
- `pvtr install openssf/github-repo` installs the scanner plugin from grc.store, or updates it to the latest signed release, and verifies the signature before writing anything. Run it every time: it keeps the local scanner at the version the hub runs, and it is quick when nothing changed.
- `gh` must be signed in (`gh auth status`) and `yq` must be on the `PATH`.

## 2. Write the config in a temporary directory

The config carries a GitHub token, so it never goes in the repository. The token is the user's own `gh` token; the scanner only reads with it. Never print the token, never echo the config, and never ask the user to paste a token.

Write one target per repository: the current one, plus every `project.repositories[].url` in the Security Insights file (`find . -maxdepth 2 -iname 'security-insights.y*ml'`). Set `applicability` to the project's confirmed maturity level, or `maturity-1` when the skill has no level yet.

```bash
dir=$(mktemp -d)
cat > "$dir/config.yml" <<CONFIG
targets:
  skills:
    plugin: openssf/github-repo
    policy:
      catalogs:
        - osps-baseline
      applicability:
        - maturity-1
    vars:
      owner: security-slam
      repo: skills
      token: $(gh auth token)
  website:
    plugin: openssf/github-repo
    policy:
      catalogs:
        - osps-baseline
      applicability:
        - maturity-1
    vars:
      owner: security-slam
      repo: website
      token: $(gh auth token)
CONFIG
```

The target name (`skills`, `website`) names the results folder. Keep it to the repository name.

## 3. Run

```bash
(cd "$dir" && pvtr run -c config.yml --silent)
```

Run from inside the temporary directory: pvtr 0.23 writes `evaluation_results/` into the current directory and ignores `--write-directory` and `--output`. Exit code 1 means at least one target has a `Failed` result. That is an honest scan, not a broken run. A run that stops before any control is logged is a token or API problem: check `gh auth status` and run it once more.

## 4. Read the results

One file per target, `$dir/evaluation_results/TARGET/TARGET.yaml`. Flatten it to the controls at the chosen level:

```bash
level=1
yq -r '.evaluation-suites[].control-evaluations.evaluations[].assessment-logs[]
       | select(.applicability[] == "maturity-'"$level"'")
       | [.requirement.entry-id, .result, .message] | @tsv' \
  "$dir/evaluation_results/TARGET/TARGET.yaml" | sort
```

Record the scanner version from `.plugin-version` in the report. Then sort the controls:

| Result | Meaning | What the skill does |
| --- | --- | --- |
| `Passed` | The scanner verified the control | Met. The message is the evidence. |
| `Failed` | The scanner found the control unmet | A gap. The message names the cause, often a Security Insights field or a setting. |
| `Needs Review` | The scanner looked and could not decide | Read the message: it says what it saw and what to check by hand. |
| `Not Run`, message says the identifier is retired | Not in the 2026-08-28 Baseline | Skip it. |
| `Not Run`, empty message | The scanner has no check for this control | Audit by hand. This is where the skill's own judgement goes. |

The scanner reads the Security Insights file for several controls (DO-01.01, DO-02.01, GV-03.01, QA-04.01, VM-02.01, and others at higher levels), so a `Failed` message that names a Security Insights declaration is fixed by adding the field, not by writing prose.

The maintainer's token sees more than the job token in CI: branch protection (AC-03.01), secret scanning (BR-07.01), and other repository settings pass or fail locally and come back `Needs Review` from the publish workflow. A local `Passed` is real evidence. The published result catches up once the `defender` skill records the setting in Security Insights or the scan runs with a broader token.

## 5. Clean up

```bash
rm -rf "$dir"
```

The config holds the token. Remove the directory when the skill is done, and never copy `evaluation_results/` into the repository.
