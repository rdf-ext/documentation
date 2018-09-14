# RDF-EXT documentation

## What is rdf-ext

Rdf-ext is a JavaScript library for working with RDF. RDF is a graph data model by the [W3C](https://www.w3.org/standards/semanticweb/) and is widely implemented. If you are new to RDF have a look at our [Linked Data Training](https://github.com/zazuko/linked-data-training/blob/master/Resources.md), we assume some basic knowledge of RDF and its data model in this documentation.

## What is the spec, where is the spec, why is there a spec

For many years there were multiple RDF libraries and interfaces available in the JavaScript world. In 2013 the [RDF JavaScript Libraries Community Group](https://www.w3.org/community/rdfjs/) was initiated. A result of this group is the [RDFJS specification](http://rdf.js.org/), a low-level interface for working with RDF and Linked Data in ECMAScript platforms like Web browsers and Node.js.

All libraries described in this document are based on the RDFJS interface and read/write data structures defined in this specification.

## What are the main packages, what do they do

The RDFJS interface is a low-level interface specification. Next to the core-module `rdf-ext` we provide several other module categories that use the same interface:

* **Parsers & serializers**: RDF is a data model and unlike other data models it is not bound to a particular serialization. There are many different formats available, some are plaintext based, others are JSON or XML based or even a binary file. In general one can convert each serialization to another one without any loss of data. Parser and serialiser modules implement the specification of each format and transform it from/to an RDFJS interface structure. Always use these parsers and serializers and avoid doing this in your own code.

* **Stores**: Stores provide a way to persist RDFJS interface structures. This can be a simple in-memory store or a persistent backend in an RDF graph database (for example a SPARQL endpoint). Each store implements the same abstract interface, if you start with an in-memory store it is easy to switch to a more persistent layer by simply choosing another store implementation. No other code changes are necessary.

In many cases developers want to work with simpler interfaces to reduce code complexity and focus on solving problems using RDF. For that reason we introduce modules that are built on top of `rdf-ext`:

* **Dataset**: Dataset is a (work in progress) specification of a high-level interface on top of RDFJS interface specification. It provides additional functions that facilitate interacting with RDF data. Unless you have a good reason to do so, *this is the library you want to start working with.*

* **Helpers**: Helpers provide abstractions for common tasks in the RDF programming world exposed as simple interfaces. While you could code everything helpers do on your own, you save quite some lines of code for particular tasks.

## Basics

Note that we only explain our libraries in this document. If you don't understand a particular RDF concept, please follow on the links provided within the text to learn more about it.

### Create a triple/quad

```javascript
const rdf = require('rdf-ext')

let subject = rdf.namedNode('http://example.org/subject') 
let predicate = rdf.namedNode('http://example.org/predicate')
let object = rdf.literal('object')

let quad = rdf.quad(subject, predicate, object)

// log the triples to console with toString()
// note that this is N-Triples/N-Quads serialization in rdf-ext
console.log(quad.toString())
```

In this example we start with an `rdf` object from the `rdf-ext` package. We then create a [subject, a predicate and an object](https://www.w3.org/TR/rdf11-primer/#section-triple). As always with RDF, subject and predicate need to be [IRIs](https://www.w3.org/TR/rdf11-primer/#section-IRI). The object can either be an IRI or a [literal](https://www.w3.org/TR/rdf11-primer/#section-literal), as in our example.

We then assign the triple to a quad object. This represents the [graph-concept in RDF 1.1](https://www.w3.org/TR/rdf11-primer/#section-multiple-graphs) and can be omitted. In this case the triple is simply added to the so-called "[default graph](http://rdf.js.org/#dom-quad-graph)".

In the last line we use `quad.toString()` to log the triple/quad to the console. In rdf-ext for a triple this is always [N-Triples](https://www.w3.org/TR/rdf11-primer/#section-n-triples) like syntax with one triple per line and a dot at the end. In case of a quad, it will be [N-Quads](https://www.w3.org/TR/n-quads/) syntax.

If you want to create a [blank node](https://www.w3.org/TR/rdf11-primer/#section-blank-node) instead of a named-node, simply use the `blankNode` function:

```javascript
let bnode = rdf.blankNode()
```

You obviously have to use this identifier both as subject and object if you want to create a proper graph structure.

### Work with triples

Most of the time we want to work with more than one triples so we need to have some kind of container where we can add multiple triples. While one could do that manually by using arrays, a more suitable structure is `dataset`. By using this structure you get several useful functions, for example for matching triples. Also you do not have to take care of duplicate detection so you won't have the same triple twice in your data.

Let us describe a person, in particular the [Sheldon Cooper](https://en.wikipedia.org/wiki/Sheldon_Cooper) from the TV series The Big Bang Theory. We maintain triples about him in a specific GitHub repository, see [here](https://github.com/zazuko/tbbt-ld/blob/master/data/person/sheldon-cooper.ttl). We will mainly use [schema.org/Person](https://schema.org/Person) as vocabulary in this example.

```javascript
const rdf = require('rdf-ext')

// create a new dataset using the rdf-ext factory
let dataset = rdf.dataset()
let bnode = rdf.blankNode()

dataset.add(rdf.quad(rdf.namedNode('http://example.org/sheldon'), rdf.namedNode('http://schema.org/givenName'), rdf.literal('Sheldon')))
dataset.add(rdf.quad(rdf.namedNode('http://example.org/sheldon'), rdf.namedNode('http://schema.org/familyName'), rdf.literal('Cooper')))
dataset.add(rdf.quad(rdf.namedNode('http://example.org/sheldon'), rdf.namedNode('http://schema.org/address'), bnode))
dataset.add(rdf.quad(bnode, rdf.namedNode('http://schema.org/addressCountry'), rdf.literal('US')))
dataset.add(rdf.quad(bnode, rdf.namedNode('http://schema.org/addressLocality'), rdf.literal('Pasadena')))
dataset.add(rdf.quad(bnode, rdf.namedNode('http://schema.org/addressRegion'), rdf.literal('CA')))
dataset.add(rdf.quad(bnode, rdf.namedNode('http://schema.org/postalCode'), rdf.literal('91104')))
dataset.add(rdf.quad(bnode, rdf.namedNode('http://schema.org/streetAddress'), rdf.literal('2311 North Los Robles Avenue, Aparment 4A')))

// log the triples to console with toString()
// note that this is N-Triples serialization in rdf-ext
console.log(dataset.toString())

```

This was fun but not what we do all the time in code. It is way more common to load Triples from any RDF serialization and do something with it.

### Parse triples from a file

By installing the npm package `tbbt-ld` you will get several characters of The Big Bang Theory described in [Turtle](https://www.w3.org/TR/turtle/) format. By using the parser package `rdf-parser-n3` we can parse it into a dataset structure. Note that the [N3 parser](https://github.com/rdfjs/N3.js) (used by `rdf-parser-n3`) can parse [Turtle](https://www.w3.org/TR/turtle/), [TriG](https://www.w3.org/TR/trig/), [N-Triples](https://www.w3.org/TR/n-triples/), [N-Quads](https://www.w3.org/TR/n-quads/), and [Notation3 (N3)](https://www.w3.org/TeamSubmission/n3/). 

```javascript
const fs = require('fs')
const rdf = require('rdf-ext')
const N3Parser = require('rdf-parser-n3')

// create N3 parser instance
let parser = new N3Parser({factory: rdf})

// Read a Turtle file and stream it to the parser
let quadStream = parser.import(fs.createReadStream('./node_modules/tbbt-ld/data/person/sheldon-cooper.ttl'))

// create a new dataset and import the quad stream into it (reverse pipe) with Promise API
rdf.dataset().import(quadStream).then((dataset) => {
  // loop over all quads an write them to the console
  dataset.forEach((quad) => {
    console.log(quad.toString())
  })
})

```

 



## What are the other packages, what do they do

### [`rdf-ext`](https://github.com/rdf-ext/rdf-ext)

1. What is this package, when should it be used?
2. With what other packages does it play well? What are the related packages I should check out?
3. What are the main objects or classes, methods and their signature
4. Examples

### [`rdf-dataset-simple`](https://github.com/rdf-ext/rdf-dataset-simple)

1. What is this package, when should it be used?
2. With what other packages does it play well? What are the related packages I should check out?
3. What are the main objects or classes, methods and their signature
4. Examples

---

After that: Same thing but for the rest of the packages.

### [`parser-jsonld`](https://github.com/rdfjs/parser-jsonld)

### [`parser-n3`](https://github.com/rdfjs/parser-n3)

### [`comunica-browser`](https://github.com/rdfjs/comunica-browser)

### [`N3.js`](https://github.com/rdfjs/N3.js)

### [`data-model`](https://github.com/rdfjs/data-model)

### [`serializer-jsonld`](https://github.com/rdfjs/serializer-jsonld)

### [`serializer-ntriples`](https://github.com/rdfjs/serializer-ntriples)

### [`sink`](https://github.com/rdfjs/sink)

### [`to-ntriples`](https://github.com/rdfjs/to-ntriples)
