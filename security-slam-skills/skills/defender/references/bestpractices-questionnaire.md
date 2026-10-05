# Optional: the bestpractices.dev OSPS Baseline Questionnaire

The Defender badge no longer uses bestpractices.dev. Offer this only when the user wants a bestpractices.dev Baseline badge as well. It reuses the gap analysis, so it costs little extra.

## Prepare the questionnaire

1. Tell the user to sign in at [bestpractices.dev](https://www.bestpractices.dev), add the project if needed, and open the OSPS Baseline section. The site autodetects some answers. It needs repo admin access, so the user submits. Never claim you submitted it.
2. Produce a copy-ready answer sheet: one line per control with status and the evidence URL for the justification field.
   Check every evidence URL before it goes on the sheet. Anything other than `200` gets fixed or replaced, never submitted:

   ```bash
   curl -sL -o /dev/null -w '%{http_code} %{url_effective}\n' URL
   ```

   For a URL with a `#fragment`, also confirm the target page has that heading. Wrong: a justification that cites a docs page the project moved or deleted. Reviewers follow these links, and a dead one reads as an unmet control.
3. Once the project exists, also give the user a prefilled link that loads every answer into the form for review: `https://www.bestpractices.dev/en/projects/PROJECT_ID/baseline-1/edit?osps_ac_01_01_status=Met&osps_ac_01_01_justification=...`, one `osps_<family>_<nn>_<nn>_status` and `_justification` pair per control, URL-encoded. Open it in the user's browser with `open` (macOS) or `xdg-open` rather than asking them to copy a long URL from the terminal.
4. Warn that the site's autodetector marks OSPS-DO-01.01 Unmet when the user guide is a README instead of a docs folder, and can override a prefilled answer. Tell the user to set it to Met by hand with the README justification.
5. The user saves progress and returns as needed.

## Display the badge

Once the questionnaire shows the level as met, the user can add the badge to the README. Link the badge to the level page, not the project root: `/projects/PROJECT_ID` redirects to the metal-series page, which shows "in progress" for a project that only holds a Baseline badge.

Correct:

```markdown
[![OpenSSF Best Practices](https://www.bestpractices.dev/projects/PROJECT_ID/baseline)](https://www.bestpractices.dev/projects/PROJECT_ID/baseline-1)
```

Wrong - the link lands on the metal-series page:

```markdown
[![OpenSSF Best Practices](https://www.bestpractices.dev/projects/PROJECT_ID/baseline)](https://www.bestpractices.dev/projects/PROJECT_ID)
```

Add the level page to Security Insights as a third-party assessment:

```yaml
repository:
  security:
    assessments:
      third-party:
        - name: OpenSSF Best Practices OSPS Baseline
          evidence: https://www.bestpractices.dev/projects/PROJECT_ID/baseline-1
          comment: OSPS Baseline Level 1 questionnaire, self-reported and autodetected.
```

An existing "metal" badge (passing, silver, gold) stays. The Baseline badge sits alongside it.
