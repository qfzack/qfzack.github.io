The branch `main` is used to storage all github pages source files.

---
The workflow to update github pages:

1. Generate github pages
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

3. Deploy to branch `gh-pages`:
```bash
hexo deploy
```
