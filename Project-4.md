## Project 4 - MEAN Stack Implementation 

MEAN Stack is a combination of following components:

- **MongoDB (Document database)** – Stores and allows to retrieve data.

- **Express (Back-end application framework)** – Makes requests to Database for Reads and Writes.

- **Angular (Front-end application framework)** – Handles Client and Server Requests

- **Node.js (JavaScript runtime environment)** – Accepts requests and displays results to end user.

## Task ##
- Implement a simple Book Register web form using MEAN stack.
**Step 1: Install NodeJs**
Node.js is a JavaScript runtime built on Chrome’s V8 JavaScript engine. Node.js is used in this tutorial to set up the Express routes and AngularJS controllers.

### STEP 1 – BACKEND CONFIGURATION ###
- Update ubuntu
```
sudo apt update
```
![alt](./Images/sudo%20apt%20update.JPG)

- Upgrade Ubuntu
```
sudo apt upgrade
```
![alt](./Images/sudo%20apt%20upgrade.JPG)

- Add certificates
```
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```
![alt](./Images/sudo%20apt%20-y%20Install%20certificates.JPG)


![alt](./Images/Add%20Certificate.JPG)


![alt](./Images/curl%20-sl%20https.JPG)

- Install NodeJS
```
sudo apt install -y nodejs
```
![alt](./Images/sudo%20apt%20install%20-y%20nodejs.JPG)

### Step 2: Install MongoDB ###

MongoDB stores data in flexible, *JSON-like* documents. Fields in a database can vary from document to document and data structure can be changed over time. For the example application, add book records to MongoDB that contain book name, isbn number, author, and number of pages.
```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
```
```
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
```
- Install MongoDB
```
sudo apt install -y mongodb
```
![alt](./Images/sudo%20apt%20Install%20-y%20mongodb.JPG)

- Start The server and verify that the service is up and running
```
sudo service mongodb start
sudo systemctl status mongodb
```
![alt](./Images/sudo%20systemctl%20status%20mongodb.JPG)

- Install npm – Node package manager.
```
sudo apt install -y npm
```
![alt](./Images/sudo%20apt%20install%20-y%20npm.JPG)

- Install body-parser package: to help us process JSON files passed in requests to the server.
```
sudo npm install body-parser
```
- Create a folder named ‘Books’
```
mkdir Books && cd Books
```
- In the Books directory, Initialize npm project
```
npm init
```
- Add a file to it named *server.js*
```
vi server.js
```
- Copy and paste the web server code below into the *server.js* file
```
var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});
```
### STEEP - 3: INSTALL EXPRESS AND SET UP ROUTES TO THE SERVER ###
Express is a minimal and flexible Node.js web application framework that provides features for web and mobile applications. Mongoose package which provides a straight-forward, schema-based solution to model the application data will be used.
```
 sudo npm install express mongoose
 ```
 ![alt](./Images/sudo%20npm%20install%20express%20mongoose.JPG)

 - In ‘Books’ folder, create a folder named *apps*.
 ```
 mkdir apps && cd apps
 ```
 - Create a file named *routes.js* and copy the below code into it.
 ```
 vi routes.js
 ```
 ```
 var Book = require('./models/book');
module.exports = function(app) {
  app.get('/book', function(req, res) {
    Book.find({}, function(err, result) {
      if ( err ) throw err;
      res.json(result);
    });
  }); 
  app.post('/book', function(req, res) {
    var book = new Book( {
      name:req.body.name,
      isbn:req.body.isbn,
      author:req.body.author,
      pages:req.body.pages
    });
    book.save(function(err, result) {
      if ( err ) throw err;
      res.json( {
        message:"Successfully added book",
        book:result
      });
    });
  });
  app.delete("/book/:isbn", function(req, res) {
    Book.findOneAndRemove(req.query, function(err, result) {
      if ( err ) throw err;
      res.json( {
        message: "Successfully deleted the book",
        book: result
      });
    });
  });
  var path = require('path');
  app.get('*', function(req, res) {
    res.sendfile(path.join(__dirname + '/public', 'index.html'));
  });
};
```
- In the *apps* folder, create a folder named *models*

```
mkdir models && cd models
```
Create a file named *book.js* and pacte the below code into it.
```
vi book.js
```
```
var mongoose = require('mongoose');
var dbHost = 'mongodb://localhost:27017/test';
mongoose.connect(dbHost);
mongoose.connection;
mongoose.set('debug', true);
var bookSchema = mongoose.Schema( {
  name: String,
  isbn: {type: String, index: true},
  author: String,
  pages: Number
});
var Book = mongoose.model('Book', bookSchema);
module.exports = mongoose.model('Book', bookSchema);
```

### Step 4 – Access the routes with AngularJS ###

AngularJS provides a web framework for creating dynamic views in the web applications. AngularJS will be used to connect the web page with Express and perform actions on the book register.
- Change directory into *books* folder and create a folder named *Public*

```
cd ../.. && mkdir public && cd public
```
- Add a file named *script.js* and copy the below code into it:
```
vi script.js
```
```
var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope, $http) {
  $http( {
    method: 'GET',
    url: '/book'
  }).then(function successCallback(response) {
    $scope.books = response.data;
  }, function errorCallback(response) {
    console.log('Error: ' + response);
  });
  $scope.del_book = function(book) {
    $http( {
      method: 'DELETE',
      url: '/book/:isbn',
      params: {'isbn': book.isbn}
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
  $scope.add_book = function() {
    var body = '{ "name": "' + $scope.Name + 
    '", "isbn": "' + $scope.Isbn +
    '", "author": "' + $scope.Author + 
    '", "pages": "' + $scope.Pages + '" }';
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
});
```
- In public folder, create a file named *index.html* and paste the below code into it:
```
vi index.html
```
```
<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>

        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>

          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>
```
- Change the directory back up to *Books*

```
cd ..
```
- Start the server by running the command:
```
node server.js
```
![alt](./Images/node%20server.js.jpg)

The server is now up and running. 

- Connect it via port 3300 by opening port 3300 on the EC2 console Security group.

- Access the Book Register web application from the Internet with a browser using Public IP address or Public DNS name.

**http://PublicIP-or-PublicDNS:3300**

![alt](./Images/Web%20book%20register%20application.JPG)


