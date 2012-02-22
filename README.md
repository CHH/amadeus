# Amadeus, a simple generator for truly static Composer Repositories

## Why the heck is this useful?

Here's why:

 * Adding every Git Repository as repository in the package's
   `composer.json` is cumbersome and slows down installing of packages.
 * Imagine your packages live in _private_ Git repositories, probably
   because you are working in a company. You're using Composer for
   installing dependencies of your apps before deployment and don't want
   to embed the login info for the VCS server in your `composer.json`s.
   You want them to be served really fast too. With Amadeus you can use
   your CI system to continuously build packages to your repository and 
   upload the repository to S3 or something.

## Requirements

 * > PHP 5.3.3
 * PHAR Extension (PharData is used to generate Zip Archives)

## Install

Download the source and copy the `amadeus` file to wherever you
see fit. It's probably good to copy it somewhere in your `PATH`, 
for example to `/usr/local/bin`.

## Usage

Amadeus inits the repository in the current working directory,
where `amadeus` was invoked.

This directory must also contain an `amadeus.json` file, which is
Amadeus' configuration.

This is an example:

    // amadeus.json
    {
        "repository_url": "http://s3.amazonaws.com/acme-corp.com/a76a6d5asrd891623123/composer"
    }

The `repository_url` defines the location where your repository will be
available for usage. You should add this URL as Composer repository to
all packages which need dependencies from there:

    // composer.json
    {
        "name": "acme-corp/foo",
        "repositories": [
            {
                "type": "composer",
                "url": "http://s3.amazonaws.com/acme-corp.com/a76a6d5asrd891623123/composer"
            }
        ],
        "require": {
            "acme-corp/bar": ">=v1.0.0"
        }
    }

Amadeus has some prerequisites for your packages:

 * The version __must__ be either contained in the package's `composer.json`
   or passed as second argument to `amadeus build`.
 * Each package's `composer.json` __must__ contain the `source` key, which defines where
   the package's source can be retrieved. Usually this is a VCS.
   This is not needed for packages on Packagist, because you tell them
   the source location when creating a package. Amadeus has no way of
   knowing this, so you have to tell it:

       // composer.json
       {
            // ...
            "source": {
                "type": "git",
                "url": "git://github.com/acme-corp/foo.git",
                "reference": "master"
            }
       }

To add a package to your repository just call:

    % amadeus build <package_source_dir>

This adds the information to the `public/packages.json` in your
repository and makes an archive which it places in
`public/get/<package_name>-<version>.zip` in your repository.

Then upload all files generated in the `public` folder to a static web
host of your choice, e.g. S3.

> Optionally make sure no one is able to guess the public path of
> your repository.

