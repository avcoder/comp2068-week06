# Delete

## Where will <a href= go?

```html
href=/admin/delete/
```

## How do we reference id for that indivdual document?

```html
<td><a href=/admin/delete/<%= games[i]._id %>">Delete</a></td>
```

As a result, our URL now has the id appended

## Method to process /delete/: id

Using a colon indicates it's variable (like php/mysql).
The way to access this is via req.params.
req.params is one way of getting vars from url
req.body is one way of getting vars from form

```js
/* GET: /games/delete/abc123 */
router.get('/delete/:_id', (req, res, next) => {
  // get the _id paramter from the url and store in a local variable
  const _id = req.params._id;

  // use the Game model to delete the document with this id
  Game.remove({ _id: _id }, err => {
    if (err) {
      console.log(err);
    } else {
      res.redirect('/admin');
    }
  });
});
```

## Use js to confirm delete before delete

<a ... onclick="return confirm('Are you sure you want to delete this?');">Delete</a>

# Edit/Update

```html
<td><a href=/admin/edit/<%= games[i]._id %>">Edit</a></td>
```

Make sure form has no action so it posts to itself
```html
 <form method="POST">
```

## GET handler to process /edit/: id

```js
router.get('/edit', (req, res, next) => {
  // get _id param from url
  const _id = req.params._id;

  // use Game model to find the selected document
  Game.findById(_id, (err, game) => {
    if (err) {
      console.log(err);
    } else {
      res.render('edit', {
        title: 'Car Details',
        game: game
      });
    }
  });
});
```

## Create new edit view

Copy addform html and paste it in edit.ejs
Now form is same as addform but populated

```html
<input ... value="<%= game.title %>">
<input ... value="<%= game.publisher %>">
<input ... value="<%= game.imageUrl %>">
```

## POST handler to process /edit/: id

```js
/* POST:/ game/edit/abc123 */
router.post('/edit/:_id', (req, res, next) => {
  const _id = req.params._id;

  // this is Rich Freeman's example
  const car = new Car({
    _id: _id,
    title: req.body.title,
    publisher: req.body.publisher,
    imageUrl: req.body.imageUrl
  });

  // this is Wes Bos' example
  const game = await Game.findOneAndUpdate({ _id: req.params.id }, req.body, {
    new: true, // return the new store instead of the old one
    runValidators: true
  }).exec();

  // call Mongoose update method, passing the _id and new game object
  Game.update({ _id: _id }, req.body, err => {
    if (err) {
      console.log(err);
    } else {
      res.redirect('/games');
    }
  });
});
```
