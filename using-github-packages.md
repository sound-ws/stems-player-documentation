# Using github packages

## Quickstart configuring npm for use with github packages

After you have been granted read access to the private github repository,

1. Create a personal access token
2. Then make sure your ~/.npmrc looks something like this

```text
//npm.pkg.github.com/:_authToken=$TOKEN
@sound-ws:registry=https://npm.pkg.github.com/
```

This tells npm to get any @sound-ws packages from the github packages npm repository, and that is should authenticate using the TOKEN.

For further information please consult the [Github documentation](https://help.github.com/en/packages/using-github-packages-with-your-projects-ecosystem/configuring-npm-for-use-with-github-packages)
