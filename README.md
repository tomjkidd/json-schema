[![CircleCI](https://circleci.com/gh/luposlip/json-schema/tree/master.svg?style=svg)](https://circleci.com/gh/luposlip/json-schema/tree/master) [![Clojars Project](https://img.shields.io/clojars/v/luposlip/json-schema.svg)](https://clojars.org/luposlip/json-schema) [![Dependencies Status](https://versions.deps.co/luposlip/json-schema/status.svg)](https://versions.deps.co/luposlip/json-schema) [![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0) [![Downloads](https://versions.deps.co/luposlip/json-schema/downloads.svg)](https://versions.deps.co/luposlip/json-schema)

# Clojure JSON Schema Validator

```clojure
[luposlip/json-schema "0.1.7"]
```

A Clojure library for data validation according to JSON Schema https://json-schema.org.

It's very simple, and supports JSON Schema Draft-07.

## Usage

This library can be used to validate data (EDN or JSON strings) based on a JSON Schema.

It has a single public function, `validate` that return the validated data when no errors are found. This makes it very easy to incorporate validation in a pipeline (see pseudo example below).

All found errors will cause the function to throw an instance of `clojure.lang.ExceptionInfo`, which can be inspected with the help of `ex-data` and the likes.

If you're using Clojure 1.7 or newer, you can convert any Throwable to a map via `Throwable->map`.

To switch draft versions, simply use the according version notation in the `$schema` uri (in the root of the JSON document):

```
{"$schema": "http://json-schema.org/draft-04/schema", ...}

or:
{"$schema": "http://json-schema.org/draft-06/schema", ...}

or:
{"$schema": "http://json-schema.org/draft-07/schema", ...}
```

JSON and JSON Schema params has to be input as either a JSON encoded string or EDN (map for both or vector for JSON).

Example usage:

```clojure
(let [schema {:$schema "http://json-schema.org/draft-07/schema#"
              :id "https://luposlip.com/some-schema.json"
              :type "object"
              :properties {:id {:type "number"
                                :exclusiveMinimum 0}}
              :required [:id]}
      json "{\"id\": 0.001}"] ;; get from url, or anywhere
  (validate schema json)
  (comment do whatever you only wanna do when JSON is valid)
  :success)
```

Pseudo-example for pipelining (note the reuse of the prepared schema):

```clojure
(let [schema (json-schema/prepare-schema
                (-> "resources/json-schema.json"
                    slurp
                    (cheshire.core/parse-string true)))]
  (->> huge-seq-of-edn-or-jsonstrings
       (map do-stuff-to-each-doc)
       (map do-even-more-to-each)
       (map (partial json-schema/validate schema))
     (lazily-save-docs-to-disk "/path/to/output-filename.ndjson")
     dorun))
```

More usage examples can be seen in the tests.

## deps.edn

If you use `tools.deps` (as opposed to Leiningen), you'll have to copy all dependencies from the Java library manually into `deps.edn`. This is due to [a bug in `tools.deps`](https://dev.clojure.org/jira/browse/TDEPS-46). Refer to [issue #1](https://github.com/luposlip/json-schema/issues/1) for more information.

## Thanks

To the maintainers of: https://github.com/everit-org/json-schema, on which this Clojure Library is based.

## Copyright & License

Copyright (C) 2019 Henrik Mohr

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

            http://www.apache.org/licenses/LICENSE-2.0
            
Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
