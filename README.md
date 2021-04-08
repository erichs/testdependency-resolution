## Testing dependency resolution (NPM/Yarn) vs reporting with cdxgen

### Using Yarn

Repo was initialized and several public NPM packages added with `yarn add`:
```
yarn add \
  lodash@^4.17.21 \
  @lifeomic/dynamodb-dataloader@1.0.2 \
  aws-sdk@2.883.0 \
  @jupiterone/typescript-tools@13.0.2 \
  @lifeomic/alpha@1.3.1 \
  @lifeomic/lambda-tools@13.0.1 \
  node-cache@4.2.1 \
  np@2.18.3 \
  request-promise-native@1.0.5
```

One privately scoped repo in the public NPM registry was added with `yarn add`:
```
 yarn add @jupiterone/koa@11.0.7
```

NOTE: the presence of this private repo seems to cause `cdxgen` issues...

At this point, `npm ls lodash` output shows:
```
├─┬ @jupiterone/koa@11.0.7
│ ├─┬ @jupiterone/platform-headers@2.3.2
│ │ └── lodash@4.17.21  deduped
│ └── lodash@4.17.21  deduped
├─┬ @jupiterone/typescript-tools@13.0.2
│ ├─┬ @babel/cli@7.13.14
│ │ └── lodash@4.17.21  deduped
│ ├─┬ @babel/core@7.13.15
│ │ └─┬ @babel/types@7.13.14
│ │   └── lodash@4.17.21  deduped
│ ├─┬ @jupiterone/eslint-config@1.0.2
│ │ ├─┬ @typescript-eslint/eslint-plugin@4.21.0
│ │ │ └── lodash@4.17.21  deduped
│ │ └─┬ eslint-plugin-jest@23.20.0
│ │   └─┬ @typescript-eslint/experimental-utils@2.34.0
│ │     └─┬ @typescript-eslint/typescript-estree@2.34.0
│ │       └── lodash@4.17.21  deduped
│ ├─┬ eslint@7.23.0
│ │ └── lodash@4.17.21  deduped
│ └─┬ jest@26.6.3
│   └─┬ @jest/core@26.6.3
│     └─┬ jest-config@26.6.3
│       └─┬ jest-environment-jsdom@26.6.2
│         └─┬ jsdom@16.5.2
│           ├─┬ request-promise-native@1.0.9
│           │ └─┬ request-promise-core@1.1.4
│           │   └── lodash@4.17.21  deduped
│           └─┬ whatwg-url@8.5.0
│             └── lodash@4.17.21  deduped
├─┬ @lifeomic/alpha@1.3.1
│ └── lodash@4.17.21  deduped
├─┬ @lifeomic/dynamodb-dataloader@1.0.2
│ └── lodash@4.17.21  deduped
├─┬ @lifeomic/lambda-tools@13.0.1
│ ├─┬ archiver@3.1.1
│ │ └─┬ async@2.6.3
│ │   └── lodash@4.17.21  deduped
│ ├─┬ ava@3.15.0
│ │ ├─┬ concordance@5.0.4
│ │ │ └── lodash@4.17.21  deduped
│ │ └── lodash@4.17.21  deduped
│ ├── lodash@4.17.21  deduped
│ └─┬ webpack-babel-env-deps@1.6.4
│   └── lodash@4.17.21  deduped
├── lodash@4.17.21
├─┬ node-cache@4.2.1
│ └── lodash@4.17.21  deduped
├─┬ np@2.18.3
│ └─┬ inquirer@3.3.0
│   └── lodash@4.17.21  deduped
└─┬ request-promise-native@1.0.5
  └─┬ request-promise-core@1.1.1
    └── lodash@4.17.21  deduped
```

NOTE the 'deduped' entries

and `yarn why lodash` shows:

```
=> Found "lodash@4.17.21"
info Has been hoisted to "lodash"
info Reasons this module exists
   - Specified in "dependencies"
   - Hoisted from "@jupiterone#koa#lodash"
   - Hoisted from "@lifeomic#dynamodb-dataloader#lodash"
   - Hoisted from "node-cache#lodash"
   - Hoisted from "@lifeomic#alpha#lodash"
   - Hoisted from "@lifeomic#lambda-tools#lodash"
   - Hoisted from "request-promise-native#request-promise-core#lodash"
   - Hoisted from "@jupiterone#koa#@jupiterone#platform-headers#lodash"
   - Hoisted from "@jupiterone#typescript-tools#@babel#cli#lodash"
   - Hoisted from "@lifeomic#lambda-tools#ava#lodash"
   - Hoisted from "@jupiterone#typescript-tools#eslint#lodash"
   - Hoisted from "@lifeomic#lambda-tools#webpack-babel-env-deps#lodash"
   - Hoisted from "np#inquirer#lodash"
   - Hoisted from "@lifeomic#lambda-tools#archiver#async#lodash"
   - Hoisted from "@jupiterone#typescript-tools#@jupiterone#eslint-config#@typescript-eslint#eslint-plugin#lodash"
   - Hoisted from "@lifeomic#lambda-tools#ava#concordance#lodash"
   - Hoisted from "@jupiterone#typescript-tools#@babel#core#@babel#types#lodash"
   - Hoisted from "@jupiterone#typescript-tools#@jupiterone#eslint-config#eslint-plugin-jest#@typescript-eslint#experimental-utils#@typescript-eslint#typescript-estree#lodash"
   - Hoisted from "@jupiterone#typescript-tools#jest#@jest#core#jest-config#jest-environment-jsdom#jsdom#whatwg-url#lodash"
   - Hoisted from "@jupiterone#typescript-tools#jest#@jest#core#jest-config#jest-environment-jsdom#jsdom#request-promise-native#request-promise-core#lodash"
```

