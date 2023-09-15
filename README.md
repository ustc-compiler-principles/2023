# USTC _Principles and Techniques of Compiler_ 2023 course homepage

Homepage link: <https://ustc-compiler-principles.github.io/2023>

This README is a toturial of how to write and preview the docs.

## Development

First install dependencies and init:

```bash
# It's recommended to install in a virtual environment.
pip install -r requirements.txt
```

Then make your modifications. You can preview your changes by running:

```bash
# In python environment
mkdocs serve
```

### format

In order to keep the doc style consistent, we use [Prettier](https://prettier.io/) and [AutoCorrect](https://github.com/huacnlee/autocorrect).

```bash
# you need Node.js installed first
npm install # then install Prettier and AutoCorrect
```

See [Prettier Doc: Editor Integration](https://prettier.io/docs/en/editors.html) and [AutoCorrect: VS Code Extension](https://github.com/huacnlee/autocorrect#vs-code-extension) for editor intergration.

You are **recommended** to format files before commit:

```bash
# see .prettierignore for ignoring certain files or folders
npm run test    # to see if there is a format error
npm run format  # to format all files under docs/
```

### File Structure

`mkdocs.yml` is the configuration file. When you want to add a new page, you need to add it to `nav` in `mkdocs.yml`.

`docs/` is the directory of all Markdown files. You can create subdirectories to organize your files.

### Admonitions

Read <https://squidfunk.github.io/mkdocs-material/reference/admonitions/> for more details.

## License

All texts are All Rights Reserved by default.
