# How To Migrate From Strapi v3 to v4 Walkthrough

This post will cover the "bird's view" of the Strapi v3 to v4 migration process.

We will cover the general process overview, the steps you must take, and how to use the migration guide to help you succeed effectively.

This article aims to have you follow along with the migration process of our Food Advisor app to understand better what it entails before you do it in your project.

This article is based on our live stream that you can watch here. However, we wanted to make a supplementary article that you can use as a reference when needing to go over the general overview of the process.

We won't be able to cover all the nuances of the process. Still, our goal is to provide enough overview to make you comfortable with the process.

## Outline

- Things To Consider
- the migration process overview
- the migration
  - intro
  - set up the base project
  - set up a reference project
  - migration steps walkthrough
    - step 1: point to a new database
    - step 2: updating the config.js folder
    - step 3: using codemods
    - step 4: updating the dependencies
    - step 5: updating the generic routes
    - step 6: updating the generic controllers
    - step 7: updating the generic services
    - note on errors
- Strapi customizations
  - how to create a custom controller
  - how to create a custom resolver
  - where to find help
- Conclusion


## Things To Consider

Please note that depending on how much of your project is custom will dictate the time and effort it will take to migrate your application. The more custom code you have, the more time it will take.

But to help us along the way, we have a code migration script and a database migration script to assist us in the process and documentation that covers every step of the process.

With that said, there is no "one-click migration" due to the changes that the Strapi team made between Strapi v3 to Strapi v4.

We rewrote most of Strapi's code to build a foundation to support our future features, allowing a more straightforward migration process from v4 to v5.

**Note on GraphQL:** Fundamentally, we used a more programmatic way to handle GraphQL. And the controllers are no longer coupled with the GraphQL resolvers.

Also, due to our new "populate" feature, you can now designate what data you would like to get from your REST API to prevent over-fetching.

Due to these two things and more flexible REST API implementation, you may have to revisit if GraphQL is still a requirement.

If GraphQL is still something you would like to use from your previous version, you would have to rewrite any of your custom resolvers with the required logic.

## The Migration Process Overview

`code migration diagram`

<img width="1112" alt="Screen Shot 2022-07-25 at 2 17 15 PM" src="https://user-images.githubusercontent.com/6153188/181594611-1bbb9e07-6943-4576-af6c-6390567db023.png">


`database migration diagram`

<img width="1361" alt="Screen Shot 2022-07-25 at 2 18 26 PM" src="https://user-images.githubusercontent.com/6153188/181594664-0b3c615d-14e0-452c-8f56-7e1c11858349.png">

In this post, we will only cover the **_code migration_** process, but once this is complete, you will be able to move to the **_data migration_** step in a future post.

Looking at the **_code migration_** diagram, let's go over each step.

## Data Migration Steps

**_Step 1_**: Start with backend code migrations. (make sure you are connecting to a new database)

**_Step 2_**: Manually migrate the configs as shown in the docs.

**_Step 3_**: Use **_codemods_**.

**_Step 4_**: Update dependencies + plugins folder structure can be automatically migrated.

**_Step 5_**: If the app contained any customizations to routes, controllers, or services. it needs to be checked and done manually.

**_Step 6_**: Your app should now build.

We are now left with the final step, **_data migration_**, which we will cover in detail in a future article.

Don't worry if these steps are not yet clear; we will go through them in detail as we go through the migration process.

But you can find the documentation here to help us along in the process, which we will refer to as we make our way through this tutorial.

