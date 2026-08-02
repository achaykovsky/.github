## Summary

<!-- What changed and why -->

## Test plan

- [ ] Job `name:` values and ruleset `context` entries match [docs/REQUIRED_CHECKS.md](docs/REQUIRED_CHECKS.md) and [rulesets/main-protection.json](rulesets/main-protection.json)
- [ ] [docs/ADOPT.md](docs/ADOPT.md) and [README.md](README.md) updated if workflow inputs, check names, or caller contract changed
- [ ] No duplicate policy added across [.github/copilot-instructions.md](.github/copilot-instructions.md) and `.github/instructions/*.instructions.md` (link to the canonical file instead)
- [ ] Ruleset JSON validates; `bypass_actors` remains `[]` unless explicitly justified
- [ ] Required checks green on this PR before merge
