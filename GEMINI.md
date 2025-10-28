# Project Overview

This is a personal blog created with the [Hugo](https://gohugo.io/) static site generator. The content is written in Markdown and the site is styled using the "no-style-please" theme, with the "PaperMod" theme also available in the project.

The website is hosted on GitHub Pages at [https://fwrq41251.github.io/](https://fwrq41251.github.io/).

## Building and Running

To run the website locally for development, you can use the following Hugo command. This will start a local server with live reloading enabled.

```bash
hugo server
```

To build the static site for production, run:

```bash
hugo
```

The generated static files will be placed in the `public/` directory.

## Development Conventions

### Content Creation

New blog posts and pages are created as Markdown files (`.md`) in the `content/` directory. You can create a new post by running:

```bash
hugo new posts/my-new-post.md
```

This will create a new Markdown file with a pre-configured front matter.

### Themes

The project is currently configured to use the "no-style-please" theme. Theme files are located in the `themes/no-style-please/` directory. Configuration for the theme can be found in `hugo.toml` and in the theme's own `config.toml` file.

The "PaperMod" theme is also included in the `themes/PaperMod/` directory. To switch to this theme, you would need to update the `hugo.toml` file.

### Customization

- **Menu:** The main navigation menu can be customized by editing the `data/menu.toml` file.
- **Styling:** While "no-style-please" is a minimalist theme, you can add custom CSS by modifying the theme's asset files in `themes/no-style-please/assets/css/`.
- **Layouts:** Hugo's layout system is used to define the structure of the pages. You can override the theme's default layouts by creating your own layout files in the `layouts/` directory at the root of the project.
