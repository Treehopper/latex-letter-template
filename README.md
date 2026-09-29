# LaTeX Letter Template

A template repository for writing letters in LaTeX with private data kept out of Git.

- The repository contains the letter and **dummy data** only.
- Your **real data** lives in a local `data.tex` that is ignored by Git.
- A **GitHub Actions workflow** builds a PDF (with dummy data) and attaches it to a release whenever you push a tag, which is useful as a layout preview.
- The **real letter** is built locally on your machine.

## Repository structure

```
.
├── letter.tex                    # The letter; uses macros instead of personal data
├── data.example.tex              # Dummy values (committed)
├── data.tex                      # Real values (local only, ignored by Git)
├── signature.png                 # Optional signature image (local only, ignored by Git)
├── .gitignore
└── .github/workflows/release.yml # Builds the PDF and creates a release
```

## How the private data works

All personal details (names, addresses, IBAN, case numbers, dates, amounts) are defined as macros. The letter only ever uses the macros, e.g. `\RecipientName` instead of the actual name.

`letter.tex` loads the real data if it exists, then loads the dummy data as a fallback:

```latex
\InputIfFileExists{data.tex}{}{}
\input{data.example.tex}
```

`data.example.tex` defines every macro with `\providecommand`, which only takes effect if the macro isn't already defined:

```latex
\providecommand{\SenderName}{XXX Sender Name XXX}
\providecommand{\SenderAddress}{XXX Street 1\\12345 City XXX}
\providecommand{\RecipientName}{XXX Recipient Name XXX}
\providecommand{\RecipientAddress}{XXX Street 2\\12345 City XXX}
\providecommand{\IBAN}{XXX DE00 0000 0000 0000 0000 00 XXX}
```

`data.tex` uses the same macro names with real values and plain `\newcommand`:

```latex
\newcommand{\SenderName}{Erika Beispiel}
% ...
```

Result:

| Where | `data.tex` present? | Output |
| --- | --- | --- |
| Your machine | Yes | Real letter |
| GitHub Actions | No | Layout preview with dummy data |

If `data.tex` is missing a macro, the dummy value is used instead of failing the build. The `XXX` markers make such leftovers easy to spot, so **check the PDF before sending**.

## Getting started

1. Click **Use this template** on GitHub and create a **private** repository.
2. Clone it locally.
3. Create your real data file:
   ```sh
   cp data.example.tex data.tex
   ```
4. In `data.tex`, replace `\providecommand` with `\newcommand` and fill in the real values.
5. Confirm Git ignores it:
   ```sh
   git check-ignore -v data.tex
   ```
   This must print the matching `.gitignore` line. If it prints nothing, **do not commit**.

## Building locally

Requires a TeX distribution (TeX Live or MiKTeX) with `latexmk`.

```sh
latexmk -pdf letter.tex
```

Clean up auxiliary files:

```sh
latexmk -c
```

## Building on GitHub (release)

`.github/workflows/release.yml`:

```yaml
name: Build letter
on:
  push:
    tags: ['v*']
permissions:
  contents: write
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: xu-cheng/latex-action@v3
        with:
          root_file: letter.tex
      - uses: softprops/action-gh-release@v2
        with:
          files: letter.pdf
```

Create a release:

```sh
git tag v1.0
git push --tags
```

The PDF appears under **Releases**. It always contains dummy data, since `data.tex` never reaches GitHub.

### GitHub Free plan

This works on the free plan. Private repositories include 2,000 Linux Actions minutes per month; a letter build takes roughly 1–2 minutes. Keep `runs-on: ubuntu-latest`, because Windows and macOS runners use up included minutes 2× and 10× faster. Releases in private repositories are only visible to you and your collaborators.

## Privacy notes

- **Private repos are not end-to-end encrypted.** GitHub (Microsoft) treats them as confidential and only accesses them for support with your consent, for security, or when legally required, but it holds the keys.
- **Actions runners see everything in the repo.** That's why the real data never goes there.
- **Don't store real data as a repository secret** to get real PDFs from CI. That hands the data back to GitHub.
- **Git history is permanent.** If `data.tex` is committed even once and pushed, deleting it later does not remove it. Treat the data as exposed and rewrite history (e.g. `git filter-repo`) if that happens.
- **AI assistants.** If you use an assistant to edit the letter, share only the repository contents (dummy data), not `data.tex`.

## Tips

- Put **every** personal detail in a macro, including details in the body text such as reference numbers, dates and amounts.
- Keep dummy values similar in length to real ones, so the preview reflects real line breaks and page count.
- For German letters, the KOMA-Script class `scrlttr2` handles DIN-compliant address fields and folding marks. It is included in the TeX Live image used by the workflow.
- Put a scanned signature in `signature.png` to have it printed above your name. Without it, the letter leaves space for a handwritten signature.
- Run `git status` before every commit and make sure `data.tex` is not listed.

## `.gitignore`

```
# Private data
data.tex
signature.png

# LaTeX build files
*.aux
*.log
*.out
*.fls
*.fdb_latexmk
*.synctex.gz
*.pdf
```
