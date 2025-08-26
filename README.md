## Local Setup

Local development playground for Strapi v4 with MySQL.

1. `cp .env.example .env`
2. `docker compose build`
3. `docker compose up`
4. Open http://localhost:1337/admin to register admin (or import db dump if provided)

Tested on Mac M processor / Docker Desktop.

Make sure to stop other projects with MySQL/DB with `docker compose down`.

## Usage

Import database dump: 

```bash
docker compose exec strapiDB sh -c \
  'mysqldump -u root -p"$MYSQL_ROOT_PASSWORD" strapi' > dump.sql.gz
```

Restore a database dump:

```bash
docker compose exec -T strapiDB sh -c \
  'mysql -u root -p"$MYSQL_ROOT_PASSWORD" strapi' < dump.sql.gz
  ```

Install plugins via:

```bash
docker compose exec strapi npm install strapi-plugin-translate

# restart with CRTL + or when detached with docker compose up -d
docker compose down && docker compose up
```

## TODOs

- [ ] add instructions for db dumping & restoring
- [ ] re-add `sharp` for image resizing, issues (even with `platform: linux/amd64`)
  - [ ] remove `platform: linux/amd64` from strapi or use `strapi:latest`?
- [ ] add phpMyAdmin

(StrapiDB needs `platform: linux/amd64` since MySQL v5 has no ARM/M image)

## How was this created?

Followed [Setup strapi in docker by Ladabees (YouTube)](https://www.youtube.com/watch?v=dzctBJtNTfs)

1. Created project folder, `cd` to it
2. `npx create-strapi-app@legacy .`, select "Quickstart"
3. Removed `package-lock.json` (for now due to simplicity), removed local `node_modules/`
3. Added Dockerfile, docker compose and dockerignore file from https://docs-v4.strapi.io/dev-docs/installation/docker, adapted them - see below:

Ran into 

```
strapi    | Knex: run
strapi    | $ npm install mysql --save
strapi    | Cannot find module 'mysql'
strapi    | Require stack:
strapi    | - /opt/node_modules/knex/lib/dialects/mysql/index.js
```

Also ran into when trying to install `mysql2` for knex locally with npm v10:

```
npm error code ERR_INVALID_ARG_TYPE
npm error The "path" argument must be of type string or an instance of Buffer or URL. Received null
```

Added this to `package.json` instead

```
    "mysql2": "3.14.3",
```

and switched client to `mysql2` in `.env`.

`npm install` is done in Dockerfile.

Also needed to downgrade to `npm v9` in Dockerfile, due to some "ERR_INVALID_ARG_TYPE with "path" being null" issues with npm v10 locally.

Also needed to use `platform: linux/amd64` for now as well for `strapi` container, since build fails for sharp:

```
79.01 npm ERR! sharp: Building from source via node-gyp 79.01 npm ERR! gyp info it worked if it ends with ok 79.01 npm ERR! gyp info using node-gyp@9.4.1 79.01 npm ERR! gyp info using node@18.20.8 | linux | arm64 79.01 npm ERR! gyp info find Python using Python version 3.12.11 found at "/usr/bin/python3"
```

But did not work as well, therefore disabled sharp dependencies for now.

Then freshly rebuild containers:

```bash
# remove local folder, left over from install
rm -rf node_modules/

# Rebuild
docker-compose build
docker-compose up
```

### Further resources

- https://docs.strapi.io/cms/installation/cli
- https://docs-v4.strapi.io/dev-docs/installation/docker
