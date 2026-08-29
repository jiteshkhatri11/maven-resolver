# Dependency Management



Dependency management controls how dependency information is modified during dependency collection. It is an important part of resolving a project's dependency graph, especially when dependencies are inherited or introduced transitively.



## Dependency Collection and Resolution



Resolver separates dependency collection from dependency resolution.



**Dependency collection** builds the dependency graph. It determines which dependencies are reachable from the root dependency and applies dependency management while constructing that graph.



**Dependency resolution** takes the collected dependency graph and resolves the artifacts required for the final classpath.



This distinction is useful when troubleshooting dependency management because managed dependency information is applied during collection, before the final artifacts are resolved.



## Transitive Dependencies



A dependency can introduce additional dependencies of its own. These are called transitive dependencies.



For example:



```text

application

└── A:1.0

&#x20;   └── B:1.0

&#x20;       └── C:1.0

```



Here, `B` is a transitive dependency of `application`, and `C` is a transitive dependency introduced through `B`.



Dependency management can change the dependency information used while traversing this graph.



## Managed Dependencies



Resolver represents managed dependency information separately from ordinary dependencies.



The `CollectRequest` can contain managed dependencies, while the `DependencyManager` is responsible for applying dependency management during collection.



Managed information can affect properties such as:



* dependency version

* dependency scope

* dependency exclusions

* optionality

* other dependency attributes supported by the resolver



For example, consider:



```text

application

└── A:1.0

&#x20;   └── B:1.0

```



If dependency management specifies `B:2.0`, the dependency collected through `A` can be managed to use version `2.0`.



The important distinction is that dependency management does not simply add another dependency to the graph. Instead, it modifies dependency information as dependencies are collected.



## Dependency Management vs Dependency Mediation



Dependency management and dependency mediation solve different problems.



**Dependency management** provides rules for modifying dependency information during collection.



**Dependency mediation** determines which dependency is selected when multiple paths in the dependency graph provide conflicting versions.



For example:



```text

application

├── A:1.0

│   └── B:1.0

└── C:1.0

&#x20;   └── B:2.0

```



The graph contains two paths to `B`. Dependency mediation is responsible for resolving this conflict according to the configured conflict-resolution rules.



Dependency management, on the other hand, can explicitly manage the dependency information for `B` before conflict resolution takes place.



Keeping these concepts separate makes dependency-resolution behavior easier to understand and troubleshoot.



## DependencyManager



The `DependencyManager` API is used during dependency collection to apply dependency management rules.



A dependency manager can derive managed dependency information from the current dependency context and apply it to dependencies encountered further down the graph.



This allows dependency management to be inherited through the dependency graph.



Resolver implementations can provide different dependency-management behavior through the `DependencyManager` interface.



## DependencyManagement



`DependencyManagement` represents the managed dependency information that is available for a dependency.



It can contain information such as:



* managed version

* managed scope

* managed optionality

* managed exclusions



The dependency manager uses this information when processing dependencies during collection.



## DependencyManagerUtils



Resolver also provides `DependencyManagerUtils` for working with dependency-management information attached to dependency nodes.



When verbose dependency-management information is enabled, Resolver can retain information about the dependency before dependency management modifies it.



This additional information can be useful when investigating why a dependency has a particular version or scope.



For example, applications can enable:



```java

DependencyManagerUtils.CONFIG\_PROP\_VERBOSE

```



when collecting dependencies if they need to inspect the original dependency information.



The API documentation for this configuration property describes how the additional information can be accessed.



## Example



Consider the following dependency graph:



```text

root

├── A:1.0

│   └── B:1.0

└── C:1.0

&#x20;   └── B:2.0

```



Without dependency management, both versions of `B` can appear in the collected dependency graph before conflict resolution selects the version that will be used.



Now suppose dependency management specifies:



```text

B -> 2.0

```



During dependency collection, the managed version can be applied to dependencies for `B`.



The resulting dependency information can therefore differ from the original declaration:



```text

Original:

B:1.0



Managed:

B:2.0

```



This is why dependency collection should be considered when investigating dependency-management behavior.



## Troubleshooting



When dependency resolution produces an unexpected version or scope, inspect the dependency graph rather than looking only at the final resolved artifacts.



The dependency graph can contain information that is not visible in a simplified dependency tree.



For example, verbose conflict-resolution information can be enabled with:



```java

ConflictResolver.CONFIG\_PROP\_VERBOSE

```



This preserves conflicting nodes and can help identify the paths that introduced a dependency.



Similarly, enabling:



```java

DependencyManagerUtils.CONFIG\_PROP\_VERBOSE

```



can preserve dependency information from before dependency management was applied.



These options can help answer questions such as:



* Why was a particular dependency version selected?

* Which dependency introduced the dependency?

* Which managed version changed the dependency?

* Why does a dependency have a particular scope?

* Which paths in the graph contain conflicting versions?



## Relevant Resolver APIs



The following APIs are particularly relevant when working with dependency management:



* `org.eclipse.aether.collection.DependencyManager`

* `org.eclipse.aether.collection.DependencyManagement`

* `org.eclipse.aether.collection.CollectRequest`

* `org.eclipse.aether.util.graph.manager.DependencyManagerUtils`

* `org.eclipse.aether.util.graph.transformer.ConflictResolver`



Refer to the API documentation for the exact behavior and configuration options of these components.



## Summary



Dependency management is applied during dependency collection and modifies dependency information as the dependency graph is constructed.



Dependency collection builds the graph, dependency management modifies dependency information according to management rules, and dependency mediation resolves conflicts between competing dependency paths.



Understanding these stages makes it easier to reason about transitive dependencies and diagnose unexpected dependency versions, scopes, and other dependency attributes.
