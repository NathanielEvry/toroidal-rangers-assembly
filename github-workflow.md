name: Toroidal Rangers CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  markdown-lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '16'
          
      - name: Install markdownlint
        run: npm install -g markdownlint-cli
        
      - name: Run markdownlint
        run: markdownlint '**/*.md' --ignore node_modules
        
  link-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '16'
          
      - name: Install markdown-link-check
        run: npm install -g markdown-link-check
        
      - name: Check links in markdown files
        run: find . -name "*.md" -not -path "./node_modules/*" | xargs -n1 markdown-link-check
        
  deploy-gitbook:
    needs: [markdown-lint, link-check]
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '16'
          
      - name: Install GitBook CLI
        run: npm install -g gitbook-cli
        
      - name: Setup GitBook
        run: |
          gitbook install
          gitbook build
          
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./_book
