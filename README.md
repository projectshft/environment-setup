# Environment Setup

## Intro
The goal of this short set-up process is to make sure your environment is working as it should. To do that, we're going to walk through a short "project" to ensure that we have our environment ready for future lessons.

## Getting Started
First, find and open your `Terminal` (for Mac users) or `Command Prompt` (Windows users).

Create a folder in your home directory called `code` where you'll store all of your code.

Now, open your "command-line" (the term we'll use for Terminal and/or Command Prompt) and navigate to your `code` folder (or wherever you're going to keep track of your code). The command `cd` will change directories for you:

```
cd code
```

Now that you're inside this folder, we're going to create a new project by creating a new folder. Let's call it `environment-fun`. We'd call this the "root" of our project.

```
mkdir environment-fun
```

As you can see, `mkdir` will make a new directory. Now let's view all the files/folders in our current directory.

```
ls
```

Or if you're using a Windows machine:

```
dir
```

Now let's navigate into `environment-fun` with `cd`:

```
cd environment-fun
```

It's time to create a file. Let's start by merely creating an `index.html` file via the command-line:

```
touch index.html
```

Or for windows users:

```
copy nul index.html
```

Next, we'll want to open this up inside of a text-editor or IDE (integrated development environment).

You may use any text-editor you wish, but we recommend either [VS Code](https://code.visualstudio.com/) or [Atom](https://atom.io/). This set-up will walk you though using Atom, but in the future, we'll switch over to using VS Code.

So go to either site and install. Follow along if you're using Atom.

To open our project in Atom, make sure you're in the `environment-fun` directory via your command-line and type:

```
atom .
```

Did you get an error? Let Aaron know on Monday and we'll debug. Of course you're welcome to google and figure it out yourself (which is what Aaron is probably going to do). If you don't have any errors, you're all good!

---

## Installing and Checking Git
Next we'll install Git. Go [here](https://git-scm.com/downloads) and download it to your machine.

We're not going to actually build anything or commit anything with `git` (today), but let's check that it works. In your command-line write:

```
git init
```

If the response was something like, `Initialized empty Git repository in...` then you're good to go! If there were any kind of errors, let Aaron know. We may need to set up git in a different capacity. Or you could google...

---

## Downloading and Checking Node
Like we did with Git, we're going to download Node [here](https://nodejs.org/en/download/).

Next, we're going to check and see if you have Node installed correctly. From your command-line type:

```
npm init
```

If you see something like, `This utility will walk you through creating a package.json file...`, then you're good to go! You can press "control" + "c" to quit. Again, if there is some kind of error, let Aaron know (or [consult our detailed set-up guide here](https://www.google.com/)).

---

## Making Your Code Pretty
One of the things that will be important for you as a JavaScript developer is writing code that is easy to read in both logic and aesthetics. Since JavaScript isn't whitespace sensitive (it doesn't care too much about spaces, tabs, etc), then we have to try really hard to make it look good.

As a beginner, your perception of what "looks good" make be off. So for now, we'll utilize an Atom package to help us out. Be sure to have your `environment-fun` folder opened in Atom.

First, go to `Preferences` in Atom (`Command` + `,` for mac). Then find `Install` on the left. In the search bar at the top, type in `Beautify`:

