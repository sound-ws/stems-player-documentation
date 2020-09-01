# Using github packages

## Create a token

Create a github developer token, paying attention to giving the correct scopes. The scopes we need are (at a minimum):

- repo
- read:packages

See also [here](https://docs.github.com/en/packages/publishing-and-managing-packages/about-github-packages#about-tokens

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

## Quickstart configuring docker for use with github packages

Login with github packages (where GH_TOKEN contains the value of your developer token and USERNAME your github username)

```
echo GH_TOKEN | docker login https://docker.pkg.github.com -u USERNAME --password-stdin
```

See also [here](https://docs.github.com/en/packages/using-github-packages-with-your-projects-ecosystem/configuring-docker-for-use-with-github-packages)
