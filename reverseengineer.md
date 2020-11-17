****CONFIG FOLDER
    ****Middleware
        ****isAuthenticated.js

// This is to only allow people that are logged in to be directed to the next page of the site
                module.exports = function(req, res, next) {
            
                if (req.user) {
                    return next();
                }

// If the user isn't logged in, redirect them back to the login page

                return res.redirect("/");
                };


    ****Config.json
//stores information about the databse

    ****passport.js

        var passport = require("passport");
        var LocalStrategy = require("passport-local").Strategy;

        var db = require("../models");

// makes it so we can log in with username and password
        passport.use(new LocalStrategy(
//takes in an email rather than username
        {
            usernameField: "email"
        },
        function(email, password, done) {
//checks databse to see if that email is present
            db.User.findOne({
            where: {
                email: email
            }
            }).then(function(dbUser) {
// It will run this code and display that message if the email is not found
            if (!dbUser) {
                return done(null, false, {
                message: "Incorrect email."
                });
            }
// This will run if the email is valid but the password is not
            else if (!dbUser.validPassword(password)) {
                return done(null, false, {
                message: "Incorrect password."
                });
            }
// This runs if login credentials were inputed correctly
            return done(null, dbUser);
            });
            }
            ));

// Seriallizes and deserializes ther user 
            passport.serializeUser(function(user, cb) {
            cb(null, user);
            });

            passport.deserializeUser(function(obj, cb) {
            cb(null, obj);
            });

            // Exporting our configured passport
            module.exports = passport;

****Models
    ****index.js
//gets database using sequelize

    ****user.js
//creates user model and takes in email and password, it also encrypts password in the database
****Public
    ****js
        ****login.js
//adds functionality to the login page and redirects user to the pages as needed 

        ****members.js
//redirects user to members page once they are signed in

        ****signup.js
//adds the functionality for the page and send the information inputed to the database and handles user errors in the page

//this is the styling for the website, gives it its design
    ****stylesheets
        ****style.css

    ****login.html
//this is what makes up the login page

    ****members.html
// this is what makes up the members page

    ****signup.html
//this is what makes up the signup page

****routes
    ****api-routes.js
//handles routing users to the page if they successfully sign in and if they dont they will be redirected back to another page as well as routing to the page to sign out. Also handles getting and posting data on databse


    ****html-routes.js
//this handles ridercting the user to the home page, members page, and login


****server.js
// Requiring packages
    var express = require("express");
    var session = require("express-session");
// Requiring passport
    var passport = require("./config/passport");

// Uses port 8080 for server
    var PORT = process.env.PORT || 8080;
    var db = require("./models");

//creates express app and middleware 
    var app = express();
    app.use(express.urlencoded({ extended: true }));
    app.use(express.json());
    app.use(express.static("public"));
// keeps track of the users login status
    app.use(session({ secret: "keyboard cat", resave: true, saveUninitialized: true }));
    app.use(passport.initialize());
    app.use(passport.session());

// Requiring the routes
    require("./routes/html-routes.js")(app);
    require("./routes/api-routes.js")(app);

// syncs database and displayes in terminal the link to the port being used 
    db.sequelize.sync().then(function() {
    app.listen(PORT, function() {
        console.log("==> 🌎  Listening on port %s. Visit http://localhost:%s/ in your browser.", PORT, PORT);
    });
    });
