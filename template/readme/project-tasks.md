# Project Tasks

“how do I _____ on my local machine?”

## (re)Install Drupal

Pre-requisites to installation:

1. Ensure that `docroot/sites/default/settings/local.settings.php` exists by executing `blt setup:drupal:settings`.
1. Verify that correct local database credentials are set in `local.settings.php`.
1. Ensure that project dependencies have already been built via `blt setup:build`

To re-install Drupal, execute: `blt setup:drupal:install`. Note that this will drop the existing database tables and install Drupal from scratch!

## Update dependencies (core, profile, module, theme, libraries)

Composer should be used to manage Drupal core, all contributed dependencies, and most third party libraries. The primary exception to this is front end libraries that may be managed via a front-end specific dependency manager, such as [Bower](http://bower.io/) or [NPM](https://www.npmjs.com/).

Make sure to familiarize yourself with [basic usage](https://getcomposer.org/doc/01-basic-usage.md) of Composer, especially on how the [lock file](https://getcomposer.org/doc/01-basic-usage.md#composer-lock-the-lock-file) is used. In short: you should commit _both_ `composer.json` and `composer.lock` to your project, and every time you update `composer.json`, you must also run `composer update` to update `composer.lock`. You should never manually edit `composer.lock`.

To add a new package to your project, simply add it to the “require” section of `composer.json`, run `composer update`, and then commit _both_ the modified `composer.json` and `composer.lock`. To update an existing package, it’s generally sufficient to run `composer update`, and commit the modified `composer.lock`.

### Contributed projects and third party libraries

All contributed projects hosted on drupal.org, including Drupal core, profiles, modules, and themes, can be found on [Drupal packagist](https://packagist.drupal-composer.org/). Most non-Drupal libraries can be found on [Packagist](http://packagist.com/). For any required packaged not hosted on one of those two sites, you can define your own array of [custom repositories](https://getcomposer.org/doc/05-repositories.md#repository) for Composer to search.

Note that Composer versioning is not identical to drupal.org versioning. See:

* [Composer Versions](https://getcomposer.org/doc/articles/versions.md) - Read up on how to specify versions.
* [Drupal packagist site](https://packagist.drupal-composer.org/) - Find packages and their current versions.
* [Drupal packagist project](https://github.com/drupal-composer/drupal-packagist) - Submit issues and pull requests to the engine that runs Drupal packagist.
* [Drupal packagist project](https://github.com/drupal-composer/drupal-packagist) - Submit issues and pull requests to the engine that runs Drupal packagist.
* [Drupal Composer package naming conventions](https://www.drupal.org/node/2471927)
* [Packagist](http://packagist.com/) - Find non-drupal libraries and their current versions.

### Drupal core

To update drupal core:

1. Update the entry for `drupal/core` in the root composer.json.
2. Run `composer update`.
3. Run `./scripts/drupal/update-scaffold`. This will update the core files not included in `drupal/core`.
4. Use git to review changes to committed files. E.g., changes to .htaccess, robots.txt, etc.
5. Add and commit desired changes.

## Patch a project

Please see [patches/README.md](../patches/README.md) for information on patch naming, patch application, and patch contribution guidance.

## Deploy to cloud

Please see [Deploy](deploy.md) for a detailed description of how to deploy to Acquia Cloud.

## Run tests & code validation

Please see [tests/README.md](../tests/README.md) for information on running tests.

To execute PHP codesniffer and PHP lint against the project codebase, run:

```
blt validate:all
```

## Build front end assets

Ideally, you will be using a theme that uses SASS/SCSS, a styleguide, and other tools that require compilation. Like dependencies, the compiled assets should not be directly committed to the project repository. Instead, they should be built during the creation of a production-ready build artifact.

BLT allows you to define a custom command that will be run to compile your project's frontend assets. You can specify the command in your project's `project.yml` file under the `target-hooks.frontend-build` key:

```
target-hooks:
  frontend-build:
    # The directory in which the command will be executed.
    dir: ${docroot}
    command: npm install.
```

If you need to run more than one command, you may use this feature to call a custom script:

```
target-hooks:
  frontend-build:
    # The directory in which the command will be executed.
    dir: ${repo.root}
    command: ./scripts/custom/my-script.sh
```

This command will be executed when dependencies are built in a local or CI environment, and when a deployment artifact is generated. You may execute the command directly by calling the `frontend:build` target:

```
blt frontend:build
```

## Updating your local environment

The project is configured to update the local environment with a local drush alias and a remote alias as defined in `project.yml` or `project.local.yml`. Given that these aliases match, those in `drush/site-aliases/`, you can update the site with BLT. Please see [drush/README.md](../drush/README.md) for details on how to create these aliases.

### Refresh: Rebuild the codebase, copy the database, and run updates

This all in one command will make sure your local is in sync with the remote site.

```
blt local:refresh
```

### Sync: Copy the database from the remote site

```
blt local:sync
```

### Update: Run update tasks locally

```
blt local:update
```

These tasks can be seen in `build/core/phing/tasks/local-sync.xml`. An additional script can be added at `/hooks/dev/post-db-copy/dev-mode.sh` which would run at the end of this task.
