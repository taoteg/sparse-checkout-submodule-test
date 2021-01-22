# Git sparse-checkout demonstration

For use with targeted submodules. 

## Description

We are storing all our project-specific resources in a single git repo that is used as a submodule in the `Core-CMS` codebase.

When using the submodules capability to pull in the repo containing the external assets and settings, it brings in the entire branch of content, to include all the subdirs for other projects as well.

When we publish the docker images for use in deployment, we do not want to include code, settings or assets that are not part of the specific project the image is being built for.

Also, when in local development, we do not want developers overwhelmed by seeing every projects resources in their codebase.

By using a sparse-checkout along with an `.env` file, we can programatically remove the extraneous repo content (from other portal projects) so that it is not bundled into the published image and is not littering the project developers local codebase.

Additionally, switching between different portal projects is as simple as updating the `.env` file and rerunning the `sparse-checkout set ...` command (explained below) to switch to the submodule folder you want to be used during development and container/image builds.

## Demo

Steps to recreate a basic sparse-checkout demo.

```
# Clone this test repo:  
$ git clone git@github.com:taoteg/sparse-checkout-submodule-test.git

# You will see these files:
$ tree
.
├── also-not-in-sparse
│   └── test.txt
├── example-project
│   ├── customization
│   │   └── test.txt
│   ├── devops
│   │   └── test.txt
│   └── test.txt
└── not-in-sparse
    └── test.txt

# Add a project.env file to repo root.
$ touch project.env

# Populate the project.env file like so:
$ echo "project.env" >> project.env
$ echo "example-project" >> project.env

# Initialize the sparse-checkout.
$ git sparse-checkout init

# list the repo now and you will see nothing but your project.env file.
$ tree
.
└── project.env

# set the sparse-checkout filter by passing it the values in the project.var file.
$ git sparse-checkout set --stdin < project.env

# Now you will only see the files from the specific subdirectory designated in the project.env file.
$ tree
.
├── example-project
│   ├── customization
│   │   └── test.txt
│   ├── devops
│   │   └── test.txt
│   └── test.txt
└── project.env

3 directories, 4 files
```

Now you can build your images/containers with ONLY the targeted submodule present in the image artifact and extraneous repo content removed without being effected on the remote.

```
# To see the repo files again, disable the sparse-checkout.
$ git sparse-checkout disable

# They are all back (plus your new project.env file):
$ tree
.
├── also-not-in-sparse
│   └── test.txt
├── example-project
│   ├── customization
│   │   └── test.txt
│   ├── devops
│   │   └── test.txt
│   └── test.txt
├── not-in-sparse
│   └── test.txt
└── project.env
```

This process would allow for the proposed unified Submodule repository structure to be utilized while allowing another level of contextual isolation during development and publishing phases. A single private repo can be maintained privately that hods a top-level directory for every portal project (both full portals and SAD CMS sites). All resources for a project can be placed into the predesignated directory structure for that project. Jenkins can then be pointed at a single repo for ALL submodule content, yet still filter out only what it needs during builds, publishing and depoyments. The same goes for the local developers.
