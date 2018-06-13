# port 3000 in use - taskmanager way

1.  Ctrl+Alt+Del > Task Manager > (you may need to click on More Details if you don't see any tabs)
    ![task manager](./static/images/taskmgr.png)
1.  Click Processes Tab
1.  Under Processes, look for Node.js Server-side JavaScript > select it > Click button "End Task"

# Display table

1.  Add some table html template code to your .ejs file
1.  Add a new column for Edit. You can make an anchor tag for now with the text 'Edit' or use an icon
1.  Add a new column for Delete

# Add form

# fillData

Since we will play with the delete functionality, I thought it would be a time-saver if we added another menu option for Fill Data (just for development purposes) that fills the table with 3 rows of data

```html
<a ... href="/filldata">Fill Data</a>
```

In routes/index.js

```js
router.get('/filldata', gamesController.fillData);
```

In controllers/gameController.js

```js
exports.fillData = (req, res) => {
  const data = [
    {
      title: 'Pacman',
      publisher: 'Namco',
      imageUrl: 'https://archive.org/services/img/msdos_Pac-Man_1983',
      year: 1983
    },
    {
      title: 'Oregon Trail',
      publisher: 'MECC',
      imageUrl: 'https://archive.org/services/img/msdos_Oregon_Trail_The_1990',
      year: 1990
    },
    {
      title: 'Sim City',
      publisher: 'Maxis',
      imageUrl: 'https://archive.org/services/img/msdos_SimCity_1989',
      year: 1989
    }
  ];

  Game.collection.insertMany(data);
  res.redirect('/admin');
};
```

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

in views/admin.ejs

```html
<a href="/admin/delete/<%= games[i]._id %>" onclick="return confirm('Are you sure you want to delete this game?')">
```

in routes/index.js

```js
router.get('/admin/delete/:id', gamesController.deleteGame);
```

```js
exports.deleteGame = (req, res) => {
  // use the Game model's remove method to delete the document with the id passed
  Game.remove({ _id: req.params.id }, err => {
    if (err) {
      console.log(err);
    } else {
      res.redirect('/admin');
    }
  });

  // use Game's findByIdAndRemove which unlike .remove(), returns the deleted object in callback
  // TODO: make an undo feature using the deleted object in callback using connect-flash
  // Game.findByIdAndRemove({ _id: req.params.id }, async (err, docs) => {
  //   if (err) {
  //     console.log(err);
  //   } else {
  //     const games = await Game.find();
  //     res.render('admin', {
  //       msg: `${docs.title} has been deleted`,
  //       title: 'Admin',
  //       isActive: 'admin',
  //       games,
  //     });
  //   }
  // });
};
```

## Use js to confirm delete before delete

<a ... onclick="return confirm('Are you sure you want to delete this?');">Delete</a>

---

# Edit/Update

```html
<td><a href=/admin/edit/<%= games[i]._id %>">Edit</a></td>
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

Make sure form has no action so it posts to itself

```html
 <form method="POST">
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

# Using connect-flash

`npm i connect-flash express-session`

# Midterm exam

- do Quizzes
- Read only from mlab
