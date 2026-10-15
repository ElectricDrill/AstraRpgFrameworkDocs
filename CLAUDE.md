# Astra Framework Docs

## Bootstrap (read first)

Before editing documentation in this repository, read the files in `context/` in this order:

1. `context/README.md`
2. `context/style-guide.md`
3. `context/source-reference.md`

Treat `context/` as the local primer for this docs repo.

## Repository snapshot

- DocFX source lives in `DocFx/`
- Human-written documentation lives in `DocFx/MD/`
- The landing page lives in `DocFx/index.md`
- Navigation is defined in `DocFx/toc.yml` and `DocFx/MD/toc.yml`
- Generated website output lives in `docs/`
- Edit source files in `DocFx/`; do not hand-edit generated output in `docs/`

## Writing conventions

- Conversations with the user happen in **Italian**
- Documentation content is written in **English**
- Follow `context/style-guide.md` strictly for tone, headings, links, images, callouts, code fences, and formatting
- Preserve DocFX-compatible markdown, xrefs, callouts, and relative links
- Prefer linking to existing sections instead of duplicating explanations
- When documenting package behavior, verify names and behavior against the source package referenced in `context/source-reference.md`

## Source code location

The Astra ecosystem packages are located under:

- Windows: `C:\Users\emaci\Documents\AstraPublishing_6_3_8_2D\Packages`
- Ubuntu: `/home/emaci/Documents/AstraPublishing_6_3_2D_URP/Packages`

The package documented in this repository is:

- Windows: `C:\Users\emaci\Documents\AstraPublishing_6_3_8_2D\Packages\com.electricdrill.astra-rpg-framework`
- Ubuntu: `/home/emaci/Documents/AstraPublishing_6_3_2D_URP/Packages/com.electricdrill.astra-rpg-framework`

Use that package as the authoritative source for runtime/editor behavior, asset names, menu paths, and API references.
