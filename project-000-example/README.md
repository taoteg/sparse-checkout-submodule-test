# Project-000-Example

Copy this directory structure and replace the contents in order to create a customized build of the TACC Core Portal.

The directory structure expected by the Core-CMS and Camino codebases is as follows:

```bash
# Directory Structure of project-000-example
$ tree
.
├── README.md
└── project-000-example
    ├── README.md
    ├── customization           # <-- Submodule content checked out into the `taccsite_custom` folder under the Core-CMS codebase (and also used by the Core-Portal).
    │   ├── README.md
    │   ├── __init__.py
    │   ├── secrets.py
    │   ├── snippets
    │   │   └── README.md
    │   ├── static
    │   │   ├── README.md
    │   │   └── example-cms
    │   │       ├── README.md
    │   │       ├── css
    │   │       │   └── src
    │   │       │       ├── _imports
    │   │       │       │   └── generic
    │   │       │       │       └── roots.css
    │   │       │       └── site.css
    │   │       ├── fonts
    │   │       └── img
    │   │           └── org_logos
    │   │               ├── navbar_branding_example.png
    │   │               ├── nsf-white.png
    │   │               ├── portal.png
    │   │               ├── tacc-white.png
    │   │               └── utaustin-white.png
    │   └── templates
    │       ├── README.md
    │       └── fullwidth.html
    └── devops                  # <-- Submodule content checked out into the `conf` folder under the Camino codebase.
        ├── camino
        │   ├── docker-compose.override.sample.yml
        │   └── sample.env
        ├── cms
        │   └── secrets.sample.py
        ├── nginx
        │   └── nginx.conf
        ├── portal
        │   └── settings_secret.example.py
        ├── rabbitmq
        │   └── rabbitmq.env.sample
        └── uwsgi
            ├── uwsgi_cms.ini
            ├── uwsgi_core.ini
            └── uwsgi_params
```
