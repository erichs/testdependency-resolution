## Testing dependency resolution (NPM/Yarn) vs reporting with cdxgen

### Yarn

Repo was initialized and the following packaged added with yarn:
`yarn add lodash@4.17.21 @lifeomic/dynamodb-dataloader@1.0.2 aws-sdk@2.883.0`

`npm ls lodash` output shows:
```
├─┬ @lifeomic/dynamodb-dataloader@1.0.2
│ └── lodash@4.17.21  deduped
└── lodash@4.17.21
```

`yarn why lodash` shows:

```
=> Found "lodash@4.17.21"
info Has been hoisted to "lodash"
info Reasons this module exists
   - Specified in "dependencies"
   - Hoisted from "@lifeomic#dynamodb-dataloader#lodash"
```



