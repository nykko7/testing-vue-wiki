Follow up the instructions below to run the project in a local instance.

## Getting Started

Just to be sure you can run the project with no problems, double check that you already have installed the following tools with their respective versions:

```
node ^14.0.0 (preferably, version 14)
git ^2.30.0
npm ^6.5.0
or
yarn ^1.10.0
```

Now, clone the repository from the `master` branch:

```bash
git clone git@gitlab.com:oscity/vpress/talaria-front-end.git
```

Once you have the project on your computer, place yourself on the directory where it is and install dependencies by running: `yarn install` or `npm install`. Then, place yourself in the directory of `functions` and run again: `yarn install` or `npm install`.

## Configuration files

First on the root folder 📁 create a _.env_ file to configure the environment variables, many of those variables are API KEYS needed to run correctly the project. First, in the root folder, your _.env_ file will look like this: 

```js
FIREBASE_API_KEY = "value"
FIREBASE_AUTH_DOMAIN = "value"
FIREBASE_PROJECT_ID = "value"
FIREBASE_STORAGE_BUCKET = "value"
FIREBASE_MESSAGING_SENDER_ID = "value"
FIREBASE_APP_ID = "value"
LOCAL_FIREBASE_API_URL = "value"
```

Now, create another _.env_ file to configure the environment variables for the backend of the application. We are using our backend inside deployable triggered functions, all of which, reside in the functions folder 📁 . The _.env_ file will look like this: 

```js
JWT_KEY = "value"
SENDGRID_API_KEY = "value"
EMAIL_SENDER = "value"
EMAIL_NAME = Talaria
EMAIL_TEMPLATE_ID = "value"
FIREBASE_PROJECT_ID = "value"

CLOUD_SQL_CONNECTION_NAME = "value"
DB_HOST = "value"
DB_USER = "value"
DB_PASS = "value"
DB_NAME = "value"
```

To get the values of this variables, ask one of the mantainers of the project for them, or if you're already a mantainer, you can check their values [here](https://gitlab.com/oscity/vpress/talaria-front-end/-/settings/ci_cd).

## Configuration of firestore emulator

To use all the functionalities that Firebase provides (like Firestore) locally on your computer, you need to install the Firestore Emulator. Before installing the Emulator Suite you will need:

```
node ^8.0.0
java ^1.8
```

First you need to install the firebase tools to install the emulator. You can do this with: 

```bash
npm install -g firebase-tools
```

Now that the Firebase Tools are installed, you will need to sing in into your Google Account and have the project in your console. If you don't have it already, ask one of the maintainers of the project to give you permissions. After this, initialize the working directory as a Firebase project, following the onscreen prompts to specify which products to use, in this case, Talaria. 

```bash
firebase init
```

Finally, set up the emulator suit with the following command: 

```bash
firebase init emulators
```

With this, you can now run `yarn emulator` in another terminal and use all the tools and functionalities of Firestore locally in your computer. For help and more information you can go to this [link](https://firebase.google.com/docs/emulator-suite/install_and_configure).

## Configuration of SQL functionalities

For some of the backend functions, we use SQL to make some requests to a database for a specific project. To configure and test these functions locally, you need to install a proxy that lets you create, request, and alter databases from your computer. To install this proxy, you will need to pick the correct installation depending of your operating system. For example, for 64bit macOS, run this command in the root of your computer (not the project): 

```bash
curl -o cloud_sql_proxy https://dl.google.com/cloudsql/cloud_sql_proxy.darwin.amd64
```

After it's installed, run the next command: 

```bash
chmod +x cloud_sql_proxy
```

For a list with other operating systems, click [here](https://cloud.google.com/sql/docs/mysql/sql-proxy#install).

Next, you'll need to configure Google Cloud SDK for the proxy to work correctly. You can install this via Brew, with the next command: 

```bash
brew install --cask google-cloud-sdk
```

Once you do that, you need to login into your Google Account. Finally, from your terminal, open the `.zshrc` file from your computer and add in this file the next lines of code, which will let SQL Connections work correctly: 

```
source "$(brew --prefix)/Caskroom/google-cloud-sdk/latest/google-cloud-sdk/path.zsh.inc"
source "$(brew --prefix)/Caskroom/google-cloud-sdk/latest/google-cloud-sdk/completion.zsh.inc"
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && . "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

Save your file, and with this, the project Talaria is finally ready to use locally in your computer. :) 

## Running the project

After setting up the configurations you can start the project by running `yarn dev`. This will run the frontend of the application.

To run the backend of the application (cloud functions), in another terminal, in the folder of the project, run `yarn emulator`.

Finally, for the SQL part of the project, in your root folder of your computer (not the project), open a terminal and run the next command: 

```bash
./cloud_sql_proxy -instances=talaria-digital-staging:us-central1:talaria-staging=tcp:3306
```