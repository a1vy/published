# NPM smart publishing [![:white_check_mark:](https://circleci.com/gh/fiverr/published.svg?style=svg&circle-token=c887f45cd0a168ce3a1a304923f92bff11cccd81)](https://circleci.com/gh/fiverr/published)

[![NPM](https://nodei.co/npm/published.png)](https://www.npmjs.com/package/published)

## TL;DR
| Branch type | action |
| --- | --- |
| **Feature branch** | Release RC versions on tag by branch name. |
| **Master branch** | Release clean semver on "latest" tag. Create a git tag. |

---

## Use on your development machine
```sh
npm i -g published

published [testing]
```

### As a module
```js
const publish = require('published');

const result = await publish({testing: true}); // Publish version 1.1.0 to tag latest.
```

---

# CI-CD Usage
## Use NPX runner
```sh
npx published [testing]
```

### NPM Permissions
In order to publish an NPM package as a privileged user, create an NPM configuration file. One way to do it is to hide the token in an environment variable and add this preceding step:

```sh
echo "//registry.npmjs.org/:_authToken=$NPM_TOKEN" >> ~/.npmrc
```

### Slack notifications
Slack notifications will be sent using environment variable `SLACK_WEBHOOK`.

The program will post a message to the channel set in environment variable `SLACK_CHANNEL`, or pull from the default constant `DEFAULT_SLACK_CHANNEL` (`#publish`)

```
example_steps:
- echo "//registry.npmjs.org/:_authToken=$NPM_TOKEN" >> ~/.npmrc
- npm i
- npm t
- npx published
```

---

# Decision scheme

## Feature branch
- Published only if the version has a pre-release section which contains `rc`:
- Branch versions get a suffix that matches the commit ID
- Tags are named after the branch

## Master branch
- Only publishes clean semver versions, no pre-release
- Publishes versions to tag `latest`
- Creates tag in git repository

| branch | version | Will publish | to tag | Will create git tag |
| --- | --- | --- | --- | --- |
| `my_feature_branch` | `"1.3.0"` | nothing | N/A | No
| `my_feature_branch` | `"1.3.1-alpha"` | nothing | N/A | No
| `my_feature_branch` | `"1.3.1-rc"` | `1.3.1-rc-c447f6a` | `my_feature_branch` | No
| `my_feature_branch` | `"1.3.1-rc.1"` | `1.3.1-rc.1-c447f6a` | `my_feature_branch` | No
| `master` | `"1.3.0"` | `1.3.0` | `latest` | Yes
| `master` | `"1.3.0-beta"` | Throws Error | N/A | No
| `master` | `"1.3.0-rc"` | Throws Error | N/A | No

> ## Example flow:
>
> ### @fiverr/package:
> #### Developing
> `git checkout -b my_awesome_feature`
> `npm version 1.4.0-beta`
> `git push origin my_awesome_feature`
> - Circle CI works, does not release package version
>
> #### Release candidate (live testing)
> `npm version 1.4.0-rc`
> `git push origin my_awesome_feature`
> - Circle CI works, releases package version `1.4.0-rc-c447f6a` to tag `my_awesome_feature`
>
> ### Consuming Service
> #### Testing
> `npm i -S @fiverr/package@my_awesome_feature`
> - Installs `@fiverr/package` version `1.4.0-rc-c447f6a`
>
> ### @fiverr/package:
> #### Fix issue in release candidate
> `git push origin my_awesome_feature`
> - Circle CI works, releases package version `1.4.0-rc-a594081` to tag `my_awesome_feature`
>
> ### Consuming Service
> #### Testing
> `npm i -S @fiverr/package@my_awesome_feature` // same tag
> - Installs `@fiverr/package` version `1.4.0-rc-a594081` // new version
>
> ### @fiverr/package:
> #### Finalising
> `npm version 1.4.0`
> `git push origin my_awesome_feature`
> - Circle CI works, releases nothing
>
> `git checkout master`
> `git merge my_awesome_feature`
> `git push origin master`
> - Circle CI works, releases package version `1.4.0` to tag `latest`
>
> ### Consuming Service
> #### Using
> `npm i -S @fiverr/package@latest`
> - Installs `@fiverr/package` version `1.4.0`

[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Ffiverr%2Fpublished.svg?type=large)](https://app.fossa.io/projects/git%2Bgithub.com%2Ffiverr%2Fpublished?ref=badge_large)
