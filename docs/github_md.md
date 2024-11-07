Here's the updated version with the structure preserved and language improvements:
# Creating a GitHub Page for Your Markdown Documents Using MkDocs

Making a GitHub Page for your markdown documents is easy with `mkdocs`. Essentially, it involves moving all markdown documents to the `docs` folder and creating an `mkdocs.yml` file (see below). The full list of required commands is as follows:

```bash
pip install mkdocs

# This command is optional. It simply creates the docs folder and mkdocs.yml file, which can be done manually.
mkdocs new .

# Serve locally at http://127.0.0.1:8000 to check that everything is running smoothly.
mkdocs serve

# Deploy to GitHub Pages. This should be run after every change to markdown files in the docs folder or mkdocs.yml. You can automate this with a GitHub workflow (see below).
mkdocs gh-deploy
```

Move all markdown files to the `docs` folder and create the `mkdocs.yml` file in the root of the repository. If `README.md` is important, it should be renamed to `index.md`. Alternatively, running `mkdocs new .` will automatically create the `mkdocs.yml` file and `docs` folder with an `index.md` file.

### Example Repository Structure

```plaintext
.
├── docs/
│   ├── README.md
│   ├── cheatsheet.md
│   └── Tutorial/
│       └── iframe.md
├── mkdocs.yml
```

### Configuring mkdocs.yml

In `mkdocs.yml`, add the following configuration. If you're using the `material` theme, it should be installed with `pip install mkdocs-material`.

```yaml
site_name: "Web App Tutorial"
nav:
  - Home: README.md
  - Cheatsheet: cheatsheet.md
  - Tutorial:
      - Iframe: tutorial/iframe.md
theme:
  name: material  # Use 'readthedocs' or other themes if preferred
```

---

## GitHub Workflow

Without a workflow, you need to run `mkdocs gh-deploy` each time you change the documents. To automate this, create a workflow file at `.github/workflows/deploy.yml`.

```yaml
name: Deploy MkDocs Site

on:
  push:
    paths:
      - 'docs/**'
      - 'mkdocs.yml'

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.x'

    - name: Install dependencies
      run: |
        pip install mkdocs
        pip install mkdocs-material  # Only if using the Material theme

    - name: Deploy to GitHub Pages
      run: |
        mkdocs gh-deploy --force
```

### Workflow for Each Push

If you want the workflow to trigger on every push, regardless of whether `docs/` or `mkdocs.yml` is changed, modify the `on:` section:

```yaml
on:
  push:
    branches:
      - main
```

This replaces the following:

```yaml
on:
  push:
    paths:
      - 'docs/**'
      - 'mkdocs.yml'
```

### No Separate Commit for Workflow

You don't need a separate commit for rendered HTML, as `mkdocs gh-deploy` handles it for you.

```bash
    - name: Commit and Push Rendered HTML
      run: |
        git config --local user.email "fkariminejadasl@gmail.com"
        git config --local user.name "Fatemeh Karimi Nejadasl"
        git add -f *.html
        git commit -m "Generate HTML" -a || echo "No changes to commit"
        git push
```

### When You Might Need to Explicitly Use `GITHUB_TOKEN`

You only need to explicitly reference `GITHUB_TOKEN` if:

- You're passing it to a custom script or command that requires it.
- You're passing it to a different environment or third-party service that needs authentication.

In most basic workflows, like deploying with `mkdocs gh-deploy`, GitHub handles `GITHUB_TOKEN` automatically. This is why it works without setting it explicitly.

```bash
    - name: Deploy to GitHub Pages
      run: |
        mkdocs gh-deploy --force
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## `mkdocs build` vs. `mkdocs gh-deploy`

- **`mkdocs build`**: Used to build the documentation site locally. After running this, the generated site is placed in a directory called `site/` within your project folder.
- **`mkdocs gh-deploy`**: Builds the site (similar to `mkdocs build`), but also pushes the generated files to the `gh-pages` branch of your GitHub repository, which is used by GitHub Pages to serve the site.

### When to Use Each

- Use **`mkdocs gh-deploy`** when deploying to GitHub Pages.
- Use **`mkdocs build`** when generating the site files locally for testing or if you're deploying the files manually to another server.

---

## Using a Custom Domain (CNAME)

If you'd like to use a custom domain (e.g., `www.yourdomain.com`) for your GitHub Pages site, you can configure it with a **CNAME file**.

### Steps for Using a CNAME with GitHub Pages

1. **Create a CNAME File**:
   - In your repository (inside the `docs/` folder or the root, depending on your project structure), create a file called `CNAME` (without an extension).
   - Inside the `CNAME` file, write your custom domain name (e.g., `www.yourdomain.com`).

   Example content of `CNAME`:
   ```
   www.yourdomain.com
   ```

2. **GitHub Pages Settings**:
   - Go to your repository’s settings.
   - Scroll down to the **"GitHub Pages"** section.
   - In the **Custom domain** field, enter your domain (e.g., `www.yourdomain.com`).

3. **DNS Configuration**:
   - In your domain registrar's settings, create a **CNAME record** that points your custom domain (e.g., `www.yourdomain.com`) to `<your-github-username>.github.io`.
   - Example of CNAME record in DNS settings:
     - **Name**: `www`
     - **Type**: CNAME
     - **Value**: `<your-github-username>.github.io`

Once the DNS settings propagate (which can take a few hours), your GitHub Pages site will be accessible via your custom domain.

---

## Recap of Steps for Deploying MkDocs with GitHub Pages

1. Run `mkdocs gh-deploy` to build and deploy your documentation:

```bash
mkdocs gh-deploy
```

2. If you're using a custom domain, create a `CNAME` file containing your domain name (e.g., `www.yourdomain.com`) and configure your domain’s DNS settings.

3. GitHub Pages will now serve your MkDocs site at either `https://<your-username>.github.io/<repository-name>` or your custom domain, if configured.

