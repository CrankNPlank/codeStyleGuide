# JavaScript

The following articles and repos heavily influenced the development of this 
style guide:
* [Meteor.js Code Style](https://guide.meteor.com/code-style.html)
* [AirBnB JavaScript Style Guide](https://github.com/airbnb/javascript)

## Indentation

* Two spaces (not tabs)

**Example**
```javascript
// Wrong - 4 spaces and/or tabs
if (condition) {
    firstStatement();
}

// Wrong - misleading indentation (without curly braces)
if (condition)
  firstStatement();
  secondStatement();

// Better
if (condition) {
  firstStatement();
}

secondStatement();
```

## Automatic Error Checking

Use tools to keep everyone honest, such as jsLint. For example, if you adopt a 
convention that you must always use `let` or `const` instead of `var`, you can 
now use a tool to ensure all of your variables are scoped the way you expect.

### Setting-up Sublime Text For Meteor.js ES6 (ES2015) and JSX Syntax & Linting

See [this article](http://info.meteor.com/blog/set-up-sublime-text-for-meteor-es6-es2015-and-jsx-syntax-and-linting) that 
sets-up Sublime Text for Meteor.js ES6 (ES2015) and JSX syntax and linting
* *NOTE:* Set-up linting rules globally using `~/.eslintrc`

*Other resources*
* [AirBnB ESLint Configuration](https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb)

### Setting-Up ESLint For Meteor.js App

1. Install the following npm packages:
   ```bash
   meteor npm install --save-dev babel-eslint eslint-config-airbnb eslint-plugin-import eslint-plugin-meteor eslint-plugin-react eslint-plugin-jsx-a11y eslint-import-resolver-meteor eslint
   ```

   *NOTE:* Can also set-up any extra rules you want to change, as well as 
   adding a lint npm command:
   ```json
   {
      ...
      "scripts": {
        "lint": "eslint .",
        "pretest": "npm run lint --silent"
      },
      "eslintConfig": {
        "parser": "babel-eslint",
        "parserOptions": {
          "allowImportExportEverywhere": true
        },
        "plugins": [
          "meteor"
        ],
        "extends": [
          "airbnb",
          "plugin:meteor/recommended"
        ],
        "settings": {
          "import/resolver": "meteor"
        },
        "rules": {}
      }
   }
   ```
   To run the linter (in a Meteor.js app), you can now simply type:
   ```bash
   meteor npm run lint
   ```

