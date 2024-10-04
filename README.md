# MEAN Stack Project Deployment on AWS

This document outlines the steps to deploy a MEAN stack application on an AWS EC2 instance.

## 1. Launching a Free Tier EC2 Instance on AWS

Follow these steps to launch a free-tier EC2 instance on AWS:

1. Log in to your AWS Management Console.
2. Navigate to the EC2 Dashboard.
3. Click on "Launch Instance."
4. Choose an Amazon Machine Image (AMI) (e.g., Ubuntu Server).
5. Select the instance type (e.g., t2.micro for free tier).
6. Configure instance details and storage as needed.
7. Review and launch the instance.
8. Ensure you have the necessary key pair to SSH into your instance.

![Screenshot](https://github.com/Prince-Tee/stegHub_MEANSTACK/blob/main/screenshot%20fom%20my%20local%20env/launch%20a%20instance%20on%20aws%20for%20MEAN%20stack.PNG)

## 2. SSH Into the EC2 Instance

Use the following command to SSH into your EC2 instance:

```bash
ssh -i "your-key.pem" ubuntu@your-public-ip
```

![Screenshot](https://github.com/Prince-Tee/stegHub_MEANSTACK/blob/main/screenshot%20fom%20my%20local%20env/ssh%20into%20your%20instance%20o%20git%20bash%20using%20the%20aws%20IP.PNG)

## 3. Update Ubuntu

```bash
sudo apt update
```

![Screenshot](link_to_your_screenshot)

## 4. Upgrade Ubuntu

```bash
sudo apt upgrade
```

![Screenshot](link_to_your_screenshot)

## 5. Add Certificates

Install the required packages:

```bash
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
```

![Screenshot](link_to_your_screenshot)

## 6. Install Node.js

Add the Node.js repository:

```bash
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
```

Install Node.js:

```bash
sudo apt install -y nodejs
```

![Screenshot](link_to_your_screenshot)

## 7. Install MongoDB

First, install required packages:

```bash
sudo apt-get install -y gnupg curl
```

Then, add the MongoDB repository:

```bash
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor
```

Add the MongoDB source list:

```bash
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```

Install MongoDB:

```bash
sudo apt-get install -y mongodb-org
```

Start the MongoDB server:

```bash
sudo service mongodb start
```

Verify that the service is running:

```bash
sudo systemctl status mongodb
```

![Screenshot](link_to_your_screenshot)

## 8. Install Yarn

To install Yarn globally, you can use the following command:

```bash
npm install --global yarn
```

![Screenshot](link_to_your_screenshot)

## 9. Create the Project Structure

Create a folder named `Books`:

```bash
mkdir Books && cd Books
```

Initialize the npm project:

```bash
yarn init
```

![Screenshot](link_to_your_screenshot)

## 10. Create the Server File

Create a file named `server.js`:

```bash
nano server.js
```

Copy and paste the following code into `server.js`:

```javascript
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const path = require('path');

const app = express();
const PORT = process.env.PORT || 3300;

// MongoDB connection
mongoose.connect('mongodb://localhost:27017/test', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
})
.then(() => console.log('MongoDB connected'))
.catch(err => console.error('MongoDB connection error:', err));

app.use(express.static(path.join(__dirname, 'public')));
app.use(bodyParser.json());

require('./apps/routes')(app);

app.listen(PORT, () => {
  console.log(`Server up: http://localhost:${PORT}`);
});
```

![Screenshot](link_to_your_screenshot)

## 11. Install Express and Mongoose

Install the necessary packages:

```bash
yarn add express mongoose body-parser
```

![Screenshot](link_to_your_screenshot)

## 12. Set Up Routes

Create a folder named `apps`:

```bash
mkdir apps && cd apps
```

Create a file named `routes.js`:

```bash
nano routes.js
```

Copy and paste the following code into `routes.js`:

```javascript
const Book = require('./models/book');
const path = require('path');

