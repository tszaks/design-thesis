# Contributing to the design thesis

This repo is a personal design standard, published so others can read it and give it to their agents. It is not a community style guide. Corrections are welcome. Wholesale changes of taste are not.

By taking part you agree to follow the [Code of Conduct](CODE_OF_CONDUCT.md).

## Before you start

- **Wrong fact, broken link, typo, bad code sample:** open an issue with the "Report an error" template, or send a pull request directly.
- **A rule you disagree with:** open an issue with the "Suggest a change" template and give the reason. Rules here come from shipped products, so evidence from a real screen beats opinion.
- Security problems go through [SECURITY.md](SECURITY.md), not a public issue.

## What is in the repo

| File | What it is |
|---|---|
| `README.md` | The thesis in Tyler's own voice, for people |
| `LLM.md` | The same law rewritten for AI agents |
| `APPLE_NATIVE.md` | Catalog of Apple-native SwiftUI components (EXISTS / USE WHEN / BUILD, iOS 13 to 26) |
| `APPLE_WEB.md` | How Apple builds apple.com and icloud.com, and the ingredients that make a site feel native |
| `APPLE_WEB_APPS.md` | How iCloud and iWork run as desktop-class apps in a browser |

`README.md` and `LLM.md` carry the same rules in two voices. A change to a rule goes into both in the same pull request.

## Style

- Match the voice of the file you are editing. The README is first person and plain.
- Short, specific sentences. No em dashes, no exclamation marks, no emoji, no marketing words.
- A fact needs a source: Apple documentation, a WWDC session, or something you built and tested. Say which OS version a claim applies to.
- Keep catalog entries in the EXISTS / USE WHEN / BUILD shape.
- Update the version line only when the thesis itself changes, and add a line under `[Unreleased]` in [CHANGELOG.md](CHANGELOG.md).

## Pull request checklist

- [ ] Facts are correct and sourced.
- [ ] Links work.
- [ ] Rule changes appear in both `README.md` and `LLM.md`.
- [ ] The change is small and focused.
- [ ] A line added under `[Unreleased]` in the changelog.

## Using AI agents to contribute

Fine, as long as you read and understand what you submit and the checklist above is true. [AGENTS.md](AGENTS.md) tells coding agents how this repository works.

## License

By contributing you agree that your contribution is licensed under the [MIT License](LICENSE).
