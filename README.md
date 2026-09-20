# Minimal Hugo blog

A small, text-first Hugo site with a blog, an About page, RSS, syntax highlighting, build-time math rendering, and automatic GitHub Pages deployment.

## Install Hugo

Install a recent Hugo release (v0.146.0 or newer). On macOS:

```sh
brew install hugo
```

For Linux, Windows, and other installation methods, see the [Hugo installation guide](https://gohugo.io/installation/).

## Work locally

Start the development server:

```sh
hugo server -D
```

Then open `http://localhost:1313/`.

Create a post:

```sh
hugo new content blogs/my-new-post.md
```

Edit the generated file in `content/blogs/`, set `draft: false` when it is ready, and write the post in Markdown.

## Customize the site

- Edit `content/about.md` to change the About page.
- Change `title` and `[params.author]` in `hugo.toml` to change the site name and social links.
- Set `baseURL` in `hugo.toml` to the production URL. The GitHub Actions workflow overrides it automatically with the configured Pages URL.
- Add `math: true` to a post's front matter to enable `\(inline\)` and `$$display$$` mathematics. Hugo renders it to MathML at build time; no client-side JavaScript is used.

## Build for production

```sh
hugo --gc --minify
```

The generated site is written to `public/`.

## Deploy to GitHub Pages

1. Push the repository to GitHub on the `main` branch.
2. In **Settings → Pages**, set **Source** to **GitHub Actions**.
3. Push a commit or run the workflow manually from the Actions tab.

`.github/workflows/hugo.yaml` builds with Hugo 0.166.0, uploads `public/` as a Pages artifact, and deploys it using GitHub's current Pages actions.
