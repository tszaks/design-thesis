# Design thesis

Documentation only: Tyler Szakacs's design law for his products, in Markdown. There is no code, build or test suite.

This file is for AI agents that **edit** this repo. If you are an agent that **follows** the thesis while building UI, read [LLM.md](LLM.md) instead (add `APPLE_NATIVE.md` for iOS, `APPLE_WEB.md` for web sites, `APPLE_WEB_APPS.md` for web apps).

## Files
- `README.md`: the thesis in Tyler's first-person voice. It is the product. Do not rewrite it.
- `LLM.md`: the same law for machines. Keep it in step with the README.
- `APPLE_NATIVE.md`, `APPLE_WEB.md`, `APPLE_WEB_APPS.md`: the reference catalogs.
- `CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md`, `CHANGELOG.md`, `.github/`: repo housekeeping.

## Conventions
- A rule change goes into `README.md` and `LLM.md` in the same pull request. Update in place, never fork.
- Product-specific exceptions live in that product's own `DESIGN.md`, not here.
- Match the voice of the file you edit. Short, specific sentences. No em dashes, no exclamation marks, no emoji.
- Facts need a source and an OS version. Do not invent hex codes, API names or availability floors.
- Keep catalog entries in the EXISTS / USE WHEN / BUILD shape.
- Add a line under `[Unreleased]` in `CHANGELOG.md` for any change a reader would notice.

## Do not
- Do not restyle the thesis or soften its blocklist.
- Do not apply the thesis to client or marketing sites. The jurisdiction list in the README says where it applies.
- Do not add secrets, personal data or client names.
