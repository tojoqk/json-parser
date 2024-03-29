# json-parser

This is a JSON parser implemented in Coalton.

## Installation

Since it depends on Coalton, please refer to the link below to install Coalton.

https://github.com/coalton-lang/coalton

Next, place json-parser in your local repository (`~/common-lisp`, etc.).

```shell:~/common-lisp
git clone https://github.com/tojoqk/json-parser.git
```

If you are using Quicklisp, you can load the system with the following.

```lisp
(ql:quickload :tokyo.tojo.json-parser)
```

## Examples

```lisp
(defpackage #:json-parser-example
  (:use #:coalton
        #:coalton-prelude)
  (:local-nicknames
   (#:parser #:tokyo.tojo.json-parser)
   (#:map #:coalton-library/ord-map)))

(in-package #:json-parser-example)

(named-readtables:in-readtable coalton:coalton)

(coalton-toplevel
  (declare get-number (parser:JSON -> (Optional Double-Float)))
  (define (get-number x)
    (match x
      ((parser:JSON-Number (parser:JSON-Integer n)) (as-optional (tryInto n)))
      ((parser:JSON-Number (parser:JSON-Float n)) (Some n))
      (_ None)))

  (declare get-string (parser:JSON -> (Optional String)))
  (define (get-string x)
    (match x
      ((parser:JSON-String str) (Some str))
      (_ None)))

  (declare get-object (parser:JSON -> (Optional (map:Map String parser:JSON))))
  (define (get-object x)
    (match x
      ((parser:JSON-Object m) (Some m))
      (_ None)))

  (declare eval (String -> (Optional Double-Float)))
  (define (eval str)
    (do (json <- (as-optional (parser:parse str)))
        (obj <- (get-object json))
        (left <- (>>= (map:lookup obj "left")
                      get-number))
        (op <- (>>= (map:lookup obj "op")
                    get-string))
        (right <- (>>= (map:lookup obj "right")
                       get-number))
        (match op
          ("+" (pure (+ left right)))
          (_ (default))))))
```

in REPL:

```lisp
CL-USER> (in-package #:json-parser-example)
#<COMMON-LISP:PACKAGE "JSON-PARSER-EXAMPLE">
JSON-PARSER-EXAMPLE> (eval "{\"left\": 10, \"op\": \"+\", \"right\": 32.0}")
#.(SOME 42.0d0)
JSON-PARSER-EXAMPLE> (eval "{\"left\": 10, \"op\": \"-\", \"right\": 32.0}")
#.NONE
JSON-PARSER-EXAMPLE>
```

## LICENSE

This program is licensed under the MIT License. See the LICENSE file for details.
