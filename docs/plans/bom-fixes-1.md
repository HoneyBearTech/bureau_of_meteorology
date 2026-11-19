# Plan: bom-fixes-1

## What Sean asked for
"Start the next repo on the shortlist" (bremor/bureau_of_meteorology), respecting the project's contribution rules: fix real bugs and send each fix to the original project as its own pull request.

## Contribution rules
- **Written rules:** there is no CONTRIBUTING.md, pull request template, code of conduct or AI policy, in this repository or at the owner's account level. `DEVELOPMENT.md` covers releasing only: run `./scripts/test` and `./scripts/lint`, and releases are the maintainer's job.
- **Release notes:** these are built from pull request titles, so titles must start with `fix(...)`.
- **What the maintainer asks for in practice:**
  - Link the issue (`fixes #N`).
  - Include tests.
  - Respond to the automatic code review's findings.
  - Keep one pull request per problem.
  - Rebase when asked.
- **What we'll follow:** all of the above. We won't touch `manifest.json`'s version, and we won't reformat code we didn't change.

## What we found
- **Three real bugs:** each reproduced on the current code, with no open or recent pull request covering it.
- **#262 doesn't reproduce:** the text-length problem comes from the reporter's own template, not this integration.
- **#238 is a bigger, riskier change:** the integration deletes its own sensors from Home Assistant's registry on reload. It's left for a later batch.

## What we'll change
1. **#284 UV forecast crash:** when the weather service sends a UV start time but no end time, the UV forecast sensor crashes on every update. Show "unknown" until both times arrive.
   - **File:** `custom_components/bureau_of_meteorology/sensor.py`
   - **Check:** a new test that crashes on the current code and passes with the fix; the live weather service's normal UV text is unchanged.
2. **#282 no recovery after starting offline:** if Home Assistant starts while the internet is down, the integration loads with no data and its sensors never appear until it's reloaded by hand. Tell Home Assistant setup isn't ready yet, so it retries by itself.
   - **File:** `custom_components/bureau_of_meteorology/__init__.py`
   - **Check:**
     - a new test that fails on the current code;
     - a real Home Assistant run: offline it waits and retries, and once the network is back all sensors load by themselves;
     - it must still work on Home Assistant 2023.9, the oldest version the project supports.
3. **#287 "Rain Since 9Am":** the sensor name capitalises the letter after a digit. Use a capitalisation that leaves "9am" alone.
   - **File:** `custom_components/bureau_of_meteorology/sensor.py`
   - **Check:** a new test; no other sensor's name or ID changes.
4. **For all three:**
   - The project's tests (`./scripts/test`) pass on Python 3.11 with Home Assistant 2023.9, as the project's CI uses.
   - The style checker (`ruff`) finds no new problems in the changed files.
   - Each fix is one commit that applies on its own to the latest `main`.
5. **Send upstream, only after Sean's final approval on this PR:**
   - Each fix gets its own clean branch from the original project's latest `main`, holding only that fix. The plan file never goes upstream.
   - Sean opens one pull request per fix to bremor/bureau_of_meteorology from his own account, each linking its issue. The full title and description of each are shown here first.
   - Draft titles:
     - "fix(sensor): handle missing UV end time in UV forecast summary (fixes #284)"
     - "fix(setup): retry setup when BOM cannot be reached (fixes #282)"
     - "fix(sensor): keep "9am" lowercase in observation sensor names (fixes #287)"

## Who's on it
Researcher (done), Engineer, QA, Security Officer, Auditor.

## Risks
- **Overlap with the maintainer's draft #291:** it touches the same areas for #284 and #282. He has said he'd split it into smaller pieces, so small separate fixes should suit him, but he may prefer to fold them in.
- **#282 may keep retrying:** if the weather service ever stops serving a location for good, setup keeps retrying instead of loading half-broken.
- **Automatic review:** an automatic reviewer comments on every pull request, and the maintainer expects replies. We'll handle those after submission.

## Out of scope
- #238 (registry clean-up on reload): a later batch.
- #262 (doesn't reproduce).
- Feature requests, the `codeowners` typo, and the misleading retry log line.
- Running `./scripts/lint` across the whole project, which would rewrite 12 unrelated files.
