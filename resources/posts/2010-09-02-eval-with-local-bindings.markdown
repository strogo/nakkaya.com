---
title: Eval with Local Bindings
tags: clojure
---

Using Clojure vectors as templates for
[static](http://github.com/nakkaya/static) meant they need to be read
and evaluated at runtime and for a template to be useful it needs access
to certain local bindings when rendering (i.e access to metadata is
needed to determine the title tags etc.) so the usual *read-string*
followed by *eval* won't work,

     ;;won't work
     (let [title "Eval with Local Bindings"] 
       (-> "[:title title]" read-string eval))

This is a consequence of lexical binding, binding is only in effect for
code that is physically in the let block, which means eval has only
access to global variables, one solution is to define the title just
before you use it,

     ;;will work but don't use.
     (def title "Eval with Local Bindings")
     (-> "[:title title]" read-string eval)

but thats like shooting your self in the foot because you just
lost the ability to run this piece of code in parallel, which brings us to
binding, binding makes a value available to the code inside the binding
and any functions called by that code,

     (declare title)
     (binding [title "Eval with Local Bindings"]
       (-> "[:title title]" read-string eval))

and since bindings are thread-local, you can bind stuff in multiple threads
and not worry about them messing each other. The part that took an hour
to figure out was that the above snippet worked in REPL and lein ran the
unit tests just fine but when ran from the jar it kept failing, after
some trial and error what made it work was to bind the \*ns\*,

     (declare title)
     (binding [*ns* (the-ns 'user)
               title "Eval with Local Bindings"]
       (-> "[:title title]" read-string eval))
