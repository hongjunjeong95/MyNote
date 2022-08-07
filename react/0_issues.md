# Issues

## Failed to load config "react-app" to extend from

1. npx create-react-app appy --template typescript
2. cd app
3. npm i eslint@latest
4. remove folder node_modules
5. remove yarn.lock package.lock
6. remove eslint config entry in package.json
7. add .eslintrc.js

## Parsing error: Cannot read file '.../tsconfig.json'.eslint

Add the code in `.eslintrc.js`

```javascript
module.exports = {
  // ...
  parserOptions: {
    project: "tsconfig.json",
    tsconfigRootDir: __dirname,
    sourceType: "module",
  },
  // ...
}
```

