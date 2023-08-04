# monas-submodule-demo
Example showing how to use a monas mono-repo as a submodule in another project

## Motivation

Monorepos encourage code reuse, but getting a team to fully commit to using a mono-repo structure for all code is difficult.
Using a poly-repo makes sharing common code difficult.

A third option exists: putting all shared libraries into a mono-repo, and then submoduling that shared library accross service repos.

This allows the mono-repo to stay lean (it only contains code that needs to be shared), and it allows for the flexibility offered by poly-repos.

This project demonstrates how to accomplish this with [monas](https://github.com/frostming/monas).

## Common Mono-Repo

To demonstrate how to use a common mono-repo in a service repo, this project contains a "service" called `acme-demo-service`.
This service depends on code from [monas-namespace-package-example](https://github.com/AGiantSquid/monas-namespace-package-example), which is a monas managed mono-repo.
That mono-repo is submoduled into this repo as a sibling directory of our service.

```
├── acme-demo-service                # code for our "service"
├── monas-namespace-package-example  # submoduled common mono-repo
└── pyproject.toml
```

Any other project that needs to reference this common library would similarly create a new repository, and then submodule the common mono-repo code.
All the service repository needs is the githash of the common library to version all of the shared code.

## Dependency Trail

This example contains a "service" called `acme-demo-service`.
This service has a dependency declared in its `pyproject.toml` on `acme-service-a`.
This dependency exists in the submoduled common mono-repo.
The package `acme-lib-a` depends on `acme-lib-b`, and `acme-lib-b` in turn depends on `acme-lib-c`, all of which live in the common mono-repo.

When installing `acme-demo-service`, monas will recursively find all of these local dependencies and install them in "editable" mode.
This means that a developer can edit code in `acme-lib-c`, and have the code change immediately available in `acme-demo-service`.

## Usage

To install the service, run
```
monas install --include acme-demo-service
```

This will create a .venv in the `acme-demo-service/` directory and install the `acme-demo-service` package and all of its local dependencies in "editable" mode.

Then, you can activate the .venv and begin using the local code.

```
cd acme-demo-service
source .venv/bin/activate
python -c "import acme.demo_service"
```

This should print:

```
hello from acme.demo_service, demo_service_func
hello from acme.lib_a, func_a
hello from acme.lib_b, func_b
hello from acme.lib_c, func_c
```

Changing the code in any of the local dependencies in the submodule will be reflected when running the code for `acme-demo-service` in its virtual environment.
