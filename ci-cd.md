# CI/CD

## Goal and motivation

- The goal with our CI/CD implementation was to provide a seamless integration between Hasura environments in particulr
- Why was this challenging? Hasura out of the box does not track changes. Likewise, it is GUI driven, which meant that changes are stored in Hasura's internal DB (this is trackable, but not ergonomic). The solution to this was to kill the default console and require everyone to use the CLI which generates a proper `migrations` & `metadata` folder.

## Workflow

!(diagram)[brewbean_env.png "Brewbean Diagram"]

## Environments

| environment | graphql                     | server                  | client            |
| ----------- | --------------------------- | ----------------------- | ----------------- |
| production  | graphql.brewbean.io         | api.brewbean.io         | brewbean.io       |
| staging     | staging-graphql.brewbean.io | staging-api.brewbean.io | stage.brewbean.io |
| development | localhost:8080              | localhost:4000          | localhost:3000    |

### Environment Variables & Files

#### GraphQL

**production & staging**

set via:
```console
$ heroku config:set {key}={value} --app gql-brewbean
```
_example shows setting for production app `gql-brewbean`_

**Environment variables**
```bash
HASURA_GRAPHQL_ADMIN_SECRET
HASURA_GRAPHQL_CORS_DOMAIN
HASURA_GRAPHQL_DEV_MODE
HASURA_GRAPHQL_ENABLED_APIS
HASURA_GRAPHQL_ENABLE_CONSOLE
HASURA_GRAPHQL_JWT_SECRET
HASURA_GRAPHQL_UNAUTHORIZED_ROLE
```

#### Client

* use `.env` & `.env.development` for setting variables in conjunction with `netlify.toml`
* ***For clarification: environment variables are not sensitive for client. This is just for setting different environment cleanly***

### Changelog

**Netlify DNS**

* Cannot generate a subdomain of deploy previews which mean that deploy previews are essentially non-functional if I add CORS security for brewbean staging environment ([ref](https://answers.netlify.com/t/netlify-dns-deploy-preview-and-branch-deploys/2168/27)). Can change in the future though
* Got rid of deploy previews for CORS reason (cannot add the deploy preview URL safely for CORS check on Hasura)


### Docker Setup

**First step is to apply migrations and metadata**

```console
$ hasura migrate apply --envfile .env.development 
$ hasura metadata apply --envfile .env.development
```
_You must do this from the `/graphql` repo folder_

**Check on migration status**
```console
$ hasura migrate status --envfile .env.development
VERSION        NAME  SOURCE STATUS  DATABASE STATUS
1614143269575  init  Present        Present
```

**Get pg_dump from heroku hasura instance ([ref](https://devcenter.heroku.com/articles/heroku-postgres-import-export#download-backup))**

```console
$ heroku pg:backups:capture -a gql-brewbean
$ heroku pg:backups:download -a gql-brewbean
```

- **in this case `-a gql-brewbean` is stating to grab backup from the `gql-brewbean` app**

**Adding a pg_dump (backup) into local environment** - [ref](https://gist.github.com/jgillman/8191d6f587ffc7207a88e987e034b675)

```console
$ docker exec -i fe9157eaff44 pg_restore --verbose --clean --no-acl --no-owner -U postgres -d postgres < latest.dump
```

#### Cleanup
`docker-compose down --volumes` clears all local data in local hasura instance


### Changed `master` to `main`
* keeping all repos consistent with new repo **brewbean/graphql**
* how to migraate:
```console
$ git branch -m master main
$ git fetch origin
$ git branch -u origin/main main
```