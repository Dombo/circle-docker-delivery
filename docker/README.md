### Dockerfile Hierarchy

##### Core Application

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