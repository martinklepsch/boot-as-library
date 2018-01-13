# Boot as a library?

Some recent discussions with @thheller and others prompted me to investigate if Boot could
be used as a library with a separate tool to construct the initial classpath, say `clj`.

In my hypothetical thinking one should be able to do something like this:

```clojure
(ns boot.deploy
  (:require [boot.core :as b]
            [boot.tasks.built-in :as tasks]))

(defn -main []
  (boot (pom ,,,) 
        (jar)
        (push ,,,)))
```

And the run it using `clj` (provided one has boot dependencies in their `deps.edn`):

```
clj -m boot.deploy
```

### Known Problems

The above is all a bit further out but one thing I've tried already is this:

```bash
java -cp "$(clj -Spath)" boot.App repl
```

It gives you a REPL that can run basic Clojure stuff, e.g.

- `(def a 1)`
- `(+ a 2)`
- `(boot (with-pass-thru _ (println "test")))`
- `(require 'boot.pod)`
- `(count boot.pod/pods)` (yields '2' after repl start)

but fails with the slightly more complex (pod-using?) things I've tried:

- creating pods: `(boot.pod/make-pod {:dependencies [['bidi "2.1.3"]]})`
- running tasks: `(boot (pom :project 'a/b :version "1.2.3") (target))`

Most of these fail with some "Pop without matching push" exceptions (see below).

I assume those can somehow be traced back to the fact that there's — in
contrast to when `boot` is used — no `Boot.class` or
`boot.ParentClassLoader.class` on the classpath. 

```
clojure.lang.ExceptionInfo: Pop without matching push {:line 13}
        at clojure.core$ex_info.invokeStatic(core.clj:4739)
        at clojure.core$ex_info.invoke(core.clj:4739)
        at boot.main$_main$fn__1201.invoke(main.clj:222)
        at boot.main$_main.invoke(main.clj:216)
        at clojure.lang.Var.invoke(Var.java:396)
        at org.projectodd.shimdandy.impl.ClojureRuntimeShimImpl.invoke(ClojureRuntimeShimImpl.java:159)
        at org.projectodd.shimdandy.impl.ClojureRuntimeShimImpl.invoke(ClojureRuntimeShimImpl.java:150)
        at boot.App.runBoot(App.java:399)
        at boot.App.main(App.java:491)
Caused by: java.lang.IllegalStateException: Pop without matching push
        at clojure.lang.Var.popThreadBindings(Var.java:333)
        at clojure.core$pop_thread_bindings.invokeStatic(core.clj:1923)
        at clojure.core$pop_thread_bindings.invoke(core.clj:1923)
        at boot.core$run_tasks.invoke(core.clj:1019)
        at boot.core$boot$fn__933.invoke(core.clj:1031)
        at clojure.core$binding_conveyor_fn$fn__5476.invoke(core.clj:2022)
        at clojure.lang.AFn.call(AFn.java:18)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
        at java.lang.Thread.run(Thread.java:745)
Boot failed to start:
java.lang.IllegalStateException: Pop without matching push
        at clojure.lang.Var.popThreadBindings(Var.java:333)
        at clojure.core$pop_thread_bindings.invokeStatic(core.clj:1923)
        at clojure.core$pop_thread_bindings.invoke(core.clj:1923)
        at boot.main$_main.invoke(main.clj:174)
        at clojure.lang.Var.invoke(Var.java:396)
        at org.projectodd.shimdandy.impl.ClojureRuntimeShimImpl.invoke(ClojureRuntimeShimImpl.java:159)
        at org.projectodd.shimdandy.impl.ClojureRuntimeShimImpl.invoke(ClojureRuntimeShimImpl.java:150)
        at boot.App.runBoot(App.java:399)
        at boot.App.main(App.java:491)
```




