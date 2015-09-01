Apoc
====

Apoc is a node module and a command-line tool for making dynamic Cypher queries. Using its **Apoc Cypher Format** (ACF), it adds the following features on top of Cypher.

* Comments using `#`
* JavaScript code within backticks
* Variables between %% (when used as a node module)
* Multiple query statements in one file
* Ability to include other ACF files

Apoc is not a mapper (ORM, ODM, ONM, OxM) of any kind, nor does it provide any "friendly" or "improved" transaction methods on top of the Neo4j REST API. It is just a tool for enhancing your experience with Cypher. You will still need to write your Cypher queries, but Apoc will make them more powerful and much easier to use.

## Installation

As a node module:

```
$ npm install apoc --save
```

When installed as a node module, you can either directly pass Cypher / ACF queries to Apoc or load an ACF file with Cypher / ACF queries for Apoc to execute.

As a commandline tool:

```
$ npm install apoc -g
```

When installed as a commandline tool, you will be able to execute ACF files from the command-line, using the `apoc` command.

## Configuration

Apoc will look for your Neo4j configuration details in two places:

1. `.apoc.yml` file in your home directory
2. Shell's environment variables.

**Sample .apoc.yml**:

```
protocol: http
host: 192.168.0.8
port: 2902
username: neo4j
password: neo4j
```

**Environment variables**

```
NEO4J_PROTOCOL=http
NEO4J_HOST=192.168.0.5
NEO4J_PORT=7474
NEO4J_USERNAME=neo4j
NEO4J_PASSWORD=j4neo
```

The value defined in the environment variables will take precedence over the values set in the `.apoc.yml` file. `host` and `port` will default to 127.0.0.1 and 7474, respectively.

## Apoc Cypher File

An Apoc Cypher file is a plain-text file with a .acf extension, which contains Cypher / ACF queries.

Example of an ACF file:

```
# Create the world
CREATE (n:ApocTest { name: 'World' }) RETURN n

# Details about Asia
include includes/asia.acf

# Details about Spain
include includes/extra/spain.acf
```

The contents of `includes/asia.acf`:

```
CREATE (n:ApocTest { name: 'Asia' }) RETURN n

include misc.acf
include extra/india.acf
```

The contents of `includes/extra/spain.acf`:

```
CREATE (n:ApocTest { name: 'Spain' }) RETURN n
```

With respect to this ACF file, Apoc will look for a sibling directory named `includes` and a child directory named `extra` within `includes`, and include the files specified in the ACF file.

### Variable placeholders

```
MATCH (n:%type%) RETURN n
```

Variable placeholders are marked with a variable name between %%. The variable placeholder is replaced with the corresponding variable value, when it is passed in a **variables** object as the second parameter of an apoc query.

```
var apoc = require('apoc')
var query = apoc.query('MATCH (n:%type%) RETURN n', { type: 'Dog' })
```

resulting query:

```
MATCH (n:Dog) RETURN n
```

### JavaScript Code

```
CREATE (n:Animal { time: `Date.now()` }) RETURN n
```

Apoc will interpret any string within backticks as JavaScript code, and try to execute it. However, the API is limited only to the core JavaScript API provided by V8, hence it has no access to node's APIs or the objects created by you. 

In case you want to make any external object available to the JavaScript code, you can pass in a **context** object as the third parameter to the apoc query.

```
var apoc = require('apoc')
var acf = 'CREATE (n:Info { node: "`versions.node`", sum: `add(40, 1)` }) RETURN n'
var query = apoc.query(acf, {}, {
  versions: process.versions,
  add: function(a, b) {
    return a + b
  }
})
```

resulting query:

```
CREATE (n:Status { node: '0.10.36', sum: 41 }) RETURN n
```

The variables and the context objects maintain their order in an apoc query. Therefore, the variables object (even if empty) should always preceed the context object.

### Line breaks

A single line break can be used to aesthetically break long query statements. Each line is understood as a part of the same query statement.

```
CREATE (a:ApocTest { word: 'Naina' })
CREATE (b:ApocTest { word: 'Eye' })
CREATE (a)-[r:MEANS]->(b) RETURN a, b
```

A empty linebreak in an ACF file is used to separate query statements. All of the following are interpreted and executed as independent, separate queries.

```
CREATE (n:ApocTest { lang: 'hi', word: 'Naina' }) RETURN n

CREATE (n:ApocTest { lang: 'es', word: 'Ojo' }) RETURN n

CREATE (n:ApocTest { lang: 'it', word: 'Occhio' }) RETURN n
```

Want to see some sample ACF files? Look under the `test/fixtures` directory of this project.

## Usage

### As a node module

Here is a quick preview of how the Apoc API looks like. Details will be explained in the next section.

Simple example of using an inline query:

```js
var apoc = require('apoc')
var query = apoc.query('MATCH (n) RETURN n')
console.log(query.statements) // array of statements in this query
query.exec().then(function (result) {
  console.log(result)
}, function (fail) {
  console.log(fail)
})
```

Simple example of using an ACF file query:

```js
var query = apoc.query('./test/fixtures/multiline.acf')
console.log(query.statements) // array of statements in this query
query.exec().then(function (result) {
  console.log(result)
}, function (fail) {
  console.log(fail)
})
```

The `apoc` module exposes a single method called `query` with the following signature:

**apoc(query | apoc file [,variables] [,context])**

|Parameter|Description
|----|----------
|**query**| Cypher / ACF query. ACF queries should be accompanied by their `variable` and / or `context` objects.
|**apoc file**| Path to a .acf file. ACF queries in the file should be accompanied by their `variable` and / or `context` objects.
|**variables**| Object of variables to be used with ACF queries. A variable placeholder is marked with enclosing %%. For example: `"%username%"`, will become `"yaapa"`, if `{username:"yaapa"}` was used.
|**context**| Object of variables and functions, which are made available to the JavaScript code in ACF queries.

### From the command-line

```
$ apoc populate.acf
```

Execute ACF files with `apoc` like they were shell scripts or batch files.

## License (MIT)

Copyright (c) 2015 Hage Yaapa

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
