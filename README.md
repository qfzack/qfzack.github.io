> - The branch `main` is used to storage all github pages source files.
> - The branch `gh-pages` is used to storage all generated static files.

## 1. Update GitHub pages static files

**Update manually in local**

1. Generate GitHub pages
    - Remove generated files and cache:
        ```bash
        hexo clean
        ```
    - Generate static files:
        ```bash
        hexo generate
        ```

2. Run local server:
```bash
hexo server
```

1. Deploy to branch `gh-pages`:
```bash
hexo deploy
```

**Update with GitHub Actions**

Workflow [hexo-deploy.yml](.github/workflows/hexo-deploy.yml) will be executed when push to branch `main`, and then generate static files and push to branch `gh-pages`.

## 2. Deploy GitHub pages

After the static files updated to branch `gh-pages`, another workflow [pages-build-deployment](https://github.com/qfzack/qfzack.github.io/actions/workflows/pages/pages-build-deployment) will be executed to deploy them to GitHub pages.
