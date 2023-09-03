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
# Python
mkdocs serve
```

### format

In order to keep the doc style consistent, we use [Prettier](https://prettier.io/).

```bash
# Node
npm install
npm run prepare
```

See [Prettier Doc: Editor Integration](https://prettier.io/docs/en/editors.html)
to get how to setup your edtiors with prettier to format Markdown files.

Finally commit your changes.
Git commit hook has been added to format Markdown files.

## Admonitions

Read <https://squidfunk.github.io/mkdocs-material/reference/admonitions/> for more details.

## License

All texts are All Rights Reserved by default.
