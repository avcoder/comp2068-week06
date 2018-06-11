# Add a delete link to table

## Where will <a href= go?

```html
href=/games/delete/
```

## How do we reference id for that indivdual document?

```html
href=/games/delete/<%= games._id %>
```

As a result, our URL now has the id appended