![img](https://www.projectshift.io/wp-content/uploads/2018/01/Screen-Shot-2018-01-04-at-1.51.22-PM.png)

Then click `Install`. After it finishes, let's try it out! First we'll need some ugly code though. Go ahead and create a `main.js` and `style.css` files alongside your `index.html` file.

Paste the following JavaScript inside of `main.js`:

```js
var express = require('express');
var app = express();
var async = require('async');
var request = require('request');
var Tweet = require('./twitterFeed');

var bodyParser = require('body-parser');
app.use(bodyParser.urlencoded({ extended: false }));
app.use(bodyParser.json());

// Tells express to look fr files using either of the below as a root
app.use(express.static('public'));
app.use(express.static('node_modules'));




app.get('/city/:city', function(req, res, next) { // req and res are special express utilities to help us send and recieve data

    var city = req.params.city;

    // gets the flickr data
    function flickrCall(cb) {
        getFlickrData(city, function(err, result) {
            cb(err, result);
        })
    }

    // gets the weather data
    function weatherCall(cb) {
        getWeatherData(city, function(err, result) {
            cb(err, result)
                //console.log('weather result: ', result)
        })
    }
/*
    function wikipediaCall(cb) {
        getWikipediaData(city, function(err, result) {
            cb(err, result);
        })
    }*/

    // gets the lat/long for weather data
    function weatherLatLongCall(cb) {
        getWeatherData(city, function(err, result) {
            cb(err, { lat: result.coord.lat, long: result.coord.lon })
                //console.log('weather result: ', result)
        })
    }

    // get just the lat and long from foursquare
    function foursquareCall(cb) {
        getFoursquareData(city, function(err, result) {
            cb(err, { lat: result.geocode.center.lat, long: result.geocode.center.lng })
        })
    }

    // get all the foursquare data
    function foursquareVenueCall(cb) {
        getFoursquareData(city, function(err, result) {
            cb(err, result)
        })
    }

    // uses the trends/closest route to get the woeid
    function twitterClosest(geolocation, cb) {
        Tweet.closest(geolocation)
            .then(function(id) {
                cb(null, id)
            }, function(err) {
                cb(err)
            })
    }

    // uses the trends/place route to get the trends
    function twitterPlace(id, cb) {
        Tweet.place(id)
            .then(function(result) {
                cb(null, result)
            }, function(err) {
                cb(err)
            })
    }

    // gets the tweets from the search/tweets route
    function twitterCall(cb) {
        Tweet.search({
                q: city,
                count: 5,
                result_type: 'popular recent',
                lang: 'en'
            })
            .then(function(result) {
                cb(null, result)
            }, function(err) {
                cb(err)
            })
    }

    // foursquareCall gets lat and long, then passes it to twitterClosest for the woeid then to TwitterPlace for the trends
    function twitterTrends(cb) {
        async.waterfall([
            weatherLatLongCall,
            twitterClosest,
            twitterPlace
        ], function(err, results) {
            cb(err, results);
        })
    }

    // All together now!
    async.parallel([
            foursquareVenueCall,
            flickrCall,
            weatherCall,
            twitterCall,
            twitterTrends
        ],
        function(err, results) {
            if (err) {
                console.log(err)
                return res.status(500).send(err)
            }
            return res.json(results)
        })
});
//END of get request

var getFoursquareData = function(city, cb) {

    var options = {
        qs: {
            section: 'food',
            near: city,
            venuePhotos: 1,
            limit: 20,
            client_id: 'QLJUKUZ0FU0NVLOWLUZJOOJHB1MTWSYMPHQBSKJ5FXKJH102',
            client_secret: '5L3IZX1VKHONEULQBYLDSIC4HTZWEXVJFQRL4FE4ZIAWNS20',
            v: 20161231,
            m: 'foursquare'
        }
    };

    // get something cool from the FourSquare API
    request.get('https://api.foursquare.com/v2/venues/explore', options, function(err, response, body) {
        if (err) {
            return cb(err, null)
        }
        // need to parse response because it was returning as a string
        //console.log(JSON.parse(body).response.geocode.center);
        //var coord = JSON.parse(body).response.geocode.center
        return cb(null, JSON.parse(body).response)
    })
};


// Flickr photos
var getFlickrData = function(city, cb) {
    var options = {
        qs: {
            method: 'flickr.photos.search',
            api_key: '6ae44d19471914449a7bc6764acba0ef',
            text: city,
            format: 'json',
            nojsoncallback: '?',
            page: '1',
            sort: 'relevance',
            content_type: '1',
            media: 'photos'
        }
    };
    // get something cool from the Flickr
    request.get('https://api.flickr.com/services/rest/?', options, function(err, response, body) {
        if (err) {
            return cb(err, null)
        }
        // need to parse response because it was returning as a string
        return cb(null, JSON.parse(response.body));
    })
};

var getWeatherData = function(city, cb) {
    var options = {
            qs: {
                q: city,
                units: 'imperial',
                appid: 'b51ff059850fb59ef5b5085a6e089a74'
            }
        }
        // get something cool from the OpenWeatherMap
    request.get('http://api.openweathermap.org/data/2.5/weather?', options, function(err, response, body) {
        if (err) {
            return cb(err, null)
        }
        // need to parse response because it was returning as a string
        //console.log(response.body);
        return cb(null, JSON.parse(response.body));
    })
};

app.listen(process.env.PORT || '8000');
```

It's ugly - don't worry about what it does. Using the command palette we can make it pretty. To access the command pallet type `cmd-shift-p` (mac) or `ctrl-shift-p` (windows). Then search `beautify JavaScript` and watch the magic happen:

![img](https://cloud.githubusercontent.com/assets/1885333/25775586/f3fc7ec4-327e-11e7-8576-45e735e80032.gif)

By default it's set to be indent 2 spaces (which is good). You can read more about how to change the settings [from the documentation here](https://github.com/Glavin001/atom-beautify).

Now, do the same thing with our HTML and CSS files by adding this ugly HTML:

```html
<!DOCTYPE html>
<html ng-app="cityPulse">

<head>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <script type="text/javascript" src="/d3/build/d3.js"></script>
    <script type="text/javascript" src="/d3-scale-chromatic/build/d3-scale-chromatic.js"></script>
    <link href="/bootstrap/dist/css/bootstrap.css" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css?family=Varela+Round" rel="stylesheet">
    <link rel="stylesheet" type="text/css" href="/weather-icons.css">
    <link rel="stylesheet" href="/font-awesome/css/font-awesome.min.css">
    <link rel="stylesheet" type="text/css" href="/animate.css">
    <link rel="stylesheet" type="text/css" href="/style.css">
    <link rel="stylesheet" media="(max-width: 500px)" href="mobile.css" />
    <script src="/angular/angular.js"></script>
    <title>City Pulse</title>
</head>

<body ng-controller="mainController" class="ng-cloak">
    <div ng-show="loading" class="loader"><i class="fa fa-spinner fa-spin"></i></div>
    <div class="site-wrapper" ng-class="blur">
        <div class="top-section">
            <nav class="navbar navbar-custom">
                <div class="container-fluid">
                    <!-- Brand and toggle get grouped for better mobile display -->
                    <div class="navbar-header">
                        <p class="navbar-brand" href="#">City Pulse</p><img src="/images/logo.png" />
                    </div>
                </div>
                <!-- /.container-fluid -->
            </nav>
            <!-- top section -->
            <div class="row">
                <div class="container-fluid">
                    <div class="search">
                        <div class="col-sm-12">
                            <form ng-submit="search(city)">
                                <input type="text" class="search-box col-sm-12" name="city" placeholder="Search for a city" ng-autocomplete ng-model="city" options="{types: '(cities)'}" details="details" ng-required="true" />
                                <button type="button" class="search-btn" ng-click="search(city)"><span class="glyphicon glyphicon-search" aria-hidden="true"></span></button>
                            </form>
                            <!-- <button type="button" class="btn" ng-click="search(city)">Search</button> -->
                        </div>
                    </div>
                </div>
            </div>
            <!-- General stats like weather and time -->
            <div class="row">
                <div class="container-fluid">
                    <div class="col-sm-12 text-center">
                        <div class="weather-container">
                            <div class="icon-wrap">
                                <i class="wi wi-owm-{{ weatherDB.weather[0].id}}"></i>
                            </div>
                            <p ng-click="switchTemperature()" ng-show="fahrenheit" class="temp">{{ tempFahrenheit | number: 0}} &#8457; | {{ weatherDB.weather[0].main | lowercase }}</p>
                            <p ng-click="switchTemperature()" ng-show="!fahrenheit" class="temp">{{ tempCelsius | number: 0}} &#8451; | {{ weatherDB.weather[0].main | lowercase }}</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="super-container">
            <div class="row no-data" ng-hide="photoDB || fourDB || tweets">
                <div class="container-fluid">
                    <div class="col-sm-12 section">
                        <h2>Nothing to see here. </h2>
                    </div>
                </div>
            </div>
            <!-- FLICKR PHOTOS -->
            <div class="row" ng-hide="!photoDB">
                <div class="container-fluid">
                    <div class="col-sm-12 section">
                        <h2 class="title text-center">The Sights</h2>
                        <div ng-repeat="photo in photoDB | limitTo: 3">
                            <div class="col-sm-4">
                                <div class="img-flickr {{animateClass}} animated" style="background-image: url(https://farm{{photoDB[$index+counter].farm}}.staticflickr.com/{{photoDB[$index+counter].server}}/{{photoDB[$index+counter].id}}_{{photoDB[$index+counter].secret}}.jpg);"></div>
                            </div>
                        </div>
                        <button class="btn-gallery btn-left" ng-click="lastPhoto()">
                            <span class="glyphicon glyphicon-chevron-left" aria-hidden="true"></span>
                        </button>
                        <button class="btn-gallery btn-right" ng-click="nextPhoto()">
                            <span class="glyphicon glyphicon-chevron-right" aria-hidden="true"></span>
                        </button>
                        <button class="btn-gallery btn-bottom" ng-click="nextPhoto()">
                            <span class="glyphicon glyphicon-chevron-down" aria-hidden="true"></span>
                        </button>
                        <button class="btn-gallery btn-up" ng-click="lastPhoto()">
                            <span class="glyphicon glyphicon-chevron-up" aria-hidden="true"></span>
                        </button>
                    </div>
                </div>
            </div>
            <!-- Foursquare data -->
            <div class="row" ng-hide="!fourDB">
                <div class="container-fluid">
                    <div class="col-sm-12 section">
                        <h2 class="title text-center">The Tastes</h2>
                        <div ng-repeat="venues in fourDB | limitTo: 4">
                            <div class="col-sm-3">
                                <div class="fq-venue-name-container">
                                    <span class="fq-venue-name">{{ fourDB[$index+counterPlace].venue.name | limitTo: 25 }}</span>
                                </div>
                                <div class="img-responsive {{animateClass}} img-foursquare img-border" style="background-image: url({{fourDB[$index+counterPlace].venue.photos.groups[0].items[0].prefix}}original{{fourDB[$index+counterPlace].venue.photos.groups[0].items[0].suffix}});" />
                            </div>
                        </div>
                    </div>
                    <button class="btn-gallery btn-left" ng-click="lastPlace()">
                        <span class="glyphicon glyphicon-chevron-left" aria-hidden="true"></span>
                    </button>
                    <button class="btn-gallery btn-right" ng-click="nextPlace()">
                        <span class="glyphicon glyphicon-chevron-right" aria-hidden="true"></span>
                    </button>
                    <button class="btn-gallery btn-bottom" ng-click="nextPlace()">
                        <span class="glyphicon glyphicon-chevron-down" aria-hidden="true"></span>
                    </button>
                    <button class="btn-gallery btn-up" ng-click="lastPlace()">
                        <span class="glyphicon glyphicon-chevron-up" aria-hidden="true"></span>
                    </button>
                </div>
            </div>
            <!--Wikipedia -->
            <!--         <div class="row" ng-hide="{{wikipedia.length > 20 && wikipedia.length }}">
            <div class="container"> {{wikipedia.length > 20 && wikipedia }} {{ allData[5][2]}}
                <div id="weekee" class="col-sm-12 section">
                    <h3><a href="{{wikiLink}}">{{wikipedia}} </a></h3>
                </div>
            </div>
        </div> -->
            <!-- Twitter -->
            <div class="row">
                <div class="container-fluid">
                    <div class="col-sm-12 section">
                        <h2 class="title text-center">The Talk</h2>
                        <div ng-repeat="trend in trends | filter:{tweet_volume:'!!'} | orderBy:'-tweet_volume' | limitTo: 8">
                            <div class="col-sm-3 trend-container">
                                <button type="button" class="btn"><a href="{{trend.url}}">{{ trend.name }}</a></button>
                            </div>
                            <!--{{ trend.tweet_volume }}-->
                        </div>
                        <div ng-repeat="tweet in tweets | limitTo: 3">
                            <div class="col-sm-12 tweet-container">
                                <div>
                                    <div class="col-sm-1">
                                        <img src="{{ tweet.profileimage }}" class="img-tweet" />
                                    </div>
                                    <div class="col-sm-11">
                                        <p class="tweet-username"><strong>{{ tweet.username }}</strong></p>
                                        <p ng-bind="tweet.status" linkify="twitter"></p>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <!-- END OF CONTAINER -->
    <div id="img-mapquest" class="row">
        <img src="https://beta.mapquestapi.com/staticmap/v5/map?key=c9LUj3gveNGWjKMfAUk0VKnN8ZwkLuqX&size=1500,+170&type=sat&locations={{city}}" />
    </div>
    <div id="footer">City Pulse was made by <a href="http://www.daniellecarrick.com" target="_blank">Danielle</a>, <a href="mailto:yanivsalzberg@gmail.com?Subject=City%20Pulse">Yaniv</a>, <a href="https://github.com/n8chesley" target="_blank">Nate</a> and <a href="https://github.com/yonatan-otzap" target="_blank">Yonatan</a>.</div>
    </div>
    <!--
    <div id="img-mapquest" style="background-image: url(https://beta.mapquestapi.com/staticmap/v5/map?key=c9LUj3gveNGWjKMfAUk0VKnN8ZwkLuqX&size=1500,+170&type=sat&locations={{city}})"></div> -->
    <!-- Scripts -->
    <script src="/js/angular-linkify-changed/angular-linkify.js"></script>
    <script type="text/javascript" src="https://maps.googleapis.com/maps/api/js?libraries=places&key=AIzaSyDQX1NwXDMRS8EEfA_bHLoB5N3NkB1e5a0"></script>
    <script src="/js/app.js"></script>
    <script src="/js/services/apiFactory.js"></script>
    <script src="/js/controllers/mainController.js"></script>
    <script src="/js/directives/weatherDirective.js"></script>
    <script src="/ng-autocomplete/src/ngAutocomplete.js"></script>
    <script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

  ga('create', 'UA-22209624-6', 'auto');
  ga('send', 'pageview');

</script>
</body>

</html>
```

And this ugly CSS:

```CSS
body {
    height: 100%;
    font-family: 'Varela Round', sans-serif;
    color: white;
    background-color: #f0f0f0;
}

input:focus,
select:focus,
textarea:focus,
button:focus {
    outline: none;
}

.glyphicon-search {
    /*    font-size: 30px;
    right: 10px;
    top: 10px;
    position: absolute;*/
    opacity: .75;
}

.glyphicon-search:hover {
    opacity: 1;
    font-size: 35px;
    transition: opacity .5s, font-size .5s;
}

.search-btn {
    background-color: transparent;
    border: none;
    font-size: 30px;
    right: 10px;
    top: 10px;
    position: absolute;
    opacity: .5;
}

.top-section {
    padding-bottom: 40px;
    background-color: grey;
}

.super-container {
    margin: 30px;
    border-radius: 3px;
    color: black;
}

.section {
    margin-bottom: 20px;
    background: white;
    margin-top: 20px;
    padding-bottom: 30px;
    border-radius: 3px;
}

.title {
    margin: 30px 0px;
}

.section-padding {
    margin-bottom: 30px;
}

.navbar-custom {
    background-color: rgba(255, 255, 255, .1);
}

.navbar-brand {
    color: white;
}

.navbar-header img {
    height: 40px;
}

.search-box {
    font-size: 40px;
    background-color: transparent;
    border: none; border-bottom: 1px solid white;
    text-align: center;
    text-transform: uppercase;
}

.weather-container {
    margin-top: 30px;
}

.icon-wrap {
    margin-bottom: 20px;
}

.wi {
    font-size: 70px;
}

.temp {
    font-size: 24px;
    cursor: pointer;
}


/* Twitter section */

p.tweet-username {
    margin-bottom: 0px;
}

.tweet-container {
    padding: 20px 0px;
    border-radius: 3px;
    margin: 10px 0px;
    display: inline;
}

.trend-container button {
    background-color: white;
    border: 1px solid #f0f0f0;
    margin-bottom: 10px
}

.img-tweet {
    border: 2px solid white;
    border-radius: 3px;
}


/* FOURSQUARE STUFF */

.img-thumbnail {
    padding: 0px;
}

.fq-venue-name-container {
    /*  height: 40px;
  width: 100%;
  background-color: rgba(50,50,50,.5);
  position: absolute;
  top: 50%;*/
}

.fq-venue-name {
  text-align: center}

.img-foursquare {
  height: 200px;
    background-size: cover;
background-repeat: no-repeat;
    background-position: center;
    margin-bottom: 20px;
}


/* Flickr stuff */

.img-flickr {
    height: 300px;
background-size: cover;
    background-repeat: no-repeat;
  background-position: center;
    margin-bottom: 20px;
}

.img-border {
    border: 1px solid #f7f7f7;
    border-radius: 3px;
}


/* Mapquest stuff */

#img-mapquest {
  height: 170px;
  background-size: cover;
  background-repeat: no-repeat;
    background-position: center;
    overflow-y: hidden;
}


/* animation transitions */

.img-transition { background-color: red;
}

.btn-gallery {
    color: rgba(255, 255, 255, 1);
    background-color: transparent;
    position: absolute;
  border: none;
    width: 50px;
  height: 50px;
    font-size: 30px;
}

.btn-left {
    left: 10px;
    top: 50%;
}

.btn-right {
    right: 10px;
    top: 50%;
}

.btn-bottom {
    display: none;
}

.btn-up {display: none;
}

.btn-gallery .glyphicon {
    text-shadow: 1px 1px 4px grey;}


/* Loader */

.loader {
    position: fixed;
    height: 100%;
      width: 100%;
    background: rgba(10, 10, 10, 0.05);
    z-index: 100;
    text-align: center;
    padding-top: 25%;
    font-size: 100px;
}

.blur {
    filter: blur(5px);
}

#footer {
    background-color: black;
    color: white;
    padding: 10px;
}

#footer a {
    color: white;
}
```

---

## Other Atom Settings
In addition to utilizing the beautify package, there are two more things that would be helpful. First, install the package called `autosave`. This will ensure that any time you un-focus from that window, your code will be saved.

Next, go to Preferences > Settings > Editor. The most important settings to check here are `Tab Length`, which should be set to `Default: 2` and `Tab Type` which should be set to `soft`. Yep, we just chose [spaces over tabs](https://www.youtube.com/watch?v=V7PLxL8jIl8), but you can still use the tab key thanks to Atom.
