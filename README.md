# Build Zola sites for GitHub Pages

This GitHub Action builds a Zola site using a specific Zola version and uploads
a GitHub Pages artifact. Deployment is handled by GitHub’s official
`actions/deploy-pages` action and does not push commits to a `gh-pages` branch.

## Requirements

- GitHub Pages must be configured to use **GitHub Actions** in the repository settings

## Usage

You can download the `action.yml` file provided here into `.github/workflows`. Additionally, a basic build and deploy job is available below:

```yaml
name: Build and deploy Zola site
on:
  push:
    branches:
      - action-testing

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      ZOLA_VERSION: "latest/edge"
      ZOLA_CHECK: false
      ZOLA_CHECK_FLAGS: ""
    steps:
      - name: Checkout
        uses: actions/checkout@v6
      
      - name: Configure runner environment
        run: sudo snap install zola --channel=${{ env.ZOLA_VERSION }}
      
      - name: Check Zola project
        if: env.ZOLA_CHECK == "true"
        run: zola check ${{ env.ZOLA_CHECK_FLAGS }}

      - name: Build Zola site files
        run: zola build

      - name: Upload Github Pages artifact
        id: deployment
        uses: actions/upload-pages-artifact@v5
        with:
          path: public/
  
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    
    permissions:
      contents: read
      pages: write
      id-token: write
    
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to Github Pages
        id: deployment
        uses: actions/deploy-pages@v5
```

### Options
There are several configuration options available under the build task environment variables:
 - ZOLA_VERSION: Specifies the Snap release to use to build the website. Check [the Official Snap page](https://snapcraft.io/zola) to see available versions.
 - ZOLA_CHECK: Boolean value that specifies whether `zola check` is run before building. Defaults to false.
 - ZOLA_CHECK_FLAGS: Optional flags to be added to `zola check`. Defaults to no flags.