module.exports = function(app) {
  app.get('/book', async (req, res) => {
    try {
      const books = await Book.find();
      res.json(books);
    } catch (err) {
      res.status(500).json({ message: 'Error fetching books', error: err.message });
    }
  });

  app.post('/book', async (req, res) => {
    try {
      const book = new Book({
        name: req.body.name,
        isbn: req.body.isbn,
        author: req.body.author,
        pages: req.body.pages
      });
      const savedBook = await book.save();
      res.status(201).json({
        message: 'Successfully added book',
        book: savedBook
      });
    } catch (err) {
      res.status(400).json({ message: 'Error adding book', error: err.message });
    }
  });

  app.delete('/book/:isbn', async (req, res) => {
    try {
      const result = await Book.findOneAndDelete({ isbn: req.params.isbn });
      if (!result) {
        return res.status(404).json({ message: 'Book not found' });
      }
      res.json({
        message: 'Successfully deleted the book',
        book: result
      });
    } catch (err) {
      res.status(500).json({ message: 'Error deleting book', error: err.message });
    }
  });

  app.get('*', (req, res) => {
    res.sendFile(path.join(__dirname, '../public', 'index.html'));
  });
};
```

![Screenshot](link_to_your_screenshot)

## 13. Create the Book Model

In the `apps` folder, create a folder named `models`:

```bash
mkdir models && cd models
```

Create a file named `book.js`:

```bash
nano book.js
```

Copy and paste the following code into `book.js`:

```javascript
const mongoose = require('mongoose');

const bookSchema = new mongoose.Schema({
  name: { type: String, required: true },
  isbn: { type: String, required: true, unique: true, index: true },
  author: { type: String, required: true },
  pages: { type: Number, required: true, min: 1 }
}, {
  timestamps: true
});

module.exports = mongoose.model('Book', bookSchema);
```

![Screenshot](link_to_your_screenshot)

## 14. Set Up the Public Directory with AngularJS

Change the directory back to `Books`:

```bash
cd ../..
```

Create a folder named `public`:

```bash
mkdir public && cd public
```

Add a file named `script.js`:

```bash
nano script.js
```

Copy and paste the following code into `script.js`:

```javascript
angular.module('myApp', [])
  .controller('myCtrl', function($scope, $http) {
    function fetchBooks() {
      $http.get('/book')
        .then(response => {
          $scope.books = response.data;
        })
        .catch(error => {
          console.error('Error fetching books:', error);
        });
    }

    fetchBooks();

    $scope.del_book = function(book) {
      $http.delete(`/book/${book.isbn}`)
        .then(() => {
          fetchBooks();
        })
        .catch(error => {
          console.error('Error deleting book:', error);
        });
    };

    $scope.add_book = function() {
      const newBook = {
        name: $scope.Name,
        isbn: $scope.Isbn,
        author: $scope.Author,
        pages: $scope.Pages
      };

      $http.post('/book', newBook)
        .then(() => {
          fetchBooks();
          // Clear form fields
          $scope.Name = $scope.Isbn = $scope.Author = $scope.Pages = '';
        })
        .catch(error => {
          console.error('Error adding book:', error);
        });
    };
  });
```

![Screenshot](link_to_your_screenshot)

## 15. Create the HTML File

In the `public` folder, create a file named `index.html`:

```bash
nano index.html
```

Copy and paste the following code into `index.html`:

```html
<!DOCTYPE html>
<html ng-app="myApp" ng-controller="myCtrl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Book Management</title>
  <script src="https://ajax.googleapis

.com/ajax/libs/angularjs/1.8.2/angular.min.js"></script>
  <script src="script.js"></script>
</head>
<body>
  <h1>Book Management</h1>

  <form ng-submit="add_book()">
    <input type="text" ng-model="Name" placeholder="Book Name" required>
    <input type="text" ng-model="Isbn" placeholder="ISBN" required>
    <input type="text" ng-model="Author" placeholder="Author" required>
    <input type="number" ng-model="Pages" placeholder="Pages" required min="1">
    <button type="submit">Add Book</button>
  </form>

  <ul>
    <li ng-repeat="book in books">
      {{book.name}} by {{book.author}} (ISBN: {{book.isbn}}) - {{book.pages}} pages
      <button ng-click="del_book(book)">Delete</button>
    </li>
  </ul>
</body>
</html>
```

![Screenshot](link_to_your_screenshot)

## 16. Start the Server

Start your server with the following command:

```bash
node server.js
```

![Screenshot](link_to_your_screenshot)

## 17. Access the Application

Open your browser and navigate to `http://your-public-ip:3300` to access the application.

![Screenshot](link_to_your_screenshot)

## Conclusion

This guide has covered the steps required to deploy a MEAN stack application on an AWS EC2 instance. If you encounter any issues, refer to the AWS documentation or consult relevant online resources for troubleshooting assistance.
```