[Strapi Migration Documentation](https://docs.strapi.io/developer-docs/latest/update-migration-guides/migration-guides/v4/code-migration.html#v4-code-migration-guide)

## The Migration

### Intro

Before we start, you can get the code [here](https://github.com/PaulBratslavsky/demoMigration) if you want to follow along.

We will use a version of the Food Advisor app we used in the stream as a demonstration.

It is an excellent example because it has some customizations that will allow us to review how to handle the changes in your application and where to find additional help.

### Set Up The Base Project

Let's go ahead and clone the Food Advisor Strapi instance by running the following command:

```bash
	git clone https://github.com/PaulBratslavsky/demoMigration.git demomigration

```

After cloning the repo, change directories to `demomigration.`

Make sure you use node version 12 before running `yarn` to install all the dependencies.

If you are using **_NVM_**, you can switch your node version by typing `nvm use 12` and then run `yarn && yarn seed` to seed our data.

Finally, run `yarn build && yarn develop` to start the application.

After everything is done, navigate to `http://localhost:1337/admin`. You should now see the registration page. Go ahead and create an admin account and log in.

<img width="667" alt="strapilogin" src="https://user-images.githubusercontent.com/6153188/181594748-b5c2b164-1b5f-46b8-a6ea-cb91f290cc1d.png">

Once you are logged in, you should be able to see all of content types and data.

<img width="1509" alt="data" src="https://user-images.githubusercontent.com/6153188/181595012-df2ccb88-6add-4f4a-90db-23625df62267.png">


Great success. We now have a Strapi v3 project running that we can use to learn about the migration process.

### Set Up The Reference Project

Before we move on to the next step, let's install a copy of a Strapi 4 project that we can use as a reference. Having it handy will make a few things easier to talk about.

Make sure you create it in a different folder, not in the current project. You can do so by running the command below.

```bash
	npx create-strapi-app@latest reference --quickstart
```

## Migration Streps Walkthrough

### Step 1: Point To a New Database

Before we start the migration process, back in our Food Advisor Strapi v3 demo project, let's remove the `.git` file.

You can do so by running `rm -rf .git` which will remove the previous repo, so you get to initialize your own by running `git init.`

Afterward, you can save the changes by running the following command.

```bash
	git add .
	git commit -m "initial commit"
```

Inside your project, you should see the `.tmp` folder which will have our SQLite database called `data.db`.

Let's rename it to `old_data.db`. Afterward, you can run `yarn develop`, which will create a new blank database for us to use, so we don't have to worry about messing up our data.

You will be prompted to create a new admin user since we are now using a new database.

<img width="667" alt="strapilogin" src="https://user-images.githubusercontent.com/6153188/181594748-b5c2b164-1b5f-46b8-a6ea-cb91f290cc1d.png">

So go ahead and create a new user admin and login. You should have no data in your app.

We will save the `old_data.db` to use in a later post when we cover data migration.

### Step 2: Updating the config.js folder

Next, let's update the `config` folder. Currently, our structure looks like the left side.

![v3vsv4configfolder](https://user-images.githubusercontent.com/6153188/181595424-4f5bec17-55c8-4012-bbbe-b537307eafce.png)

As you can see, there are differences in the folder structure from Strapi v3 to Strapi v4 below.

You can learn more about the changes in the documentation [here](https://docs.strapi.io/developer-docs/latest/update-migration-guides/migration-guides/v4/code/backend/configuration.html).

But here is a brief overview of what changed.

<img width="809" alt="configfoderchanges" src="https://user-images.githubusercontent.com/6153188/181596302-7946c904-02b3-42ca-a8a0-6bf4106912b5.png">

We will make this easy on ourselves.

First, open your v3 project in file explorer, navigate to the `config` folder, and delete all the items.

Then open the reference v4 project. Navigate to your config folder, select all items, and copy them to your v3 project.

![configfolderchanges](https://user-images.githubusercontent.com/6153188/181596447-6b3f6472-0495-470b-8127-a19f3a682fe0.gif)

Your v3 project `config` folder should look like this.

![finalfolderstructure 1](https://user-images.githubusercontent.com/6153188/181596572-b47b49a1-ffd9-4dde-baab-94b8bc0e64ae.gif)

After making the changes, let's save what we just did by running the following command.

```bash
	git add .
	git commit -m "updated the config folder"
```

Before we move to the next step, let's create a separate branch and switch. We need to do this before using **_codemods_**.

You can do so by running the following commands.

```bash
	git branch before-codemods
	git checkout before-codemods
```

### Step 3: Using Codemods

Codemods is a script that will allow you to easily update the folder structure, content schemas, and other items to make the migration process easier.

![codemods](https://user-images.githubusercontent.com/6153188/181596729-d857b753-d5be-4946-841e-06ba9ac85919.png)

You can learn more about it [here](https://github.com/strapi/codemods).

#### What does codemods do?

- updates the folder Structure
- updates models to content-types
- updates setting.json to schema.json
- updates the schema.json

In our newly created branch, let's run the following command.

```bash
	npx @strapi/codemods migrate
```

Next, select the `Application` option.

```bash
	? What do you want to migrate?
	❯ Application
	  Plugin
	  Only Dependencies
```

Next, set enter your root path for your Strapi application `./` and pres enter to continue.

```bash
	? What do you want to migrate? Application
	? Enter the path to your Strapi application ./

```

You can run the following command to see the changes **_codemods_** made.

```bash
	git add . && git diff --cached
```

Let's save our changes by typing the following command.

```bash
	git commit -am "migrate API to v4 structure"
```

### Step 4: Updating the dependencies

After running **_codemods_**, the next step is to update our dependencies in the `package.json` file.

You can learn more [here](https://docs.strapi.io/developer-docs/latest/update-migration-guides/migration-guides/v4/code/backend/dependencies.html)

```json
"dependencies": {
	"body-scroll-lock": "^3.1.5",
	"fs-extra": "8.1.0",
	"knex": "0.16.3",
	"lodash": "^4.17.5",
	"reactour": "^1.18.0",
	"sqlite3": "^4.0.6",
	"unzip-stream": "^0.3.0",
	"@strapi/strapi": "4.2.3",
	"@strapi/admin": "4.2.3",
	"@strapi/plugin-content-manager": "4.2.3",
	"@strapi/plugin-content-type-builder": "4.2.3",
	"@strapi/plugin-email": "4.2.3",
	"@strapi/plugin-graphql": "4.2.3",
	"@strapi/plugin-upload": "4.2.3",
	"@strapi/plugin-users-permissions": "4.2.3",
	"@strapi/utils": "4.2.3"
},
```

We can remove the following.

```json
	"knex": "0.16.3",
	"@strapi/admin": "4.2.3",
	"@strapi/plugin-content-manager": "4.2.3",
	"@strapi/plugin-content-type-builder": "4.2.3",
	"@strapi/plugin-email": "4.2.3",
	"@strapi/plugin-upload": "4.2.3",
	"@strapi/utils": "4.2.3"
```

Our `package.json` file should look like the following.

```json
"dependencies": {
	"body-scroll-lock": "^3.1.5",
	"fs-extra": "8.1.0",
	"lodash": "^4.17.5",
	"reactour": "^1.18.0",
	"sqlite3": "^4.0.6",
	"unzip-stream": "^0.3.0",
	"@strapi/strapi": "4.2.3",
	"@strapi/plugin-graphql": "4.2.3",
	"@strapi/plugin-users-permissions": "4.2.3"
},
```

Before updating our `node_modules` folder, we also need to update our `data.db` file that you can find in the `.tmp` folder.

Go ahead and delete it.

When we run `yarn develop` in the next step, it will create new `data.db` file using the updated schema that was created after running **_codemods_**

We can now run the following commands to update our `node_module` folder based on our new dependencies.

```
	nvm use 16
	rm -rf node_modules
	yarn
	yarn build
	yarn develop
```

You will run into the following error since we are referencing a package whose name has been updated.

```bash
	[2022-07-26 13:16:42.981] debug: ⛔️ Server wasn't able to start properly.
	[2022-07-26 13:16:42.982] error: Cannot find module 'strapi-utils'
```

In our case, the issue is found in the `src/api/universal/config/schema.graphql.js` file.

To fix the error, you will need to replace all references to `'strapi-utils'` with `'@strapi/utils'` and run `yarn develop` again.

Here is the updated file.

```javascript
const { sanitizeEntity } = require("@strapi/utils");

module.exports = {
  query: `
    universalBySlug(id: ID slug: String): Universal
  `,
  resolver: {
    Query: {
      universalBySlug: {
        resolverOf: "Universal.findOne",
        async resolver(_, { slug }) {
          const entity = await strapi.services.universal.findOne({ slug });
          return sanitizeEntity(entity, { model: strapi.models.universal });
        },
      },
    },
  },
};
```

After running `yard develop`, you might see the following error.

```bash
	[2022-07-26 13:36:08.815] debug: ⛔️ Server wasn't able to start properly.
	[2022-07-26 13:36:08.816] error: Knex: run
$ npm install sqlite3 --save
```

To fix this, follow these steps.

1.  run `yarn add sqlite3`
2.  remove the `data.db` file in the `.tmp` folder.
3.  and run `yarn develop` again.

We should now see a different error, which is good.

```bash
	[2022-07-26 13:45:31.590] error: Middleware "strapi::session": App keys are required. Please set app.keys in config/server.js (ex: keys: ['myKeyA', 'myKeyB'])
```

We now have to update our `.env` file with additional variables that Strapi v4 is now requiring.

In the `.env` file paste the following code.

```env
ADMIN_JWT_SECRET=LnEkeTvz3prxACbXGzQpWQ==JWT_SECRET=7+FHjdxRrJu7opx1NMBScw==
JWT_SECRET=MOCWJRO8sbglc5Pxgx6s+g==
APP_KEYS=rrigJL/NdvVc3Wu6GEm85w==,gHa+ZYdvSPpT/f8iqJewiw==,ddiMVxdXh0aZgN+OSddGSw==,jLixvXdrbKqZcUFuytFzIQ==
API_TOKEN_SALT=m/gqLSE7A+YShQgVeA+45A==
```

All these errors, but this is good since we are getting new errors that are helping us figure out what we have left to migrate.

```bash
	[2022-07-26 13:49:22.831] error: Error creating endpoint GET /categories: Cannot read properties of undefined (reading 'find')
	TypeError: Error creating endpoint GET /categories: Cannot read properties of undefined (reading 'find')
```

This new error means we need to update our routes to use the strapi factory functions.

In the future, this is something that **_codemods_** will handle.

But at the moment, we have to do this manually, which also applies to our controllers and services.

### Step 5: Updating The Generic Routes

We are getting closer. It is time to update our routes using Strapi's factory function to generate our routes.

You can read more about the difference between v3 and v4 routes [here](https://docs.strapi.io/developer-docs/latest/update-migration-guides/migration-guides/v4/code/backend/routes.html).

Let's take a quick look at our `api` folder to see all our content types where we will have to make the changes.

We will have to update the routes, controllers, and services in each content type.

**_note_**: in the future, this will be done with **_codemods_**, but as of this writing, we have to make these changes manually.

<img width="481" alt="apifolder" src="https://user-images.githubusercontent.com/6153188/181596988-b0a5838b-cbbe-4b73-bf60-99ae36f77638.png">


Let's use the `restaurant` folder as our example. What we will do here is what you must do in all other folders in the `api` folder.

<img width="461" alt="routescontrollersandservices" src="https://user-images.githubusercontent.com/6153188/181597032-d364e926-bbdf-45e1-adec-b6f5683e4514.png">

Let's look at the `restaurant.js` file inside the `routes` folder.

`restaurant/routes/restaurant.js`

```javascript
module.exports = {
  routes: [
    {
      method: "GET",
      path: "/restaurants",
      handler: "Restaurant.find",
      config: { policies: [] },
    },
    {
      method: "GET",
      path: "/restaurants/:id",
      handler: "Restaurant.findOne",
      config: { policies: [] },
    },
    {
      method: "POST",
      path: "/restaurants",
      handler: "Restaurant.create",
      config: { policies: [] },
    },
    {
      method: "PUT",
      path: "/restaurants/:id",
      handler: "Restaurant.update",
      config: { policies: [] },
    },
    {
      method: "DELETE",
      path: "/restaurants/:id",
      handler: "Restaurant.delete",
      config: { policies: [] },
    },
  ],
};
```

As we can see, this is how our generic routes were defined in Strapi v3. We will need to change this to use Strapi's factory function.

You can learn more [here](https://docs.strapi.io/developer-docs/latest/update-migration-guides/migration-guides/v4/code/backend/routes.html#migrating-core-routers).

Here is an example of how we define routes in Strapi v4.

```javascript
// path: ./src/api/<content-type-name>/routes/<router-name>.js

const { createCoreRouter } = require("@strapi/strapi").factories;

module.exports = createCoreRouter("api::api-name.content-type-name");
```

Let's update our restaurant routes using the above example.

Our `restaurant.js` file inside the `routes` folder should look like this.

`restaurant/routes/restaurant.js`

```javascript
// path: ./src/api/restaurant/routes/restaurant.js

const { createCoreRouter } = require("@strapi/strapi").factories;

module.exports = createCoreRouter("api::restaurant.restaurant");
```

Make sure you replace the appropriate `api-name` and `content-type-name`. It will be the same as the folder name. In this case, it is `restaurant`.

Make sure you replace the remaining routes in the rest of the content types.

I will link to a repo where you can see the final changes just in case you run into issues.

After replacing all the routes, you will have a new error when you run `yarn develop`.

```bash
	[2022-07-26 18:15:46.325] error: Error creating endpoint GET /categories: Handler not found "api::category.category.find"
	Error: Error creating endpoint GET /categories: Handler not found "api::category.category.find"
```

This error means we cannot find the proper controllers, so let's fix that in the next section.

### Step 6: Updating The Generic Controllers

We have to update the controllers to use Strapi's factory function to generate our controllers.

Go inside the `restaurant/controllers` folder and look inside the `restaurant.js` file.

You will still see the old code from Strapi v3.

```javascript
module.exports = {
  find: async (ctx) => {
    let restaurants;

    if (ctx.query._q) {
      restaurants = await strapi.api.restaurant.services.restaurant.search(
        ctx.query
      );
    } else {
      restaurants = await strapi.api.restaurant.services.restaurant.find(
        ctx.query
      );
    }

    restaurants = await Promise.all(
      restaurants.map(async (restaurant) => {
        restaurant.note = await strapi.api.review.services.review.average(
          restaurant.id
        );

        return restaurant;
      })
    );

    return restaurants;
  },

  findOne: async (ctx) => {
    const { id } = ctx.params;
    let restaurant = await strapi.api.restaurant.services.restaurant.findOne({
      id,
    });

    if (!restaurant) {
      return ctx.notFound();
    }

    restaurant.note = await strapi.api.review.services.review.average(
      restaurant.id
    );

    let noteDetails = await strapi
      .query("review")
      .model.query(function (qb) {
        qb.where("restaurant", "=", restaurant.id);
        qb.groupBy("note");
        qb.select("note");
        qb.count();
      })
      .fetchAll()
      .then((res) => res.toJSON());

    restaurant.noteDetails = [];

    for (let i = 5; i > 0; i--) {
      let detail = noteDetails.find((detail) => {
        return detail.note === i;
      });

      if (detail) {
        detail = {
          note: detail.note,
          count: detail["count(*)"],
        };
      } else {
        detail = {
          note: i,
          count: 0,
        };
      }

      restaurant.noteDetails.push(detail);
    }

    return restaurant;
  },
};
```

Oh no, it's a custom controller, what are we going to do?

For now, we will comment it out, and replace it with a generic controller using the Strapi's factory function as it is mentioned [here](https://docs.strapi.io/developer-docs/latest/update-migration-guides/migration-guides/v4/code/backend/controllers.html).

We will come back later and talk about customizations. But for now, let's at least get to a point where we can build our app without errors.

Below is the example of a generic controller.

```javascript
// path: ./src/api/<content-type-name>/controllers/<controller-name>.js

const { createCoreController } = require("@strapi/strapi").factories;

module.exports = createCoreController("api::api-name.content-type-name");
```

As you can see, it's similar to the changes we made in the routes.

Generic **_routes_**, **_controllers_**, and **_services_** are now generated via factory functions.

Inside the `restaurant/controllers/Restaurant.js` file, let's make the following changes.

```javascript
// path: ./src/api/restaurant/controllers/restaurant.js

const { createCoreController } = require("@strapi/strapi").factories;

module.exports = createCoreController("api::restaurant.restaurant");

// will come back to this later

// module.exports = {
//   find: async (ctx) => {
//     let restaurants;

//     if (ctx.query._q) {
//       restaurants = await strapi.api.restaurant.services.restaurant.search(ctx.query);
//     } else {
//       restaurants = await strapi.api.restaurant.services.restaurant.find(ctx.query);
//     }

//     restaurants = await Promise.all(
//       restaurants.map(async (restaurant) => {
//         restaurant.note = await strapi.api.review.services.review.average(restaurant.id);

//         return restaurant;
//       })
//     );

//     return restaurants;
//   },

//   findOne: async (ctx) => {
//     const { id } = ctx.params;
//     let restaurant = await strapi.api.restaurant.services.restaurant.findOne({ id });

//     if (!restaurant) {
//       return ctx.notFound();
//     }

//     restaurant.note = await strapi.api.review.services.review.average(restaurant.id);

//     let noteDetails = await strapi
//       .query('review')
//       .model.query(function (qb) {
//         qb.where('restaurant', '=', restaurant.id);
//         qb.groupBy('note');
//         qb.select('note');
//         qb.count();
//       })
//       .fetchAll()
//       .then((res) => res.toJSON());

//     restaurant.noteDetails = [];

//     for (let i = 5; i > 0; i--) {
//       let detail = noteDetails.find((detail) => {
//         return detail.note === i;
//       });

//       if (detail) {
//         detail = {
//           note: detail.note,
//           count: detail['count(*)'],
//         };
//       } else {
//         detail = {
//           note: i,
//           count: 0,
//         };
//       }

//       restaurant.noteDetails.push(detail);
//     }

//     return restaurant;
//   }
// };
```

Make sure you replace the appropriate `api-name` and `content-type-name`. It will be the same as the folder name. In this case, it is `restaurant`.

Make sure you replace the remaining controllers in the rest of the content types.

If there is a custom controller, for the time being, comment out the code, we will revisit this in a bit.

### Step 7: Updating The Generic Services

Finally, let's update our services to use Strapi's factory functions.

You can learn more [here]().

But the pattern is similar to what we just did for `routes` and `controllers`.

Check out the code below for an example of a generic service.

```javascript
// path: ./src/api/<content-type-name>/services/<service-name>.js

const { createCoreService } = require("@strapi/strapi").factories;

module.exports = createCoreService("api::api-name.content-type-name");
```

Go inside the `restaurant/services` folder and look inside the `Restaurant.js` file.

You will see the following code.

```javascript
module.exports = ({ strapi }) => {
  return {};
};
```

Let's replace it with the following code.

```javascript
// path: ./src/api/restaurant/services/restaurant.js

const { createCoreService } = require("@strapi/strapi").factories;

module.exports = createCoreService("api::restaurant.restaurant");
```

Make sure you replace the remaining services in the rest of the content types.

You will notice that `review/services/Review.js` has custom service logic.

For now, comment it out, and replace it with the generic code.

We will talk about this in the customization section.

Once all the services have been refactored, run `yarn develop` to see if we get any more errors.

Another error. At least it is different.

```bash
	[2022-07-26 19:20:59.256] error: GraphQL Nexus: Enum ENUM_RESTAURANT_PRICE must have at least one member
	Error: GraphQL Nexus: Enum ENUM_RESTAURANT_PRICE must have at least one member
```

We will fix this in just a moment.

### Note On Errors

You will undoubtedly run into codebase-specific errors when working on your migration; this is normal, and you will have to debug.

As in our case, now we have an issue in our `schema.json` file located in `api/restaurant/content-types/Restaurant/schema.json`.

We have an ENUM that starts with a number, which is not allowed and has to be fixed.

Let's make the following change from this.

```json
	"price": {
		"type": "enumeration",
			"enum": [
				"_1",
				"_2",
				"_3",
				"_4"
			]
	},
	"district": {
		"type": "enumeration",
			"enum": [
				"_1st",
				"_2nd",
				"_3rd",
				"_4th",
				"_5th",
				"_6th",
				"_7th",
				"_8th",
				"_9th",
				"_10th",
				"_11th",
				"_12th",
				"_13th",
				"_14th",
				"_15th",
				"_16th",
				"_17th",
				"_18th",
				"_19th",
				"_20th"
			]
	},
```

To this.

```json
	"price": {
		"type": "enumeration",
			"enum": [
				"one",
				"two",
				"three",
				"four"
				]
		},

	"district": {
		"type": "enumeration",
			"enum": [
				"first",
				"second",
				"third",
				"fourth",
				"fifth",
				"sixth",
				"seventh",
				"eighth",
				"ninth",
				"tenth",
				"eleventh",
				"twelfth",
				"thirteenth",
				"fourteenth",
				"fifteenth",
				"sixteenth",
				"seventeenth",
				"eighteenth",
				"nineteenth",
				"twentieth"
			]
	},

```

The above should fix our error. But before we run `yarn develop`, there is one more file that we have to add.

The dedicated `bootstrap.js` file no longer exists in Strapi v4 and is now a global function combined with the new `register` function. `bootstrap()` and `register()` can be found in `./src/index.js`

Inside our `src` folder, let's create an `index.js` file and add the following code.

```javascript
"use strict";

module.exports = {
  /**
   * An asynchronous register function that runs before
   * your application is initialized.
   *
   * This gives you an opportunity to extend code.
   */

  register(/*{ strapi }*/) {},

  /**
   * An asynchronous bootstrap function that runs before
   * your application gets started.
   *
   * This gives you an opportunity to set up your data model,
   * run jobs, or perform some special logic.
   */

  bootstrap(/*{ strapi }*/) {},
};
```

You can learn more about it [here](https://docs.strapi.io/developer-docs/latest/update-migration-guides/migration-guides/v4/code/backend/configuration.html#custom-functions-folder).

Let's run `yarn develop` and see what happens.

<img width="816" alt="strapi4screen" src="https://user-images.githubusercontent.com/6153188/181597372-ee739bf6-42d2-4916-a777-3907139ef10a.png">

Great success. Our app builds.

You can now create an admin user and log in.

<img width="1506" alt="strapiadmin" src="https://user-images.githubusercontent.com/6153188/181597418-3b600308-acb6-4b9e-890d-9291f7d9f8cc.png">

We still need to go through our codebase and manually update any of our custom **_routes_**, **_controllers_**, **_services_**, and any other customizations you may have.

The good news is that we are running Strapi v4, and although we can continue to reference the migration guide found [here](https://docs.strapi.io/developer-docs/latest/update-migration-guides/migration-guides/v4/code-migration.html).

We can use strapi v4 documentation from this point on, which you can find [here](https://docs.strapi.io/developer-docs/latest/development/backend-customization.html).

## Strapi Customizations

As you noticed in our Food Advisor demo application, we have a lot of customizations, including **_routes_**, **_policies_**, **_controllers_**, **_services_**, and more.

To go over all these items in detail will take a long time. But, the good news is it's already in our Strapi v4 documentation.

This post will show how to implement a custom **_controller_** and **_GraphQl resolver_** and where to go for help in the documentation when you are stuck.

We also have our Discord community, with a channel dedicated to migration from Strapi v3 to v4.

It's a great place to ask questions and work with others. If you are not yet a member, I highly recommend it.

You can join or Discord Community [here](https://discord.com/invite/strapi)

But the goal is not to show everything but to empower you to feel comfortable using our documentation and Discord community to find solutions on your own.

We feel this is the best approach since it promotes learning and sharing. That said, we will release additional video resources covering these topics in greater detail.

### How to create a custom controller

Let's take a look at how we can create a custom controller. The process here will outline how I use the documentation to help me accomplish my goals.

We will create a basic example, which you can customize based on your needs.

In our code, let's look at the `Restaurant.js` file you can find it in our controller's folder in `api/restaurant/controllers`.

As you can see, we commented on the Strapi v3 custom code. We will have to update this custom controller manually.

```javascript
// path: ./src/api/restaurant/controllers/restaurant.js

const { createCoreController } = require("@strapi/strapi").factories;

module.exports = createCoreController("api::restaurant.restaurant");

// will come back to this later

// module.exports = {
//   find: async (ctx) => {
//     let restaurants;

//     if (ctx.query._q) {
//       restaurants = await strapi.api.restaurant.services.restaurant.search(ctx.query);
//     } else {
//       restaurants = await strapi.api.restaurant.services.restaurant.find(ctx.query);
//     }

//     restaurants = await Promise.all(
//       restaurants.map(async (restaurant) => {
//         restaurant.note = await strapi.api.review.services.review.average(restaurant.id);

//         return restaurant;
//       })
//     );

//     return restaurants;
//   },

//   findOne: async (ctx) => {
//     const { id } = ctx.params;
//     let restaurant = await strapi.api.restaurant.services.restaurant.findOne({ id });

//     if (!restaurant) {
//       return ctx.notFound();
//     }

//     restaurant.note = await strapi.api.review.services.review.average(restaurant.id);

//     let noteDetails = await strapi
//       .query('review')
//       .model.query(function (qb) {
//         qb.where('restaurant', '=', restaurant.id);
//         qb.groupBy('note');
//         qb.select('note');
//         qb.count();
//       })
//       .fetchAll()
//       .then((res) => res.toJSON());

//     restaurant.noteDetails = [];

//     for (let i = 5; i > 0; i--) {
//       let detail = noteDetails.find((detail) => {
//         return detail.note === i;
//       });

//       if (detail) {
//         detail = {
//           note: detail.note,
//           count: detail['count(*)'],
//         };
//       } else {
//         detail = {
//           note: i,
//           count: 0,
//         };
//       }

//       restaurant.noteDetails.push(detail);
//     }

//     return restaurant;
//   }
// };

```

In the [migration guide](https://docs.strapi.io/developer-docs/latest/update-migration-guides/migration-guides/v4/code/backend/controllers.html) under the controller's section, we can take a look and see the difference between v3 and v4.

🤓 v3/v4 comparison

In both Strapi v3 and v4, creating content-types automatically generates core API controllers.

Controllers are JavaScript files that contain a list of methods, called actions.

In Strapi v3, controllers export an object containing actions that are merged with the existing actions of core API controllers, allowing customization.

In Strapi v4, controllers export the result of a call to the [`createCoreController` factory function](https://docs.strapi.io/developer-docs/latest/development/backend-customization/controllers.html#implementation), with or without further customization.

We can see an example of how we can implement the controller without customizations.

```javascript
// path: ./src/api/<content-type-name>/controllers/<controller-name>.js

const { createCoreController } = require("@strapi/strapi").factories;

module.exports = createCoreController("api::api-name.content-type-name");
```

It is something that we have already done.

Followed by an example of a custom controller. Something that we are going to do now.

```javascript
// path: ./src/api/<content-type-name>/controllers/<controller-name>.js

const { createCoreController } = require("@strapi/strapi").factories;

module.exports = createCoreController(
  "api::api-name.content-type-name",
  ({ strapi }) => ({
    // wrap a core action, leaving core logic in place
    async find(ctx) {
      // some custom logic here
      ctx.query = { ...ctx.query, local: "en" };

      // calling the default core action with super
      const { data, meta } = await super.find(ctx);

      // some more custom logic
      meta.date = Date.now();

      return { data, meta };
    },
  })
);
```

You can also learn more about custom controllers in our Strapi v4 documentation [here](https://docs.strapi.io/developer-docs/latest/development/backend-customization/controllers.html#implementation).

Here they show three ways you can customize your controller.

```javascript
// path: ./src/api/restaurant/controllers/restaurant.js

const { createCoreController } = require("@strapi/strapi").factories;

module.exports = createCoreController(
  "api::restaurant.restaurant",
  ({ strapi }) => ({
    // Method 1: Creating an entirely custom action
    async exampleAction(ctx) {
      try {
        ctx.body = "ok";
      } catch (err) {
        ctx.body = err;
      }
    },

    // Method 2: Wrapping a core action (leaves core logic in place)
    async find(ctx) {
      // some custom logic here
      ctx.query = { ...ctx.query, local: "en" };

      // Calling the default core action
      const { data, meta } = await super.find(ctx);

      // some more custom logic
      meta.date = Date.now();

      return { data, meta };
    },

    // Method 3: Replacing a core action
    async findOne(ctx) {
      const { id } = ctx.params;
      const { query } = ctx;

      const entity = await strapi
        .service("api::restaurant.restaurant")
        .findOne(id, query);
      const sanitizedEntity = await this.sanitizeOutput(entity, ctx);

      return this.transformResponse(sanitizedEntity);
    },
  })
);
```

**_Method 1_**: Creating an entirely custom action
**_Method 2_**: Wrapping a core action (leaves core logic in place)
**_Method 3_** Replacing a core action

We will use the example from the migration guide above, but I just wanted to show you where you can find resources.

Let's customize our controller with the following code.

```javascript
// path: ./src/api/restaurant/controllers/restaurant.js

const { createCoreController } = require("@strapi/strapi").factories;

module.exports = createCoreController(
  "api::restaurant.restaurant",
  ({ strapi }) => ({
    async find(ctx) {
      // some custom logic here
      ctx.query = { ...ctx.query, local: "en" };

      // calling the default core action with super
      const { data, meta } = await super.find(ctx);

      // some more custom logic
      meta.date = Date.now();

      console.log(meta);
      return { data, meta };
    },
  })
);
```

In the code above, we are creating a simple custom controller that adds the current date to our `meta` object.

To test it out, first, we must enable the controller in our Strapi App.

You can do that by going into settings > roles > public > restaurant and checking the **_find_** endpoint.

![restaurantcontroller](https://user-images.githubusercontent.com/6153188/181597546-89909e0b-f311-4f11-901e-8e534fb3321c.gif)


Using Insomnia to make a GET request to our custom controller we can see that we are now returning our date field in our `meta` response object.

<img width="983" alt="customcontrollerresponse" src="https://user-images.githubusercontent.com/6153188/181597646-8c268487-f3a8-44c1-a668-289308bf39bd.png">

Using Insomnia, we make a GET request to our custom controller. We can see that we are now returning our date field in our `meta` response object.

Although this was a simple example, you now know how to implement custom controllers and where to find information in the docs.

If you wanted to set up a custom route, you can learn how to do that [here](https://docs.strapi.io/developer-docs/latest/update-migration-guides/migration-guides/v4/code/backend/routes.html).

Let's do a quick example to change our controller to have its own route, but first, let's rename it to `findCustomRoute.`

Since we renamed our controller, it no longer overrides our previous **_find_** controller. Instead, it will be a new controller for which we will create a custom route.

```javascript
// path: ./src/api/restaurant/controllers/restaurant.js

const { createCoreController } = require("@strapi/strapi").factories;

module.exports = createCoreController(
  "api::restaurant.restaurant",
  ({ strapi }) => ({
    async findCustomRoute(ctx) {
      // some custom logic here

      ctxquery = { ...ctx.query, local: "en" };

      // calling the default core action with super
      const { data, meta } = await super.find(ctx);

      // some more custom logic
      meta.date = Date.now();

      console.log(meta);
      return { data, meta };
    },
  })
);
```

Inside our `routes` folder, let's create a file called `custom-routes.js`. We are using the example from the docs [here](https://docs.strapi.io/developer-docs/latest/update-migration-guides/migration-guides/v4/code/backend/routes.html#migrating-custom-routers)

Let's add the following code to create our custom route.

```javascript
module.exports = {
  routes: [
    {
      method: "GET",
      path: "/restaurants/with-meta-date",
      handler: "restaurant.findCustomRoute",
    },
  ],
};
```

Before testing our custom route, once again, we have to enable the endpoint within Strapi.

![customroute](https://user-images.githubusercontent.com/6153188/181597734-3206969a-ed57-4ec0-8498-b4f5d2fff248.gif)

Let's test the new endpoint `/api/restaurants/with-meta-date` in Insomnia.

<img width="1331" alt="withcustomroute" src="https://user-images.githubusercontent.com/6153188/181597894-0541386b-c8dc-4077-835e-9bd6fe6988ab.png">

As we can see, our custom route and controller are working.

The above example should give you a good start on how to migrate custom routes and controllers.

Next, let's see how we can create a custom resolver with GraphQl.


### How to create a custom GraphQl resolver

Finally, let's look at how we can create custom GraphQl resolvers.  

But first, let's review what changed from Strapi v3 to v4.

🤓 v3/v4 comparison

In Strapi v3, GraphQL resolvers are either automatically bound to REST controllers (from the core API) or customized using the `./api/<api-name>/config/schema.graphql.js` files.

In Strapi v4, [GraphQL](https://docs.strapi.io/developer-docs/latest/plugins/graphql.html) dedicated core resolvers are automatically created for the basic CRUD operations for each API. Additional resolvers can be [customized programmatically](https://docs.strapi.io/developer-docs/latest/plugins/graphql.html#customization) using GraphQL’s extension service, accessible using `strapi.plugin(’graphql’).service(’extension’)`.

Migrating GraphQL resolvers to Strapi v4 requires:

-   moving the Strapi v3 logic, found in `./api/<api-name>/config/schema.graphql.js` files, to [the `register` method](https://docs.strapi.io/developer-docs/latest/setup-deployment-guides/configurations/optional/functions.html#register) found in the `./src/index.js` file of Strapi v4
-   and adapting the existing Strapi v3 code to take advantage of the GraphQL extension service introduced in Strapi v4, accessible through `strapi.plugin(’graphql’).service(’extension’)`.

You can read more in the documentation [here](https://docs.strapi.io/developer-docs/latest/update-migration-guides/migration-guides/v4/code/backend/graphql.html)

For our example, we will create a custom GraphQL resolver based on the above documentation.

Let's take a look inside the `src/index.js` file. We should see the following.

``` javascript
"use strict";

module.exports = {
  /**
   * An asynchronous register function that runs before
   * your application is initialized.
   *
   * This gives you an opportunity to extend code.
   */

  register(/*{ strapi }*/) {},

  /**
   * An asynchronous bootstrap function that runs before
   * your application gets started.
   *
   * This gives you an opportunity to set up your data model,
   * run jobs, or perform some special logic.
   */

  bootstrap(/*{ strapi }*/) {},
};
```

Let's add the following code inside the `register` function.

``` javascript

register({ strapi }) {
   const extensionService = strapi.service("plugin::graphql.extension");
},
```

This will allow us to extend the GraphQL plugin and add custom resolvers.

We will write our custom resolver in this file directly, but you don't have to as long as you reference it here.

Let's override our find resolver.  Go ahead and update the code inside the `register` function to the following.

``` javascript
  register({ strapi }) {
    const extensionService = strapi.service('plugin::graphql.extension');

    extensionService.use(({ strapi }) => ({
      typeDefs: `
          type Query {
              restaurants(
                  filters: RestaurantFiltersInput
                  pagination: PaginationArg = {}
                  sort: [String] = []
                  publicationState: PublicationState = LIVE
                  ): RestaurantEntityResponseCollection  
              }
          `,
      resolvers: {
        Query: {
          restaurants: {
            resolve: async (parent, args, context) => {
              const { toEntityResponseCollection } =
                strapi.service('plugin::graphql.format').returnTypes;
              const { transformArgs } = strapi.service('plugin::graphql.builders').utils;
              const contentType = strapi.contentTypes['api::restaurant.restaurant'];

              const transformedArgs = transformArgs(args, { contentType });

              const data = await strapi.entityService.findMany(
                'api::restaurant.restaurant',
                transformedArgs
              );

              const response = toEntityResponseCollection(data, {
                args: { transformedArgs, start: 0, limit: 10 },
                resourceUID: contentType.uid,
              });

              console.log('##################', response, '##################');
              return response;
            },
          },
        },
      },
    }));
  },

```

You can learn more about Strapi GraphQL [here](https://docs.strapi.io/developer-docs/latest/plugins/graphql.html)

Once you have made the following changes, restart the server with `yarn develop`, navigate to `http://localhost:1337/graphql`, and add the following query.

``` bash
    query {
      restaurants {
      	data {
          id
          attributes {
            name
            description
            address
          }
        }
      }
    }
```

We should probably add a restaurant entry first before we test it.

I added one restaurant entry so we can see the response when we call our custom resolver.

<img width="1506" alt="catentry" src="https://user-images.githubusercontent.com/6153188/181598059-ce720110-3152-4030-b315-47bc1cf41878.png">

Once you run the query, you will see the following result.  

<img width="1504" alt="graphqlresult" src="https://user-images.githubusercontent.com/6153188/181598185-db2ce6a5-dd9f-4b75-b20b-7e8015a730f8.png">

We will also see our console.log statement from our resolver on line 46 to confirm that we are hitting our custom GraphQl resolver.

``` bash
################## {
  nodes: [
    {
      id: 1,
      name: 'Cat Treats',
      description: 'Restaurant for cats',
      address: 'Cats World',
      website: 'bestcattreats.com',
      phone: '1234567890',
      price: 'one',
      district: 'first',
      publish_at: '2022-07-07T05:00:00.000Z',
      previous_: null,
      author_: null,
      createdAt: '2022-07-27T20:24:30.668Z',
      updatedAt: '2022-07-27T20:24:32.056Z',
      publishedAt: '2022-07-27T20:24:32.053Z'
    }
  ],
  info: {
    args: { transformedArgs: [Object], start: 0, limit: 10 },
    resourceUID: 'api::restaurant.restaurant'
  }
} ##################
```


## Conclusion
You now have the tools to feel more comfortable with the migration process.  

This is not the definitive guide, but it is a start. 

We are also making additional video resources to dive into more detail on specific topics around the migration process.

I hope this article and its accommodating stream on youtube have helped to demystify the migration process and where to find the resources.

Let's continue the discussion of [Discord](https://discord.com/invite/strapi). 

This was a long article, so thank you for your support and patience.  

If you have any issues or questions, please feel free to connect with me on Discord inside the v3 to v4 migration channel.

