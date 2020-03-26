# Building your first Fullstack Serverless App with Vue and AWS Amplify

In this workshop we'll learn how to build cloud-enabled web applications with Vue & [AWS Amplify](https://aws-amplify.github.io/).

![](https://i.imgur.com/h4AaEBB.png)

### Topics we'll be covering:

- [Authentication](#adding-authentication)
- [GraphQL API with AWS AppSync](#adding-a-graphql-api)
- [Mocking and Testing](#local-mocking-and-testing)
- [Predictions](#adding-predictions)
- [Hosting](#hosting)
- [Multiple Environments](#working-with-multiple-environments)
- [Deploying via the Amplify Console](#amplify-console)
- [Removing / Deleting Services](#removing-services)

## Pre-requisites

- Node: `12.11.0`. Visit [Node](https://nodejs.org/en/download/current/)
- npm: `6.11.3`. Packaged with Node otherwise run upgrade

```bash
npm install -g npm
```

## Getting Started - Creating the Application

To get started, we first need to create a new Vue project & change into the new directory using the [Vue CLI](https://github.com/vuejs/vue-cli).

If you already have it installed, skip to the next step. If not, either install the CLI & create the app or create a new app using:

```bash
npm install -g @vue/cli
vue create amplify-app
```

Vue CLI
- ? Please pick a preset: __default (babel, eslint)__
 
Now change into the new app directory and make sure it runs

```bash
cd amplify-app
npm run serve
```

## Installing the CLI & Initializing a new AWS Amplify Project

Let's now install the AWS Amplify API & AWS Amplify Vue library:

```bash
npm install --save aws-amplify aws-amplify-vue
```
> If you have issues related to EACCESS try using sudo: `sudo npm <command>`.

### Installing the AWS Amplify CLI

Next, we'll install the AWS Amplify CLI:

```bash
npm install -g @aws-amplify/cli
```
> If your installation fails. Try `npm install -g @aws-amplify/cli --unsafe-perm=true`.
> If you have issues related to fsevents with npm install. Try: `npm audit fix --force`.

Now we need to configure the CLI with our credentials:

```js
amplify configure
```

> If you'd like to see a video walkthrough of this configuration process, click [here](https://www.youtube.com/watch?v=fWbM5DLh25U).

Here we'll walk through the `amplify configure` setup. Once you've signed in to the AWS console, continue:
- Specify the AWS Region: __eu-central-1 (Frankfurt)__
- Specify the username of the new IAM user: __amplify-app__
> In the AWS Console, click __Next: Permissions__, __Next: Tags__, __Next: Review__, & __Create User__ to create the new IAM user. Then, return to the command line & press Enter.
- Enter the access key of the newly created user:   
  accessKeyId: __(<YOUR_ACCESS_KEY_ID>)__   
  secretAccessKey:  __(<YOUR_SECRET_ACCESS_KEY>)__
- Profile Name: __default__

> To view the new created IAM User go to the dashboard at [https://console.aws.amazon.com/iam/home#/users/](https://console.aws.amazon.com/iam/home#/users/). Also be sure that your region matches your selection.

### Initializing A New Project

```bash
amplify init
```

- Enter a name for the project: __amplify-app__
- Enter a name for the environment: __dev__
- Choose your default editor: __Visual Studio Code__   
- Please choose the type of app that you're building __javascript__   
- What javascript framework are you using __vue__   
- Source Directory Path: __src__   
- Distribution Directory Path: __dist__   
- Build Command: __npm run-script build__   
- Start Command: __npm run-script serve__
- Do you want to use an AWS profile? __Yes__
- Please choose the profile you want to use __default__

Now, the AWS Amplify CLI has iniatilized a new project & you will see a new folder: __amplify__. The files in this folder hold your project configuration.

```bash
<amplify-app>
    |_ amplify
      |_ .config
      |_ #current-cloud-backend
      |_ backend
      team-provider-info.json
```

## Adding Authentication

To add authentication to our Amplify project, we can use the following command:

```sh
amplify add auth
```

> When prompted choose 
- Do you want to use default authentication and security configuration?: __Default configuration__
- How do you want users to be able to sign in when using your Cognito User Pool?: __Username__
- Do you want to configure advanced settings? __Yes, I want to make some additional changes.__
- What attributes are required for signing up? (Press &lt;space&gt; to select, &lt;a&gt; to 
toggle all, &lt;i&gt; to invert selection): __Email__
- Do you want to enable any of the following capabilities? (Press &lt;space&gt; to select, &lt;a&gt; to toggle all, &lt;i&gt; to invert selection): __None__

> To select none just press `Enter` in the last option.

Now, we'll run the push command and the cloud resources will be created in our AWS account.

```bash
amplify push

Current Environment: dev

| Category | Resource name      | Operation | Provider plugin   |
| -------- | ------------------ | --------- | ----------------- |
| Auth     | amplifyappuuid     | Create    | awscloudformation |
? Are you sure you want to continue? Yes
```


To quickly check your newly created __Cognito User Pool__ you can run

```bash
amplify status
```

> To access the __AWS Cognito Console__ at any time, go to the dashboard at [https://console.aws.amazon.com/cognito/](https://console.aws.amazon.com/cognito/). Also be sure that your region is set correctly.

### Configuring the Vue Application

Now, our resources are created & we can start using them!

The first thing we need to do is to configure our Vue application to be aware of our new AWS Amplify project. We can do this by referencing the auto-generated `aws-exports.js` file that is now in our `src` folder.

To configure the app, open __main.js__ and add the following code below the last import:

```js
import Vue from 'vue'
import App from './App.vue'
import Amplify, * as AmplifyModules from 'aws-amplify'
import { AmplifyPlugin } from 'aws-amplify-vue'
import awsconfig from './aws-exports'
Amplify.configure(awsconfig)

Vue.use(AmplifyPlugin, AmplifyModules)

Vue.config.productionTip = false

new Vue({
  render: h => h(App),
}).$mount('#app')
```

Now, our app is ready to start using our AWS services.

### Using the Authenticator Component

AWS Amplify provides UI components that you can use in your App. Let's add these components to the project

In order to use the Authenticator Component add it to __src/App.vue__:

```html
<template>
  <div id="app">
    <amplify-authenticator></amplify-authenticator>
  </div>
</template>
```

Now, we can run the app and see that an Authentication flow has been added in front of our App component. This flow gives users the ability to sign up & sign in.

> To view any users that were created, go back to the __Cognito__ dashboard at [https://console.aws.amazon.com/cognito/](https://console.aws.amazon.com/cognito/). Also be sure that your region is set correctly.

Alternatively we can also use

```bash
amplify console auth
```

### Accessing User Data

We can access the user's info now that they are signed in by calling `currentAuthenticatedUser()` which returns a Promise.

```js
<script>
import { Auth } from 'aws-amplify';

export default {
  name: 'app',
  data() {
    return {
      user: { },
    }
  },
  methods: {
    currentUser() {
      Auth.currentAuthenticatedUser().then(user => {
        this.user = user;
        console.log(user);
      })
    }
  }
}
</script>
```

### Custom authentication strategies

The `Authenticator` Component is a really easy way to get up and running with authentication, but in a real-world application we probably want more control over how our form looks & functions.

Let's look at how we might create our own authentication flow.

To get started, we would probably want to create input fields that would hold user input data in the state. For instance when signing up a new user, we would probably need 2 inputs to capture the user's email & password.

To do this, we could create a form like:

```html
<form v-on:submit.prevent>
  <div>
    <label>Username: </label>
    <input v-model='form.username' class='input' />
  </div>
  <div>
    <label>Password: </label>
    <input v-model='form.password' type="password" />
  </div>
  <button @click='signIn' class='button'>Sign In</button>
</form>
```

We'd also need to have a method that signed up & signed in users. We can us the Auth class to do this. The Auth class has over 30 methods including things like `signUp`, `signIn`, `confirmSignUp`, `confirmSignIn`, & `forgotPassword`. These functions return a promise so they need to be handled asynchronously.

```js
<script>
import { Auth } from 'aws-amplify';

export default {
  name: 'login',
  data() {
    return {
      form: {
        username: '',
        password: ''
      }
    }
  },
  methods: {
    signIn() {
      const { username, password } = this.form;
      Auth.signIn(username, password).then(user => {
        console.log('User signed in');
      })
      .catch((error) => console.log(`Error: ${error}`))
    }
  }
}
</script>
```

## Adding a GraphQL API

To add a GraphQL API, we can use the following command:

```sh
amplify add api
```

Answer the following questions

- Please select from one of the below mentioned services __GraphQL__
- Provide API name: __RestaurantAPI__
- Choose the default authorization type for the API __API key__
- Enter a description for the API key: __(empty)__
- After how many days from now the API key should expire (1-365): __180__
- Do you want to configure advanced settings for the GraphQL API __Yes, I want to make some additional changes.__
- Choose the additional authorization types you want to configure for the API (Press &lt;space&gt; to select, &lt;a&gt; to 
toggle all, &lt;i&gt; to invert selection) __None__
- Do you have an annotated GraphQL schema? __No__
- Do you want a guided schema creation? __Yes__
- What best describes your project: __Single object with fields (e.g., “Todo” with ID, name, description)__
- Do you want to edit the schema now? __Yes__

> To select none just press `Enter`.


> When prompted, update the schema to the following:   

```graphql
type Restaurant @model {
  id: ID!
  clientId: String
  name: String!
  description: String!
  city: String!
}
```

> Note: Don't forget to save the changes to the schema file!

Next, let's push the configuration to our account:

```bash
amplify push
```

- Are you sure you want to continue? __Yes__
- Do you want to generate code for your newly created GraphQL API __Yes__
- Choose the code generation language target __javascript__
- Enter the file name pattern of graphql queries, mutations and subscriptions __src/graphql/**/*.js__
- Do you want to generate/update all possible GraphQL operations - queries, mutations and subscriptions __Yes__
- Enter maximum statement depth [increase from default if your schema is deeply nested] __2__

Notice your __GraphQL endpoint__ and __API KEY__.

This step created a new AWS AppSync API. Use the command below to access the AWS AppSync dashboard. Make sure that your region is correct.

```bash
amplify console api
```

- Please select from one of the below mentioned services __GraphQL__

### Local mocking and testing

To mock and test the API locally, you can run the `mock` command:

```sh
amplify mock api
```
- Choose the code generation language target: __javascript__
- Enter the file name pattern of graphql queries, mutations and subscriptions: __src/graphql/**/*.js__
- Do you want to generate/update all possible GraphQL operations - queries, mutations and subscriptions: __Y__
- Enter maximum statement depth [increase from default if your schema is deeply nested]: __2__

This should open up the local GraphiQL editor.

From here, we can now test the API locally.

### Adding mutations from within the AWS AppSync Console

In the AWS AppSync console, on the left side click on Queries.

Execute the following mutation to create a new restaurant in the API:

```graphql
mutation createRestaurant {
  createRestaurant(input: {
    name: "Nobu"
    description: "Great Sushi"
    city: "New York"
  }) {
    id name description city
  }
}
```

Now, let's query for the restaurant:

```graphql
query listRestaurants {
  listRestaurants {
    items {
      id
      name
      description
      city
    }
  }
}
```

We can even add search / filter capabilities when querying:

```graphql
query searchRestaurants {
  listRestaurants(filter: {
    city: {
      contains: "New York"
    }
  }) {
    items {
      id
      name
      description
      city
    }
  }
}
```

### Interacting with the GraphQL API from our client application - Querying for data

Now that the GraphQL API is created we can begin interacting with it!

The first thing we'll do is perform a query to fetch data from our API. 

To do so, we need to define the query, execute the query, store the data in our state, then list the items in our UI.

> Read more about the __Amplify GraphQL Client__ [here](https://aws-amplify.github.io/docs/js/api#amplify-graphql-client).

```js
<template>
  <div v-for="restaurant of restaurants" :key="restaurant.id">
    {{restaurant.name}}
  </div>
</template>
<script>
import { API, graphqlOperation } from 'aws-amplify';
import { listRestaurants } from './graphql/queries';

export default {
  name: 'app',
  data() {
    return {
      restaurants: [],
    }
  },
  created() {
    const response = await API.graphql(graphqlOperation(listRestaurants));
    this.restaurants = response.data.listRestaurants.items;
  },
}
</script>
```

## Performing mutations

 Now, let's look at how we can create mutations.

```js
<template>
  <div>
    <form v-on:submit.prevent>
      <div>
        <label>Name: </label>
        <input v-model='form.name' class='input' />
      </div>
      ...
      <button @click='createRestaurant' class='button'>Create</button>
    </form>
  </div>
</template>
<script>
import { createRestaurant } from './graphql/mutations';

export default {
  name: 'app',
  data() {
    return {
      form: { },
      clientId: null
    }
  },
  created() {
    this.clientId = uuid();
  },
  methods: {
    async createRestaurant() {
      try {
        const { name, description, city } = this.form;
        const restaurant = { name, description, city, clientId: this.clientId };
        const response = await API.graphql(
          graphqlOperation(createRestaurant, { input: restaurant })
        );
        this.restaurants = [...this.restaurants, response.data.createRestaurant];
        this.form = { name: '', description: '', city: '' };
        console.log('item created!')
      } catch (err) {
        console.log(err)
      }
    }
  }
}
</script>
```

### GraphQL Subscriptions

Next, let's see how we can create a subscription to subscribe to changes of data in our API.

To do so, we need to listen to the subscription, & update the state whenever a new piece of data comes in through the subscription.

```js
import { onCreateRestaurant } from './graphql/subscriptions';

export default {
  name: 'app',
  created() {
    //Subscribe to changes
    API.graphql(graphqlOperation(onCreateRestaurant))
    .subscribe((sourceData) => {
      const newRestaurant = sourceData.value.data.onCreateRestaurant
      if (newRestaurant) {
        // skip our own mutations and duplicates
        if (newRestaurant.clientId == this.clientId) return;
        if (this.restaurants.some(r => r.id == newRestaurant.id)) return;
        this.restaurants = [newRestaurant, ...this.restaurants];
      } 
    });
  },
}
```

## Adding Predictions

To add the predictions category to our Amplify project, we can use the following command:

```sh
amplify add predictions
```

#### Identify Text, Labels and Entities
- Please select from one of the categories below __Identify__
- What would you like to identify? __Identify Text__
- Provide a friendly name for your resource __identifyTextId__
- Would you also like to identify documents? __No__
- Who should have access? __Auth and Guest users__

#### Translate Text
- Please select from one of the categories below __Convert__
- What would you like to convert? __Translate text into a different language__
- Provide a friendly name for your resource __translateTextId__
- What is the source language? __English__
- What is the target language? __Spanish__
- Who should have access? __Auth and Guest users__

#### Generate Speech
- Please select from one of the categories below __Convert__
- What would you like to convert? __Generate speech audio from text__
- Provide a friendly name for your resource __speechGeneratorId__
- What is the source language? __British English__
- Select a speaker __Brian - Male__
- Who should have access? __Auth and Guest users__

#### Transcribe Audio
- Please select from one of the categories below __Convert__
- What would you like to convert? __Transcribe text from audio__
- Provide a friendly name for your resource __transcriptiondId__
- What is the source language? __British English__
- Who should have access? __Auth and Guest users__

#### Interpret Text
- Please select from one of the categories below __Interpret__
- What would you like to interpret? __Interpret Text__
- Provide a friendly name for your resource __interpretTextId__
- What kind of interpretation would you like? __All__
- Who should have access? __Auth and Guest users__


Now, we'll run the push command and the cloud resources will be created in our AWS account.

```bash
amplify push
```

### Configuring the Vue Application

Now, our resources are created & we can start using them!

To configure the app, open __main.js__ and add the following code below the last import:

```js
import Predictions, { AmazonAIPredictionsProvider } from '@aws-amplify/predictions';
Predictions.addPluggable(new AmazonAIPredictionsProvider());
```

Now, our app is ready to start using **Predictions**.

#### Translate Example

The result of `Predictions.convert(config)` will be a promise. If successful will return the translation otherwise will return the input as-is.

```
Predictions.convert({
  translateText: {
    source: {
      text: "My taylor is rich!",
      language: "en"
    },
    targetLanguage: "es"
  }
})
.then(({text}) => {
  this.translation = text;
})
```
> List of [supported languages](https://docs.aws.amazon.com/translate/latest/dg/how-it-works.html#how-it-works-language-codes) to translate from.

#### Detect Language Example

The result of `Predictions.interpret(config)` call will return a promise. If successful, we will be able to pick the main language as part of the results as demonstrated in the following code snipet.

```
Predictions.interpret({
  text: {
    source: {
      text: textToInterpret,
    },
    type: InterpretTextCategories.LANGUAGE
  }
})
.then((result) => {
  let detected = result.textInterpretation.language;
})
```

#### Text to Speech Example

The result of `Predictions.convert(config)` call will return a promise with the resulting audio file which we can reference to play the voice using *HMTL Audio*.

```
Predictions.convert({
  textToSpeech: {
    source: {
      text: textToTranslate,
    },
    voiceId: "Brian"
  }
})
.then((result) => {
  if (result.speech.url) {
    this.audio = new Audio();
    this.audio.src = result.speech.url;
    this.audio.play();
  }
})
```

## Hosting

To deploy & host your app on AWS, we can use the `hosting` category.

```sh
amplify add hosting
```

- Select the environment setup: __DEV (S3 only with HTTP)__
- hosting bucket name __YOURBUCKETNAME__
- index doc for the website __index.html__
- error doc for the website __index.html__

Now, everything is set up & we can publish it:

```sh
amplify publish
```

## Working with multiple environments

You can create multiple environments for your application in which to create & test out new features without affecting the main environment which you are working on.

When you create a new environment from an existing environment, you are given a copy of the entire backend application stack from the original project. When you make changes in the new environment, you are then able to test these new changes in the new environment & merge only the changes that have been made since the new environment was created back into the original environment.

Let's take a look at how to create a new environment. In this new environment, we'll re-configure the GraphQL Schema to have another field for the pet owner.

First, we'll initialize a new environment using `amplify init`:

```sh
amplify init
```

- Do you want to use an existing environment? __N__
- Enter a name for the environment: __apiupdate__
- Do you want to use an AWS profile? __Y__
- __amplify-workshop-user__

Once the new environment is initialized, we should be able to see some information about our environment setup by running:

```sh
amplify env list

| Environments |
| ------------ |
| dev          |
| *apiupdate   |
```

Now we can update the GraphQL Schema in `amplify/backend/api/RestaurantAPI/schema.graphql` to the following (adding the owner field):

```graphql
type Restaurant @model {
  ...
  owner: String
}
```

Now, we can create this new stack by running `amplify push`:

```sh
amplify push
```

After we test it out, we can now merge it into our original dev environment:

```sh
amplify env checkout dev

amplify status

amplify push
```

- Do you want to update code for your updated GraphQL API? __Y__
- Do you want to generate GraphQL statements? __Y__


## Deploying via the Amplify Console

We have looked at deploying via the Amplify CLI hosting category, but what about if we wanted continous deployment? For this, we can use the [Amplify Console](https://aws.amazon.com/amplify/console/) to deploy the application.

The first thing we need to do is [create a new GitHub repo](https://github.com/new) for this project. Once we've created the repo, we'll copy the URL for the project to the clipboard & initialize git in our local project:

```sh
git init

git remote add origin git@github.com:username/project-name.git

git add .

git commit -m 'initial commit'

git push origin master
```

Next we'll visit the Amplify Console in our AWS account at [https://eu-central-1.console.aws.amazon.com/amplify/home](https://eu-central-1.console.aws.amazon.com/amplify/home).

Here, we'll click __Get Started__ to create a new deployment. Next, authorize Github as the repository service.

Next, we'll choose the new repository & branch for the project we just created & click __Next__.

In the next screen, we'll create a new role & use this role to allow the Amplify Console to deploy these resources & click __Next__.

Finally, we can click __Save and Deploy__ to deploy our application!

Now, we can push updates to Master to update our application.


## Removing Services

If at any time, or at the end of this workshop, you would like to delete a service from your project & your account, you can do this by running the `amplify remove` command:

```sh
amplify remove auth

amplify push
```

If you are unsure of what services you have enabled at any time, you can run the `amplify status` command:

```sh
amplify status
```

`amplify status` will give you the list of resources that are currently enabled in your app.


## Appendix

### Setup your AWS Account

In order to follow this workshop you need to create and activate an Amazon Web Services account. 

Follow the steps [here](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account)

### Trobleshooting

Message: The AWS Access Key Id needs a subscription for the service

Solution: Make sure you are subscribed to the free plan. [Subscribe](https://portal.aws.amazon.com/billing/signup?type=resubscribe#/resubscribed)


Message: TypeError: fsevents is not a constructor

Solution: `npm audit fix --force`
