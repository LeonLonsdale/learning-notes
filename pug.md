# Pug

## Contents

- [What is PUG?](#what-is-pug)
- [Basic Syntax](#basic-syntax)
  - [HTML](#html)
  - [Commenting](#commenting)
- [Rendering data](#rendering-data)
  - [Interpolation](#interpolation)
- [Using JavaScript in PUG](#using-js-in-pug)
  - [Buffered JS](#buffered-js)
  - [Unbuffered JS](#unbuffered-js)
- [Looping](#looping-through-arrays)
  - [Using the Index](#using-the-index)
- [Conditionals](#conditionals)
- [Mixins](#mixins)
- [Template Inheritance](#template-inheritance)
- [Partials](#partials)
- [The difference between partials and block & extends](#the-difference-between-partials-and-block--extends)

# What is pug?

PUG is a white-space sensitive syntax for writing html. To write pug, we only need to write the html element name, and the indentation - similar to Python.

# Basic syntax

## HTML

```pug
html
  head
    title Movies
    link(rel='stylesheet' href='css/style.css')
    link(rel='shortcut icon' type='image/png' href='img/favicon.png')

  body.class
    h1 I Like Movies
    p Not all of them.
```

## Commenting

To comment pug files:

```pug
// this comment will be shown in the html output
//- this comment will not be shown in the html output
```

# Rendering data

We can use data passed in by adding `=` after the html element name. There must be a space after the `=` and before the data.

```pug
//- everything after '=' is treated as code.
body
  h1= movieTitle
  p= star
  p= story.
```

## Interpolation

Interpolation allows us to use passed data in a string of plaintext.

```pug
//- Use when combining plain text with code
html
  head
    title Movies | #{movieTitle}
```

# Using JS in pug

## Buffered JS

`Buffered` JS will be displayed in the output.

```pug
body
  h1= movieTitle.concat(' (', releaseYear, ')')
  p= star
  p= story.
```

## Unbuffered JS

`Unbuffered` JS will not be displayed in the output, but can be used by buffered code to display something.

```pug
body
  - const x = 10;
  h1 2 * x
```

# Looping through arrays

When passing an array of data to a pug template, pug has built in looping features. If each item in a loop should have a rendered display, the 'template' of that rendered display should be indented into the loop. To loop in pug we use `each in`. `each of` also works.

```pug
each item in items
  startOfTemplate
```

## Using the index

We can also get and use the array index:

```pug
each item, i in items
```

# Conditionals

Best option for conditionals is to use `unbuffered JavaScript`. The html/pug output of the conditional should be indented iniside of the JS.

```pug
- if(condition)
  // display this
- if(condition)
  // display this
```

However PUG conditionals can be used for simple checks, such as checking if the user is logged in (see Security for middleware to provide this information to the templates).

```pug
if user
  display this
else
  display this
```

# Mixins

Used to create a `template of a repeated element`. Can pass arguments in similar to a function.

```pug
mixin overviewBox(label, text, icon)
  .overview-box__detail
    svg.overview-box__icon
      use(xlink:href=`/img/icons.svg#icon-${icon}`)
    span.overview-box__label= label
    span.overview-box__text= text
```

To insert the mixin into the page we use a `+` symbol, immediately followed by the mixin name and its arguaments:

```pug
+overviewBox(label, text, icon)
```

Mixins can also be used in loops:

```pug
each review in reviews
  +reviewBox(review)
```

# Template Inheritance

Template inheritance is achieved using `block` and `extends` keywords.

The `block` keyword is used in the layout or `parent` template. The block should be provided a name. A `child` template then replaces this block when rendered, using the `extends` keyword.

When rendering a page, you render the child. The `child extends the parent`, and `the content of the child replaces the block in the parent` with its own content.

```pug
//- parent template
//- base.pug
doctype html
html(lang='en')
  head
    meta(charset='UTF-8')
    meta(name='viewport' content='width=device-width, initial-scale=1.0')
    title Title text
    link(rel='stylesheet' href='/css/style.css')
    link(rel='shortcut icon' type='image/png' href='/img/favicon.png')

  body
    // Header
    include _header

    // Content
    block content

    // Footer
    include _footer
```

Then each block can have its own file:

```pug
//- content template
//- A separate content template should be created for each page
extends base

block content
  main.main
    // content code
```

## block append

Block can also be used to append to existing code by using the `block append` keywords. It's important to note that the existing code that must be appended to should be indented into the block.

```pug
//- head partial
doctype html
html(lang='en')
  head
    block head
      meta(charset='UTF-8')
      meta(name='viewport' content='width=device-width, initial-scale=1.0')
      title My title
      link(rel='stylesheet' href='/css/style.css')
```

Then in another file:

```pug
block append head
  script(src='...')
```

The `script` will now be appended to the rest of the head.

# Partials

A a partial is a piece of the pug split off into it's own file. This can be included in the template using the `include` keyword. No `extend` is necessary for this.

```pug
// Header
include _header
```

```pug
//- in the _header.pug
header.header
  nav.nav
    //more code
```

A mixin can also be saved to a separate file and included in the same way:

```pug
include _overviewBox
```

# The difference between partials and block & extends

A block replaces the code at the point the block declared in a template, by extending the template. The same block can be replaced by different files depending on the page requested. For example, you may have a file for the following pages: blog, merchandice, news, gallery. Each would only contain the content for that page, and would replace the `block content` declaration in the template when called.

A partial included into a file. The content of the partial will not change. Usually used for parts of a page that should always be the same - such as the header, nav, footer.
