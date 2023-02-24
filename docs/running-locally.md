# Running the website locally
There are some times that one would like to run the page locally, for example if you're adding stuff, here's the gist of how to go at it.

## Dependencies
We are using the SSG called [11ty.dev](11ty.dev) to make it as simple as possible for us to maintain and build this website, which requires some stuff.

1. [NodeJS](https://nodejs.org/) if you don't know which version to go it's usually recommended to use the LTS (Long Time Support) version.
2. ... yeah that's basically it

## Running locally
Clone the project from github, pop out your favourite terminal and move into the directory of this project.

1. run `npm install`, this will create a `node_modules` folder for all the dependencies 11ty requires to run the website and download all the dependencies into it.
2. run `npm run serve` to start a tiny webserver that will host the website at [http://localhost:8080](http://localhost:8080). (If you have something else running on port 8080 it will just try +1 to the port until it finds a free one)

When you have the webserver running, every time you make a change to a file the webserver will update with those changes pretty instantly. Which means that you don't need to restart the webserver after every change.

## Testing your new pages
We also have some kind of testing added, which mostly just checks so that all the links added in our pages are valid and actually points somewhere.

How to run tests:
1. run `npm run test`
2. check output, if it says any link failed, please fix that link in the corresponding file.