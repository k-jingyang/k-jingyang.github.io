## Just a learning journal

<https://k-jingyang.github.io>

## Commands

To serve with live reload

```sh
$ bundle exec jekyll serve --livereload
```

To serve with live reload and include drafts

```sh
$ bundle exec jekyll serve --livereload --drafts
```

## Overriding CSS used by Minima 2.5.1

> This is primarily used to override the syntax highlighting used by Minima

1. We copied `main.scss` from the `minima` gem into `assets/main.scss`
```scss
---
# Only the main Sass file needs front matter (the dashes are enough)
---

@import "minima";
```

2. Add our custom CSS

```bash
bundle exec rougify style base16.solarized  >> assets/main.scss
```