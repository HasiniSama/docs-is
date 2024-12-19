---
template: templates/quick-start.html
heading: JavaScript Quickstart
description: Welcome to the JavaScript Quickstart guide! In this document, you will learn to build a single-page JavaScript app, add user login and display user profile information using Asgardeo.
what_you_will_learn:
  - Create new single-page JavaScript app
  - Install Asgardeo JavaScript SDK
  - Add user login and logout
  - Display user profile information
prerequisites:
  - About 15 minutes
  - <a href="{{ base_path }}/get-started/create-asgardeo-account/">Asgardeo account</a>
  - Install <a href="https://nodejs.org/en/download/package-manager" target="_blank" rel="noopener noreferrer">Node.js</a> on your system.
  - Make sure you have a JavaScript package manager like <code>npm</code>, <code>yarn</code>, or <code>pnpm</code>.
  - A favorite text editor or IDE
source_code: <a href="https://github.com/dasuni-30/asgardeo-javascript-vite-app/tree/master" target="_blank" class="github-icon">JavaScript App Sample</a>
whats_next:
  # - Try out <a href="{{ base_path }}/complete-guides/nodejs/introduction/" target="_blank">{{ product_name }} complete Node.js guide</a>
  # - Try out {{product_name}} user onboarding complete guide for JavaScript
  # - Read security best practices for JavaScript app guide
---
## Configure an Application in {{ product_name }}

- Sign into {{ product_name }} console and navigate to **Applications > New Application**.
- Select **Single Page Application** and complete the wizard popup by providing a suitable name and an authorized redirect URL. 

!!! Example
    **name:** asgardeo-javascript
    
    **Authorized redirect URL:** http://localhost:5173

Note down the following values from the **Protocol** tab of the registered application. You will need them to configure  Asgardeo JavaScript SDK.

- **`client-id`** from the **Protocol** tab. 
- **The name of your Asgardeo organization**


!!! Info

    The authorized redirect URL determines where Asgardeo should send users after they successfully log in. Typically, this will be the web address where your app is hosted. For this guide, we'll use`http://localhost:5173`, as the sample app will be accessible at this URL.

## Create a JavaScript app using Vite

Create (a.k.a scaffold) your new JavaScript app using Vite.

=== "npm"

    ```bash
    npm create vite@latest asgardeo-javascript -- --template vanilla

    cd asgardeo-javascript

    npm install

    npm run dev
    ```

=== "yarn"

    ```bash
    yarn create vite@latest asgardeo-javascript -- --template vanilla

    cd asgardeo-javascript

    yarn install

    yarn dev
    ```

=== "pnpm"

    ```bash
    pnpm create vite@latest asgardeo-javascript -- --template vanilla

    cd asgardeo-javascript

    pnpm install

    pnpm dev
    ```

## Install @asgardeo/auth-spa

Asgardeo JavaScript SDK provides all the components and hooks you need to integrate Asgardeo into your app. To get started, simply add the Asgardeo JavaScript SDK to the project. Make sure to stop the dev server started in the previous step. 

=== "npm"

    ```bash
    npm install @asgardeo/auth-spa

    ```

=== "yarn"

    ```bash
    yarn add @asgardeo/auth-spa
    ```

=== "pnpm"

    ```bash
    pnpm add @asgardeo/auth-spa
    ```

## Initialize @asgardeo/auth-spa

Replace the existing code of the `main.jsx` file with the following code to initialize `AuthSPAClient`. 

!!! Important

    Replace below placeholders with your registered organization name in Asgardeo and the generated`client-id` from the app you registered in Asgardeo.

    - `<your-app-client-id>`
    - `https://api.asgardeo.io/t/<your-organization-name>`



