# Git sparse-checkout demonstration

A simple demonstration on how to use git submodules to support Core-CMS and Camino codebase customization on a per-project basis through the use of the git sparse-checkout command and git partial cloning filters for development and deployments.

## Background

The TACC Core Portal codebases have been designed to be as extensible and customizable as possible while still providing robust out-of-the-box solutions to the most common challenges faced by distributed multi-disciplinary research teams comprised of scientists, engineers, data analysts, programmers leveraging shared resources on collaborative projects that utilize common datasets and which require both storage and computational resources that scale.

In order to enable the rapid develpoment and deployment of these codebases, both the Core-CMS and Camino codebases require additional per-project scripts, settings and resources in order to properly configure, build and deploy the TACC Core Portal for a given specific project on specific target systems. Additional secret settings, those containing senstivie credentials and passwords, are stored external to the codebase and are provided on a per-project, per-developer basis as needed through internal secure tracking mechanisms such as UT Stache.

In order to simplify the overhead of establishing new deployment workflows on TACC resources, a single monorepo is utilized to track all of these resources across all projects. A single set of auth credentials then enables Jenkins to build images, publish images and deploy images to project host machines. To deploy a new portal (once the corresponding subdirectory and files are added to the [Core-Project-Resources](https://github.com/TACC/Core-Project-Resources) repo as detailed below), the name of the new project folder should be added as a parameterized option in the Jenkins deployment script options.

To prepare a new portal project for deployment, the `project-000-example` directory should be copied, renamed after the new project and populated accordingly with the values specific to the new project's target host machines and configuration settings.

The directory structure expected by the Core-CMS and Camino codebases is defined and explained in the `Project-000-example` subdirectory `README` file [here](project-000-example/README.md).

## Problem Description

We are storing all our project-specific resources in a single git repo that is used as a submodule in the `Core-CMS` codebase.

When using the submodules capability to pull in the repo containing the external assets and settings, it brings in the entire branch of content, to include all the subdirs for other projects as well.

When we publish the docker images for use in deployment, we do not want to include code, settings or assets that are not part of the specific project the image is being built for.

Also, when in local development, we do not want developers overwhelmed by seeing every projects resources in their codebase.

By using a `git sparse-checkout` [command](https://git-scm.com/docs/git-sparse-checkout) along with an `.env` file, we can programatically remove the extraneous repo content (from other portal projects) so that it is not bundled into the published image and is not littering the project developers local codebase.

Additionally, switching between different portal projects is as simple as updating the `.env` file and rerunning the `git sparse-checkout set ...` command (explained below) to switch to the submodule folder you want to be used during development and container/image builds.

## Goals

We seek to accomplish several things with this setup:

- Enable a manageable workflow for rapidly replicating and customizing our portal codebase.
- Enable a configuration management control procedure for retaining and versioning project portal deployments.
- Simplify the task of developing locally on the Core-\* and Camino codebases.
- Simplify the Jenkins configuration for generating new image builds, publishing images and deplying portal releases.
- Unify the development and deployment workflows across all portal projects and codebases that inter-operate (Core-Portal, Core-CMS, Core-Docs and Camino).

This process allows a single monorepo Submodule repository structure to be utilized across all Core-based portal projects while allowing another level of contextual isolation during development and image publishing phases.

## Demo

Steps to recreate a basic sparse-checkout of a single subfodler.

```bash
# Clone this test repo:
$ git clone git@github.com:taoteg/sparse-checkout-submodule-test.git --depth=1 --filter=blob:none --sparse sparse-test
$ cd sparse-test

# Initialize the sparse-checkout.
$ git sparse-checkout init --cone

# Inspect the repo.
# No files will be checked out yet.
$ tree
.
└── README.md

0 directories, 1 file

# Add a project.env file to repo root.
# This file is untracked (see .gitignore).
$ touch project.env

# Populate the project.env file like so:
$ echo "project-000-example" >> project.env

# list the repo now and you will see nothing but your project.env file.
.
├── README.md
└── project.env

0 directories, 2 files

# set the sparse-checkout filter by passing it the values in the project.var file.
$ git sparse-checkout set --stdin < project.env
remote: Enumerating objects: 25, done.
remote: Counting objects: 100% (25/25), done.
remote: Compressing objects: 100% (24/24), done.
remote: Total 25 (delta 0), reused 25 (delta 0), pack-reused 0
Receiving objects: 100% (25/25), 520.94 KiB | 1.26 MiB/s, done.
Updating files: 100% (25/25), done.

# Now you will only see the files from the specific subdirectory designated in the project.env file.
$ tree
.
├── README.md
├── project-000-example
│   ├── README.md
│   ├── customization
│   │   ├── README.md
│   │   ├── __init__.py
│   │   ├── secrets.py
│   │   ├── snippets
│   │   │   └── README.md
│   │   ├── static
│   │   │   ├── README.md
│   │   │   └── example-cms
│   │   │       ├── README.md
│   │   │       ├── css
│   │   │       │   └── src
│   │   │       │       ├── _imports
│   │   │       │       │   └── generic
│   │   │       │       │       └── roots.css
│   │   │       │       └── site.css
│   │   │       └── img
│   │   │           └── org_logos
│   │   │               ├── navbar_branding_example.png
│   │   │               ├── nsf-white.png
│   │   │               ├── portal.png
│   │   │               ├── tacc-white.png
│   │   │               └── utaustin-white.png
│   │   └── templates
│   │       ├── README.md
│   │       └── fullwidth.html
│   └── devops
│       ├── camino
│       │   ├── docker-compose.override.sample.yml
│       │   └── sample.env
│       ├── cms
│       │   └── secrets.sample.py
│       ├── nginx
│       │   └── nginx.conf
│       ├── portal
│       │   └── settings_secret.example.py
│       ├── rabbitmq
│       │   └── rabbitmq.env.sample
│       └── uwsgi
│           ├── uwsgi_cms.ini
│           ├── uwsgi_core.ini
│           └── uwsgi_params
└── project.env
```

You can now update your corresponding submodule and build customized images/containers with ONLY the targeted submodule present in the image artifact and any extraneous repo content removed without it being altered on the remote.

```
# To see the complete repo again, disable the sparse-checkout.
$ git sparse-checkout disable
remote: Enumerating objects: 1, done.
remote: Counting objects: 100% (1/1), done.
Receiving objects: 100% (1/1), 48 bytes | 48.00 KiB/s, done.
remote: Total 1 (delta 0), reused 1 (delta 0), pack-reused 0
Updating files: 100% (24/24), done.

# All files are now back:
$ tree -L 2
.
├── README.md
├── project-000-example
│   ├── README.md
│   ├── customization
│   └── devops
├── project-001
│   ├── README.md
│   ├── customization
│   └── devops
├── project-002
│   ├── README.md
│   ├── customization
│   └── devops
└── project.env
```

For further evidence of the ease of switching projects, update the `project.env` file and reapply the `git sparse-checkout set ...` command to see only the new project folders.

```
$ vim project.env

# Edit the text to read project-001
# Write and save the changes to disk (esc, :w, :q)

# Apply the sparse-checkout filter again.
$ git sparse-checkout set --stdin < project.env
$ tree
.
├── project-001
│   ├── README.md
│   ├── customization
│   │   ├── snippets
│   │   │   └── test.txt
│   │   ├── static
│   │   │   └── test.txt
│   │   ├── templates
│   │   │   └── test.txt
│   │   └── test.txt
│   └── devops
│       ├── camino
│       │   └── test.txt
│       ├── cms
│       │   └── test.txt
│       ├── nginx
│       │   └── test.txt
│       ├── portal
│       │   └── test.txt
│       ├── rabbitmq
│       │   └── test.txt
│       ├── test.txt
│       └── uwsgi
│           └── test.txt
└── project.env
```
