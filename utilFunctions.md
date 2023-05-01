# Capitalise Slug

Capitalises each word in a slug and replaces '`-`' with a space.

```js
exports.capitaliseSlug = (string) => {
  const words = string.split('-');
  const capitalisedWords = words.map((word) => word[0].toUpperCase() + word.substr(1));
  return capitalisedWords.join(' ');
};
```

# Filter Objects

Stripes unwanted properties from an object, and returns a new object

```js
exports.filterObject = (obj, fields) => {
  const output = {};
  Object.keys(obj).forEach((key) => {
    if (fields.includes(key)) {
      output[key] = obj[key];
    }
  });
  return output;
};
```