```javascript title="src/main.js" 

import { AsgardeoSPAClient } from "@asgardeo/auth-spa";

const auth = AsgardeoSPAClient.getInstance();

await auth.initialize({
  signInRedirectURL: "http://localhost:5173",
  signOutRedirectURL: "http://localhost:5173",
  clientID: "<your-app-client-id>",
  baseUrl: "https://api.asgardeo.io/t/<your-organization-name>",
  scope: ["openid", "profile"]
});

```

## Add login and logout link to your app

The `AsgardeoSPAClient` provides `signIn` and `signOut` methods and access to the authentication state. Add the following code to the `main.js` file to conditionally render a login or logout button based on whether the user is authenticated.

```javascript title="src/main.js" hl_lines="1 14-30 32-41 43-49"

import { AsgardeoSPAClient, SPAUtils } from "@asgardeo/auth-spa";

const auth = AsgardeoSPAClient.getInstance();

await auth.initialize({
  signInRedirectURL: "http://localhost:5173",
  signOutRedirectURL: "http://localhost:5173",
  clientID: "<your-app-client-id>",
  baseUrl: "https://api.asgardeo.io/t/sagaraorg",
  scope: ["openid", "profile"]
});

(async () => {
  let user = undefined;

  if (SPAUtils.hasAuthSearchParamsInURL()) {
    user = await auth.signIn({ callOnlyOnRedirect: true });
  } else {
    user = await auth.trySignInSilently();
  }

  if (user) {
    document.getElementById("authenticated-view").style.display = "block";
    document.getElementById("unauthenticated-view").style.display = "none";
  } else {
    document.getElementById("authenticated-view").style.display = "none";
    document.getElementById("unauthenticated-view").style.display = "block";
  }
})();

document.querySelector('#app').innerHTML = `
  <div>
    <div id="authenticated-view" style="display: none">
      <button id="logout">Log Out</button>
    </div>
    <div id="unauthenticated-view" style="display: none">
      <button id="login">Log In</button>
    </div>
  </div>
`;

document.getElementById("login").addEventListener("click", async () => {
  auth.signIn();
});

document.getElementById("logout").addEventListener("click", async () => {
  auth.signOut();
});

```

Visit your app's homepage at [http://localhost:5173](http://localhost:5173){:target="_blank"}.

!!! Important

    You need to create a test user in {{ product_name }} by following this [guide]({{ base_path }}/guides/users/manage-users/#onboard-single-user){:target="_blank"} to tryout login and logout features.

## Display logged in user details

Modify the code as below to see logged in user details.

```javascript title="src/main.js" hl_lines="26 36"

import { AsgardeoSPAClient, SPAUtils } from "@asgardeo/auth-spa";

const auth = AsgardeoSPAClient.getInstance();

await auth.initialize({
  signInRedirectURL: "http://localhost:5173",
  signOutRedirectURL: "http://localhost:5173",
  clientID: "<your-app-client-id>",
  baseUrl: "https://api.asgardeo.io/t/sagaraorg",
  scope: ["openid", "profile"]
});


(async () => {
  let user = undefined;

  if (SPAUtils.hasAuthSearchParamsInURL()) {
    user = await auth.signIn({ callOnlyOnRedirect: true });
  } else {
    user = await auth.trySignInSilently();
  }

  if (user) {
    document.getElementById("authenticated-view").style.display = "block";
    document.getElementById("unauthenticated-view").style.display = "none";
    document.getElementById("username").innerHTML = "Welcome " + user.username;
  } else {
    document.getElementById("authenticated-view").style.display = "none";
    document.getElementById("unauthenticated-view").style.display = "block";
  }
})();

document.querySelector('#app').innerHTML = `
  <div>
    <div id="authenticated-view" style="display: none">
      <p id="username"></p>
      <button id="logout">Log Out</button>
    </div>
    <div id="unauthenticated-view" style="display: none">
      <button id="login">Log In</button>
    </div>
  </div>
`;

document.getElementById("login").addEventListener("click", async () => {
  auth.signIn();
});

document.getElementById("logout").addEventListener("click", async () => {
  auth.signOut();
});


```