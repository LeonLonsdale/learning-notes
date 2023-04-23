# View Engines

Pick one:

```
npm i pug
npm i ejs
```

Express recognises these out of the box so no need to import.

# Templating prep

## Tell Express which engine to use

```js
app.set('view engine', 'pug');
```

## Tell express where the views are stored

```js
// create a views directory if it doesn't already exist
app.set('views', path.join(__dirname, 'views'));
```

## Tell express where the static files are

Static files are files such as css files, js files, or any other file that may be used within a served template.

```js
// create a public directory if it doesn't already exist.
// 'public' is the standard/commonly used dir name
app.use(express.static(path.join(__dirname, 'public')));
```

## Render templates when routes are accessed

To send the template, we use a `.render()` method available on a response. Example:

```js
// base refers to a base.pug file stored in views.
// express knows to look in views when render is called.
app.get('/', (req, res) => res.status(200).render('base'));
```

## Passing data to render to be used in the template

We can add data that will be passed to the template to be rendered.

```js
app.get('/', (req, res) =>
  res.status(200).render('base', {
    movieTitle: 'The Matrix',
    releaseYear: 1999,
    star: 'Keanu Reeves',
    story:
      'Humans break free from a digital world created by machines to imprison humans, and use their bodies for energy.',
  })
);
```

# Pug

PUG is a white-space sensitive syntax for writing html. To write pug, we only need to write the html element name, and the indentation - similar to Python.

## Basic syntax

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

## Commenting Pug

To comment pug files:

```pug
// this comment will be shown in the html output
//- this comment will not be shown in the html output
```

## Rendering passed data

```pug
//- everything after '=' is treated as code.
  body
    h1= movieTitle
    p= star
    p= story.
```

### Interpolation

```pug
//- Use when combining plain text with code
html
  head
    title Movies | #{movieTitle}
```

## Using JS in pug

### Buffered JS

Buffered JS will be displayed in the output

```pug
  body
    h1= movieTitle.concat(' (', releaseYear, ')')
    p= star
    p= story.
```

### Unbuffered JS

Unbuffered JS will not be displayed in the output, but can be used by buffered code to display something.

```pug
  body
    - const x = 10;
    h1 2 * x
```

## Partials

A a partial is a piece of the html split off into it's own file. This can be included in the template.

```pug
  // Header
  include _header
```
