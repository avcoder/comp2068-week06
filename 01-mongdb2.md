# port 3000 in use - taskmanager way

1.  Ctrl+Alt+Del > Task Manager > (you may need to click on More Details if you don't see any tabs)
    ![task manager](./static/images/taskmgr.png)
1.  Click Processes Tab
1.  Under 'Background processes', look for Node.js Server-side JavaScript > select it > Click button "End Task"

# Display table
Prerequesites for today's class is you need a table to display data, and an add form.  So just to review/backtrack a bit...

1.  Add some table html template code to your .ejs file
1.  Add a new column for Edit. You can make an anchor tag for now with the text 'Edit' or use an icon
1.  Add a new column for Delete
1.  Add EJS variables using a for loop
  ```js
  <% for (let i = 0; i < games.length; i++ ) { %>
      <tr>
          <td> <%= games[i].title %> </td>
          <td> <%= games[i].publisher %> </td>
          <td> <%= games[i].year %> </td>
          <td> <%= games[i].imageUrl %> </td>
          <td> <a href="#">Edit</a> </td>
          <td> <a href="#">Delete</a> </td>
      </tr>
  <% } %>
  ```

Q. In our .ejs file above, where does games come from?

A. It came from our controllers/gameController.js
```js
const Game = require('../models/Game');

exports.admin = (req, res) => {
  Game.find((err, games) => {
    if (err) {
      res.render('error');
    } else {
      res.render('admin', {
        title: 'Admin',
        isActive: 'admin',
        games,
      });
    }
  });
};
```
Q. Where is Game defined that it knows how to populate games?

A. It's defined in models/Game.js

```js
const mongoose = require('mongoose');

// define a schema for the game model
// this and all other models be inherit from mongoose.Schema

const gameSchema = new mongoose.Schema({
  title: {
    type: String,
    required: 'Doh! You forgot to put in the game title',
  },
  publisher: {
    type: String,
    required: 'Please enter publisher',
  },
  imageUrl: {
    type: String,
    required: 'Please enter URL of game cover image',
  },
  year: {
    type: Number,
  },
});

// make the= mongoose.model('Game', gameSchema);
```

# Add form
1. Create views/addGame.ejs 
1. Insert html code for a form that takes in title, publisher, imageUrl.  
1. Leave year out, we will deal with this in a special way.

  ```html
      <form method="POST" action="/add">
              <input type="text" id="name" name="title">
              <label for="title">Title</label>
          
              <input type="text" id="publisher" name="publisher">
              <label for="publisher">Publisher</label>

              <input type="text" id="image_url" name="imageUrl">
              <label for="imageUrl">imageUrl</label>

          <button type="submit">Submit</button>
      </form>
  ```
1. Ensure that the form's method is POST, and action is "/add"
1. Ensure that each input's name attribute matches our mlab fields

  * This is where we left off last week.  If you wish to start here too, you can clone https://github.com/avcoder/comp2068-portfolio-template or continue building on your own code from last week.  But if you download mine you have to create your variables.env file to insert your own DATABASE=mongodb:// ...

1. In routes/index.js
  ```js
  router.get('/add', gamesController.addGame);
  ```

1. In controllers/gameController.js
  ```js
  exports.addGame = (req, res) => {
    res.render('addGame', {
      title: 'Add Game',
      isActive: 'add',
    });
  };
  ```
Try going to the add page now, it should appear but is not yet functional

1. In routes/index.js
```js
router.post('/add', gamesController.createGame);
```

1. In controllers/gameController.js
```js
  exports.createGame = (req, res) => {
    try {
      const game = new Game(req.body);
      game.save();
      res.redirect('/admin');
    } catch (err) {
      console.log(err);
    }
  };
```

1. To account for the year we're going to make our code automatically extract the year from the imageURL.  In models/Game.js
```js
// before it is saved, it will run this function
gameSchema.pre('save', function (next) {
  // ! must use 'function' above so 'this' refers to correct object

  // if imageUrl is not modified, then do nothing, otherwise get year
  if (!this.isModified('imageUrl')) {
    next(); // skip it
    return; // stop this fn from running
  }
  // get year from last 4 charactesr of imageURL
  this.year = this.imageUrl.substr(-4);
  next();
});
```
* Try adding some data, it should work!

# Fix up tiny things

## Fix underline for nav if is-active

1. in views/header.ejs make other links potentially show is-active 

## Use icons
1. In views/header.ejs, replace words edit/delete with [icons](https://material.io/tools/icons/?style=baseline)
```html
<a href="#"><i class="material-icons">edit</i></a>
<a href="#"><i class="material-icons">delete_forever</i></a>
```

## Center play.ejs
1. remove h1
1. Make iframe centered `style="width: 700px; margin: 0 auto;"`



# fillData

Since we will play with the delete functionality, I thought it would be a time-saver if we added another menu option for Fill Data (just for development purposes) that fills the table with 3 rows of data.

1. Replace nav link 'Home' with 'FillData'
  ```html
  <a ... href="/filldata">Fill Data</a>
  ```

1. In routes/index.js

  ```js
  router.get('/filldata', gamesController.fillData);
  ```

1. In controllers/gameController.js

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
req.body is one way of getting vars from form - but you need to require('body-parser')

1. in views/admin.ejs

  ```html
  <a href="/admin/delete/<%= games[i]._id %>">
  ```

1. in routes/index.js

  ```js
  router.get('/admin/delete/:id', gamesController.deleteGame);
  ```

1. in controllers/gameController.js
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
  ```

1. Use confirm before delete

  `<a ... onclick="return confirm('Are you sure you want to delete this?');">Delete</a>`

  For assignments, above confirm() is good enough, but you may want to reconsider [not using confirm prompts](https://alistapart.com/article/neveruseawarning)

  For example, see my node/lesson5c/ app which has both getmdl's snackbar and materialize's toast as examples
  We'll come back to this at the end of class if there's time.

---

# Edit/Update

1. in views/admin.ejs
  ```html
  <td><a href=/admin/edit/<%= games[i]._id %>">Edit</a></td>
  ```

1. GET handler to process /edit/: id

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

1. Create new edit view

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

1. POST handler to process /edit/: id

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


# Midterm exam

- do Quizzes
- 1st part, you will get 30 randomized m/c questions from a pool of 50 questions.
- 2nd part: CRUD? Read only from mlab. Given a table screenshot and data to insert, recreate it.
- today's class of create, delete, update won't be part of exam
- create your own, don't clone mine otherwise 0


# Use .findByIdAndRemove() with connect-flash for Delete function
* concept 1 - async/await functions
* concept 2 - object destructuring
* concept 3 - connect-flash

1. Alternatively, instead of using .remove(), you can use .findByIdAndRemove()   
  ```js
  exports.deleteGame = (req, res) => {
    // use Game's findByIdAndRemove which unlike .remove(), returns the deleted object in callback
    Game.findByIdAndRemove({ _id: req.params.id }, async (err, docs) => {
      if (err) {
        console.log(err);
      } else {
        const { title, publisher, imageUrl } = docs;
        undoGame.title = title;
        undoGame.publisher = publisher;
        undoGame.imageUrl = imageUrl;
        req.flash('success', `Successfully deleted ${docs.title}`);
        res.redirect('/admin');
      }
    });
  };
  ```

1. `npm i connect-flash express-session`  

