# ohmusso.github.io

## jekyll info

### local test

``` powershell
bundle exec jekyll serve
```

or

``` powershell
bundle command: bundle exec jekyll serve --draft
```

#### if error "cannot load such file -- webrick (LoadError)"

add wbrick to bundle
command: bundle add webrick
command: bundle install

## theme

use minima

<https://github.com/jekyll/minima>

## note

### css

#### _includes/head.html

Import css.

```html
  <link rel="stylesheet" href="{{ "/assets/css/style.css" | relative_url }}">
```

#### assets/css/style.scss

Import scss or sass.
Following, import **\_sass/minima/skins/\.scss** and **\_sass/minima/initialize.scss**.
We can customize style by writing to **\sass/\*.scss**.

```scss
---
# Only the main Sass file needs front matter (the dashes are enough)
---

@import
  "minima/skins/{{ site.minima.skin | default: 'classic' }}",
  "minima/initialize";
```

#### build and output

jekyll build **\*.scss** and ouput **\_site/assets/css/style\.css**.
