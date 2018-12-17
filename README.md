---

An example of leveraging docker throughout your entire development process.
Delivering a consistent environment while maintaining the benefits of tooling and a minimal production image.

### Motivation

This repository demonstrates shipping a consistent build, development & continuous integration environment through multi-stage builds.
You can leverage this pattern to ship a minimal release artefact - this offers benefits when it comes to elastic workloads.

The stages are tested to demonstrate a simple preventative measure against bad releases.

The build, test & release of the containers is conducted within CircleCI (but could easily be implemented elsewhere).
The build/test phases are only performed given changes to the relevant files, saving continuous integration resources.
Consult `.circleci/config.yml` for more detail.

If you implemented this in production, you'd use the CI stage to run your tests & build your source artefacts (source code).
You would copy the output (transpiled etc.) into the release container & deploy it.

At this point you've built, tested & developed your project against a base runtime - with all the benefits of additional tooling.
You've then gone on to run the application - against the same base runtime - without having to recompile/link it.
If you had expensive compilation steps during dependency installation for example, such as compiling native extensions for database adapters - you'd avoid having to run these twice while still delivering a minimal artefact.

### Components

- [CircleCI](https://circleci.com/docs/)
- [Docker](https://docs.docker.com/engine/reference/builder/)
- [Goss](https://github.com/aelsabbahy/goss)

### Dockerfile Hierarchy

```
                                          
+-------------------+                     +-----------------+
| Name: Project     |                     | Name: Rel       |
| Deps: Spec tools  |                     | Deps: Artefact  |
+-------------------+                     +--------+--------+
+-------------------+ +------------------+         ^
| Name: Dev         | | Name: CI         |         |
| Deps: QoL tooling | | Deps: TBD        +---------+
+-------------------+ +------------------+         |
                                                   |
+----------------------------------------+         |
| Name: Build                            |         |
| Deps: Build Tooling, Test Deps         |         |
+----------------------------------------+         |
                                                   |
+--------------------------------------------------+--------+
| Name: Base                                                |
| Deps: Runtime, Package Manager, Sys Deps, OS              |
+-----------------------------------------------------------+

```

### Testing

[Goss](https://github.com/aelsabbahy/goss) is used to assert that the critical components of the container layers remain intact.

You can target any layer you want by adding a goss.{layer_name}.yaml file with the desired assertions.

Convenience helpers for running the tests locally should be added to the root package.json run scripts.
Examples include `"test:base-container": "npm run test-container -- base"`

##### Developing Container Tests

1. Build the target layer wrapped in the goss test instructions (execute this from the repo root)

TARGET_LAYER=[ci|dev|build|release] or any stage in the multi-stage hierarchy you wish to test

`docker build -t developing-goss --build-arg TARGET_TEST_LAYER=$TARGET_LAYER -f docker/Dockerfile . --target=test`

2. Launch into the target layer via dgoss

`dgoss edit developing-goss`

Inspect the goss.yaml copied out at the end of this process.
More details can be found in the [dgoss](https://github.com/aelsabbahy/goss/tree/master/extras/dgoss) documentation.