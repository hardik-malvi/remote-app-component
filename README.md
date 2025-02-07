# Guide to Integrating Angular Applications with Module Federation

This guide walks you through integrating two Angular applications using Webpack Module Federation to implement a micro-frontend architecture. By following these steps, you can:

- Dynamically load components from one application (remote) into another application (host) at runtime.
- Build modular, scalable, and flexible Angular applications.

---

## Overview

### What You Will Do:
1. Create two Angular apps:
   - **Host Application:** Loads and displays content from another app.
   - **Remote Application:** Exposes components to be used by the host app.
2. Use **Webpack Module Federation** to connect these apps for runtime sharing without pre-bundling.

---

## Prerequisites

- **Angular Version:** 16.0.0
- **Node Version:** Compatible with Angular 16 (e.g., Node.js 18.x or later)

---

## Steps to Set Up the Project

### Step 1: Create Two Angular Projects

1. Create the **Host Application**:

   ```bash
   ng new host-app --routing --style=css
   ```

2. Create the **Remote Application**:

   ```bash
   ng new remote-app --routing --style=css
   ```

### Step 2: Add Module Federation to Both Applications

1. Install the Module Federation package in both apps:

   - For the **Host Application**:
     ```bash
     cd host-app
     ng add @angular-architects/module-federation
     ```

   - For the **Remote Application**:
     ```bash
     cd remote-app
     ng add @angular-architects/module-federation
     ```

### Step 3: Set Up the Remote Application

1. Expose a component in the remote app:
   Update the `webpack.config.js` file:

   ```javascript
   const ModuleFederationPlugin = require("webpack/lib/container/ModuleFederationPlugin");
   const mf = require("@angular-architects/module-federation/webpack");
   const path = require("path");

   module.exports = {
     output: {
       uniqueName: "remoteApp",
       publicPath: "auto"
     },
     plugins: [
       new ModuleFederationPlugin({
           name: "remoteApp",
           filename: "remoteEntry.js",
           exposes: {
               './Component': './src/app/app.component.ts',
           },
           shared: mf.share({
             "@angular/core": { singleton: true, strictVersion: true },
             "@angular/common": { singleton: true, strictVersion: true },
             "@angular/router": { singleton: true, strictVersion: true },
           })
       })
     ]
   };
   ```

2. Build the remote app:

   ```bash
   ng build remote-app
   ```

### Step 4: Set Up the Host Application

1. Configure the `webpack.config.js` file to reference the remote app:

   ```javascript
   const ModuleFederationPlugin = require("webpack/lib/container/ModuleFederationPlugin");
   const mf = require("@angular-architects/module-federation/webpack");
   const path = require("path");

   module.exports = {
     output: {
       uniqueName: "hostApp",
       publicPath: "auto"
     },
     plugins: [
       new ModuleFederationPlugin({
           remotes: {
             "remoteApp": "http://localhost:4201/remoteEntry.js",
           },
           shared: mf.share({
             "@angular/core": { singleton: true, strictVersion: true },
             "@angular/common": { singleton: true, strictVersion: true },
             "@angular/router": { singleton: true, strictVersion: true },
           })
       })
     ]
   };
   ```

2. Declare the remote component in TypeScript:
   Create a `declarations.d.ts` file in the `src` folder:

   ```typescript
   declare module 'remoteApp/Component';
   ```

3. Dynamically load the remote component in the host app:

   Update the `app.component.ts` file:

   ```typescript
   import { Component, ViewChild, ViewContainerRef, AfterViewInit } from '@angular/core';
   import { AppComponent } from 'remoteApp/Component';

   @Component({
     selector: 'app-root',
     template: `<ng-template #dynamicContainer></ng-template>`,
   })
   export class RootAppComponent implements AfterViewInit {
     @ViewChild('dynamicContainer', { read: ViewContainerRef }) container!: ViewContainerRef;

     ngAfterViewInit() {
       const factory = this.container.createComponent(AppComponent);
     }
   }
   ```

### Step 5: Run the Applications

1. Start the **Remote Application**:

   ```bash
   cd remote-app
   ng serve --port 4201
   ```

2. Start the **Host Application**:

   ```bash
   cd host-app
   ng serve --port 4200
   ```

---

## Key Takeaways

- **Host Application:** The main app that consumes remote modules.
- **Remote Application:** The app that exposes components/modules for the host app.
- **Webpack Module Federation:** Enables sharing code between apps at runtime.

This approach helps you implement a **micro-frontend architecture** where different teams can develop and deploy parts of an app independently, loaded dynamically at runtime.

---

## License

This project is licensed under the MIT License. See the LICENSE file for more details.
