# Microfrontend Multirepo
This application is having multi-repo structure to demo how to build microfrontend apps using module federation.

## Creation of applications using a specific angular/cli version

```
npx @angular/cli@16.2.6 new host-app --routing --style=scss

```

```
npx @angular/cli@16.2.6 new remote-app --routing --style=scss

```

## Adding Module Federation package from Webpack 5 which is available in angular v15. To explicitly allow angular to use the custom builder namely '@angular-architects/module-federation' to enable this Module Federation package from Webpack 5 we need to run the below commands for both the applications i.e. host-app and remote-app

```
ng add @angular-architects/module-federation --project host-app --port 4200

```

```
ng add @angular-architects/module-federation --project remote-app --port 4300

```

## Creating a module and component inside remote-app named counter and added default route

```
ng g m counter --routing

```

```
ng g c counter

```

## Exposing remote application 'remote-app' to host application 'host-app'
### In remote-app, webpack.config.js modify 'exposes' attribute to expose 'CounterModule' as defined in host-app routes as shown below:

```
exposes: {
    './CounterModule': './src/app/counter/counter.module.ts',
},

```

Also modify output 'scriptType' to 'text/javascript' to allow output to be inferred as javascript output

```
module.exports = {
  output: {
    uniqueName: "remoteApp",
    publicPath: "auto",
    scriptType: "text/javascript",
  },
  ...
}
```

### In host-app, webpack.config.js looks like below:

```
plugins: [
    new ModuleFederationPlugin({       
        // For hosts (please adjust)
        remotes: {
            "remoteApp": "remoteApp@http://localhost:4300/remoteEntry.js",
        },
        ...
    }),
    ...
]

```

## Modify host-app routes to load remote-app's CounterModule in route path 'counterApp' as shown below:

```
const routes: Routes = [
  {
    path: '',
    redirectTo: 'counterApp',
    pathMatch: 'full',
  },
  {
    path: 'counterApp',
    loadChildren: () => {
      return loadRemoteModule({
        remoteEntry: 'http://localhost:4300/remoteEntry.js',
        remoteName: 'remoteApp',
        exposedModule: './CounterModule'
      }).then(m => m.CounterModule).catch(err => console.log(err));
    }
  }
];

```
