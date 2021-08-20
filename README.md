# Yarn hoisting issue
Minimal reproducible example

## Setup
- Yarn v1.22.5
- mono-repo with workspaces

Two packages from a mono-repo depend on two different versions of Webpack.
In addition to that they also use a Webpack plugin of the same version.
This plugin is compatible with both Webpack versions.

Because Webpack doesn't play well with hoisting, no-hoist is forced for it
in both packages.

Note that in reality both Webpack and the plugin are transient dependencies
(of react-script and Gatsby), therefore the requested versions can't be unified.

`packages/a/package.json`:
```json
  "workspaces": {
    "nohoist": [
      "webpack",
      "webpack/**"
    ]
  },
  "dependencies": {
    "webpack": "4.44.1",
    "@pmmmwh/react-refresh-webpack-plugin": "0.4.3"
  }
```

`packages/b/package.json`:
```json
  "workspaces": {
    "nohoist": [
      "webpack",
      "webpack/**"
    ]
  },
  "dependencies": {
    "webpack": "5.51.1",
    "@pmmmwh/react-refresh-webpack-plugin": "0.4.3"
  }
```

`@pmmmwh/react-refresh-webpack-plugin/package.json`:
```json
  "dependencies": {
    // no webpack at all
  },  
  "devDependencies": {
    // ..    
    "webpack": "^4.44.2",
    // ..
  },
  "peerDependencies": {
    // ..
    "webpack": ">=4.43.0 <6.0.0",
    // ..
  }
```

## Problem
After installing everything via `yarn`
- the plugin (`@pmmmwh/react-refresh-webpack-plugin`) resides in `node_modules` of both packages 
- nested `node_modules` of the plugin
    - is nearly empty in case of the package `a` - expected
    - contains many packages including a copy of `webpack` - unexpected
- the copy of `webpack` in the plugin's `node_modules` of package `b` shadows
  the `node_modules/webpack` from the package `b` directly !!!
- the version of `webpack` in the plugin's `node_modules` of package `b` matches
  requested version of package `a` not `b` !!!
  
As you can imagine, this causes a tremendous amount of issues...
