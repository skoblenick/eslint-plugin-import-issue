# eslint-plugin-import Issue

## Setup

  1. Install dependencies:

    ```
    npm install
    ```

  1.  Run lint script with: `nx lint demo`

## Issue

The following error occurs when applying eslint-plugin-import rules globally
for all files.

```
> nx run demo:lint

Linting "demo"...
Cannot read property 'forEach' of undefined
Occurred while linting <omitted-path>/eslint-plugin-import-issue/apps/demo/src/app/app.component.html:1

———————————————————————————————————————————————

>  NX   ERROR  Running target "demo:lint" failed

  Failed tasks:

  - demo:lint

  Hint: run the command with --verbose for more details.
```

Running in verbose mode:

```
> nx run demo:lint --verbose

Linting "demo"...
Cannot read property 'forEach' of undefined
Occurred while linting <omiited-path>/eslint-plugin-import-issue/apps/demo/src/app/app.component.html:1
TypeError: Cannot read property 'forEach' of undefined
Occurred while linting <omiited-path>/eslint-plugin-import-issue/apps/demo/src/app/app.component.html:1
    at Program (<omiited-path>/eslint-plugin-import-issue/node_modules/eslint-plugin-import/lib/rules/first.js:45:18)
    at <omiited-path>/eslint-plugin-import-issue/node_modules/eslint/lib/linter/safe-emitter.js:45:58
    at Array.forEach (<anonymous>)
    at Object.emit (<omiited-path>/eslint-plugin-import-issue/node_modules/eslint/lib/linter/safe-emitter.js:45:38)
    at NodeEventGenerator.applySelector (<omiited-path>/eslint-plugin-import-issue/node_modules/eslint/lib/linter/node-event-generator.js:256:26)
    at NodeEventGenerator.applySelectors (<omiited-path>/eslint-plugin-import-issue/node_modules/eslint/lib/linter/node-event-generator.js:285:22)
    at NodeEventGenerator.enterNode (<omiited-path>/eslint-plugin-import-issue/node_modules/eslint/lib/linter/node-event-generator.js:299:14)
    at CodePathAnalyzer.enterNode (<omiited-path>/eslint-plugin-import-issue/node_modules/eslint/lib/linter/code-path-analysis/code-path-analyzer.js:711:23)
    at <omiited-path>/eslint-plugin-import-issue/node_modules/eslint/lib/linter/linter.js:954:32
    at Array.forEach (<anonymous>)

———————————————————————————————————————————————

>  NX   ERROR  Running target "demo:lint" failed

  Failed tasks:

  - demo:lint

  Hint: run the command with --verbose for more details.
```

Digging the the source it seems that the forEach block isn't defended against possible undefined values due to the argument `n` not having a propery of `body`. See:

[/eslint-plugin-import/blob/main/src/rules/first.js#L32-45](https://github.com/import-js/eslint-plugin-import/blob/main/src/rules/first.js#L32-45)

When the rule executes on an HTML file the payload of `n` is:

```
n <ref *1> {
  type: 'Program',
  comments: [],
  tokens: [],
  range: [ 0, 5280 ],
  loc: { start: { line: 1, column: 0 }, end: { line: 139, column: 0 } },
  templateNodes: [
    Element {
      name: 'header',
      attributes: [Array],
      inputs: [],
      outputs: [],
      children: [Array],
      references: [],
      sourceSpan: [ParseSourceSpan],
      startSourceSpan: [ParseSourceSpan],
      endSourceSpan: [ParseSourceSpan],
      i18n: undefined,
      type: 'Element',
      parent: [Circular *1]
    },
    Text {
      value: '\n',
      sourceSpan: [ParseSourceSpan],
      type: 'Text',
      parent: [Circular *1]
    },
    Element {
      name: 'main',
      attributes: [],
      inputs: [],
      outputs: [],
      children: [Array],
      references: [],
      sourceSpan: [ParseSourceSpan],
      startSourceSpan: [ParseSourceSpan],
      endSourceSpan: [ParseSourceSpan],
      i18n: undefined,
      type: 'Element',
      parent: [Circular *1]
    },
    Text {
      value: '\n',
      sourceSpan: [ParseSourceSpan],
      type: 'Text',
      parent: [Circular *1]
    }
  ],
  value: '<ommited html blob>',
  parent: null
}
```


## Reference

Steps to recreate this repo and recreate the issue:

1.  Create an nx workspace

    ```shell
    npx create-nx-workspace@latest eslint-plugin-import-issue
    ```

1.  Navigate into the new workspace:

    ```
    cd eslint-plugin-import-issue
    ```

1.  Run the pre-configured lint script:

    ```
    nx lint demo
    ```

1.  Modify the `.eslintrc.json` and rename it to `.eslintrc.yml`:

    ```yml
    ---
    root: true

    ignorePatterns:
      - "!.*" # do not ignore hidden files
      - ".git/**"
      - "**/dist/**"
      - "**/node_modules/**"

    plugins:
      - "@nrwl/nx"
      - import

    rules:
      import/first: error
      import/order:
        - error
        - groups:
            - internal
            - external
            - builtin
            - object
            - parent
            - sibling
            - index
          newlines-between: always
          alphabetize:
            order: asc

    overrides:
      -
        files: ["*.ts", "*.js"]
        rules:
          "@nrwl/nx/enforce-module-boundaries":
              - error
              -
                allow: []
                depConstraints:
                  sourceTag: "*"
                  onlyDependOnLibsWithTags: ["*"]

      -
        files: ["*.ts"]
        extends:
          - "plugin:@nrwl/nx/typescript"
        rules: {}
      -
        files: ["*.js"]
        extends: ["plugin:@nrwl/nx/javascript"]
        rules: {}
    ```

1.  Update the demo and demo-e2e applications `.eslintrc.json` files to extend
    from the updated root `.eslintrc.yml`

    ```json
    // apps/demo/.eslintrc.json
    {
      "extends": ["../../.eslintrc.yml"],
      "ignorePatterns": ["!**/*"],
      "overrides": [


    // apps/demo-e2e/.eslintrc.json
    {
      "extends": ["plugin:cypress/recommended", "../../.eslintrc.yml"],
      "ignorePatterns": ["!**/*"],
      "overrides": [
    ```

1.  Run the lint script on demo app, which results in the error:

    ```shell
    > nx run demo:lint

    Linting "demo"...
    Cannot read property 'forEach' of undefined
    Occurred while linting /Users/rskoblenick/repos/com.github/skoblenick/eslint-plugin-import-issue/apps/demo/src/app/app.component.html:1

    ———————————————————————————————————————————————

    >  NX   ERROR  Running target "demo:lint" failed

      Failed tasks:

      - demo:lint

      Hint: run the command with --verbose for more details.
    ```
