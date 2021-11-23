#### Issue Repro

This repo attempts to reproduce a undefined var edge case when
compiling macros with shadow set to parallel-build (the default).

Start the shadow build via:

```
clojure -M:cljs
```

Once the first pass has completed, modify and save the `project.a`
namespace. This should produce the following verbose output:

```
[:app] Compiling ...
-> Resolving Module: :main
<- Resolving Module: :main (4 ms)
-> build target: :browser stage: :compile-prepare
<- build target: :browser stage: :compile-prepare (0 ms)
-> Compile CLJS: project/a.cljs
<- Compile CLJS: project/a.cljs (0 ms)
-> Cache write: project/a.cljs
-> Compile CLJS: project/core.cljs
-> Compile CLJS: project/b.cljs
<- Compile CLJS: project/core.cljs (2 ms)
Cache skipped due to warnings: project.core
<- Cache write: project/a.cljs (11 ms)
<- Compile CLJS: project/b.cljs (35 ms)
-> Cache write: project/b.cljs
<- Cache write: project/b.cljs (11 ms)
-> build target: :browser stage: :compile-finish
<- build target: :browser stage: :compile-finish (2 ms)
-> build target: :browser stage: :flush
-> Flushing unoptimized modules
<- Flush: shadow/module/main/append.js (1 ms)
<- Flush: project/a.cljs (1 ms)
<- Flush: project/b.cljs (2 ms)
<- Flush: project/core.cljs (2 ms)
<- Flushing unoptimized modules (30 ms)
<- build target: :browser stage: :flush (32 ms)
[:app] Build completed. (122 files, 3 compiled, 1 warnings, 0.11s)

------ WARNING #1 - :undeclared-var --------------------------------------------
 File: /Users/zalky/src/shadow-repro/src/project/core.cljs:7:3
--------------------------------------------------------------------------------
   4 | 
   5 | (defn init!
   6 |   []
   7 |   (c/macro))
---------^----------------------------------------------------------------------
 Use of undeclared Var project.b/b
--------------------------------------------------------------------------------
   8 | 
--------------------------------------------------------------------------------
```

Note that this can be mitigated by dropping all the `(do :nothing)`
statements in `project.b` to reduce the compile time of `project.b`.

It appears that while the compilation each of namespace is launched in
dependency order, macro usage may require the serialization of
dependent namespaces that have transitive dependencies between them.

Serializing all compilation via `:parallel-build false` does also
sidestep the issue, but is probably overkill in the vast majority of
cases.