This hosting/deduping is confirmed by:

```
❯ find . -name lodash
./node_modules/@types/lodash
./node_modules/lodash

❯ jq .version node_modules/lodash/package.json
"4.17.21"
```

There is one version of lodash installed, and it is version 4.17.21, which
satisfies all recursive version constraints.

#### Generate BOM with cdxgen -r

```
❯ cdxgen --version
2.2.19

❯ cdxgen -r -o bom-yarn.json
BOM file written to bom-yarn.json
```

The lodash results in this file are wrong:

```
❯ grep lodash bom-yarn.json
      "name": "lodash",
      "purl": "pkg:npm/lodash@4.17.4",
      "bom-ref": "pkg:npm/lodash@4.17.4"
```

### Using NPM

```
❯ rm -rf node_modules yarn.lock && npm install

❯ cdxgen -r -o bom-npm.json
BOM file written to bom-npm.json

❯ grep lodash bom-npm.json
      "name": "lodash",
      "purl": "pkg:npm/lodash@4.17.4",
      "bom-ref": "pkg:npm/lodash@4.17.4"
      "name": "lodash",
      "purl": "pkg:npm/lodash@4.17.21",
      "bom-ref": "pkg:npm/lodash@4.17.21"
...

❯ npm ls lodash
├─┬ @jupiterone/koa@11.0.7
│ ├─┬ @jupiterone/platform-headers@2.3.2
│ │ └── lodash@4.17.21  deduped
│ └── lodash@4.17.21  deduped
├─┬ @jupiterone/typescript-tools@13.0.2
│ ├─┬ @babel/cli@7.13.14
│ │ └── lodash@4.17.21  deduped
│ ├─┬ @babel/core@7.13.15
│ │ └─┬ @babel/types@7.13.14
│ │   └── lodash@4.17.21  deduped
│ ├─┬ @jupiterone/eslint-config@1.0.2
│ │ ├─┬ @typescript-eslint/eslint-plugin@4.21.0
│ │ │ └── lodash@4.17.21  deduped
│ │ └─┬ eslint-plugin-jest@23.20.0
│ │   └─┬ @typescript-eslint/experimental-utils@2.34.0
│ │     └─┬ @typescript-eslint/typescript-estree@2.34.0
│ │       └── lodash@4.17.21  deduped
│ ├─┬ eslint@7.23.0
│ │ └── lodash@4.17.21  deduped
│ └─┬ jest@26.6.3
│   └─┬ @jest/core@26.6.3
│     └─┬ jest-config@26.6.3
│       └─┬ jest-environment-jsdom@26.6.2
│         └─┬ jsdom@16.5.2
│           ├─┬ request-promise-native@1.0.9
│           │ └─┬ request-promise-core@1.1.4
│           │   └── lodash@4.17.21  deduped
│           └─┬ whatwg-url@8.5.0
│             └── lodash@4.17.21  deduped
├─┬ @lifeomic/alpha@1.3.1
│ └── lodash@4.17.21  deduped
├─┬ @lifeomic/dynamodb-dataloader@1.0.2
│ └── lodash@4.17.21  deduped
├─┬ @lifeomic/lambda-tools@13.0.1
│ ├─┬ archiver@3.1.1
│ │ └─┬ async@2.6.3
│ │   └── lodash@4.17.21  deduped
│ ├─┬ ava@3.15.0
│ │ ├─┬ concordance@5.0.4
│ │ │ └── lodash@4.17.21  deduped
│ │ └── lodash@4.17.21  deduped
│ ├── lodash@4.17.21  deduped
│ └─┬ webpack-babel-env-deps@1.6.4
│   └── lodash@4.17.21  deduped
├── lodash@4.17.21
├─┬ node-cache@4.2.1
│ └── lodash@4.17.21  deduped
├─┬ np@2.18.3
│ └─┬ inquirer@3.3.0
│   └── lodash@4.17.21  deduped
└─┬ request-promise-native@1.0.5
  └─┬ request-promise-core@1.1.1
    └── lodash@4.17.21  deduped

❯ find . -name lodash
./node_modules/@types/lodash
./node_modules/lodash

❯ jq .version node_modules/lodash/package.json
"4.17.21"
```

### Observations

Installation with Yarn produces an incorrect BOM for lodash
Installation with NPM produces slightly different (but still incorrect) BOM for
lodash

Removing the private `@jupiterone/koa` repo, and again installing with NPM, then
Yarn, and generating the BOM files `bom-{yarn,npm}-sans-private-pkg.json`
removes this false positive:

```
❯ grep lodash bom-*-sans-private-pkg.json | grep 4.17
bom-npm-sans-private-pkg.json:      "purl": "pkg:npm/lodash@4.17.21",
bom-npm-sans-private-pkg.json:      "bom-ref": "pkg:npm/lodash@4.17.21"
bom-yarn-sans-private-pkg.json:      "purl": "pkg:npm/lodash@4.17.21",
bom-yarn-sans-private-pkg.json:      "bom-ref": "pkg:npm/lodash@4.17.21"
```

This BOM discrepancy is problematic when it comes to vulnerable 3rd party
package analysis, as it results in false-positives.
