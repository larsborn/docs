---
fileID: java-tutorial
title: Tutorial Java in 10 Minutes
weight: 3955
description: 
layout: default
---
This is a short tutorial with the [Java Driver](https://github.com/arangodb/arangodb-java-driver) and ArangoDB. In less
than 10 minutes you can learn how to use ArangoDB Java driver in Maven and Gradle projects.

Check out this [repository](https://github.com/arangodb/arangodb-java-driver-quickstart) for the tutorial code.

## Project configuration

To use the ArangoDB Java driver you need to import 2 libraries into your project:
- [arangodb-java-driver](https://github.com/arangodb/arangodb-java-driver): the driver itself
- [jackson-dataformat-velocypack](https://github.com/arangodb/jackson-dataformat-velocypack): a data format backend 
  implementation enabling VelocyPack support for [Jackson Databind API](https://github.com/FasterXML/jackson-databind).

In a Maven project, you need to add the following dependencies to `pom.xml`:

{{< tabs >}}
{{% tab name="xml" %}}
```xml
<dependencies>
    <dependency>
        <groupId>com.arangodb</groupId>
        <artifactId>arangodb-java-driver</artifactId>
        <version>...</version>
    </dependency>
    <dependency>
        <groupId>com.arangodb</groupId>
        <artifactId>jackson-dataformat-velocypack</artifactId>
        <version>...</version>
    </dependency>
</dependencies>
```
{{% /tab %}}
{{< /tabs >}}

In a Gradle project, add the following to `build.gradle`:

{{< tabs >}}
{{% tab name="groovy" %}}
```groovy
dependencies {
    implementation 'com.arangodb:arangodb-java-driver:...'
    implementation 'com.arangodb:jackson-dataformat-velocypack:...'
}
```
{{% /tab %}}
{{< /tabs >}}


## Connection

First, you need to configure and open a connection to start ArangoDB.

{{< tabs >}}
{{% tab name="java" %}}
```java
ArangoDB arangoDB = new ArangoDB.Builder()
    .serializer(new ArangoJack())
    .build();
```
{{% /tab %}}
{{< /tabs >}}

{{% hints/tip %}}
The default connection is to 127.0.0.1:8529.
{{% /hints/tip %}}

## Creating a database

Then you can create a new database:

{{< tabs >}}
{{% tab name="java" %}}
```java
ArangoDatabase db = arangoDB.db(DbName.of("mydb"));
System.out.println("Creating database...");
db.create();
```
{{% /tab %}}
{{< /tabs >}}


## Creating a collection

Now you can create your first collection:

{{< tabs >}}
{{% tab name="java" %}}
```java
ArangoCollection collection = db.collection("firstCollection");
System.out.println("Creating collection...");
collection.create();
```
{{% /tab %}}
{{< /tabs >}}


## Creating a document

After you've created the collection, you can add documents to it. Any object can be added as a document to the database and be retrieved from
the database as an object.

In this example, the {{< tabs >}}
{{% tab name="BaseDocument" %}}
```BaseDocument```
{{% /tab %}}
{{< /tabs >}} class provided with the driver is used. The attributes of the document are stored in a
map as key<String>/value<Object> pair:

{{< tabs >}}
{{% tab name="java" %}}
```java
String key = "myKey";
BaseDocument doc = new BaseDocument(key);
doc.addAttribute("a", "Foo");
doc.addAttribute("b", 42);
System.out.println("Inserting document...");
collection.insertDocument(doc);
```
{{% /tab %}}
{{< /tabs >}}

Note that:

- the document key is passed to the `BaseDocument` constructor
- `addAttribute()` puts a new key/value pair into the document
- each attribute is stored as a single key/value pair in the document root


## Read a document

To read the created document:

{{< tabs >}}
{{% tab name="java" %}}
```java
System.out.println("Reading document...");
BaseDocument readDocument = collection.getDocument(key, BaseDocument.class);
System.out.println("Key: " + readDocument.getKey());
System.out.println("Attribute a: " + readDocument.getAttribute("a"));
System.out.println("Attribute b: " + readDocument.getAttribute("b"));
```
{{% /tab %}}
{{< /tabs >}}

After executing this program the console output should be:

{{< tabs >}}
{{% tab name="text" %}}
{{< tabs >}}
{{% tab name="text" %}}
```text
Key: myKey
Attribute a: Foo
Attribute b: 42
```
{{% /tab %}}
{{< /tabs >}}
{{% /tab %}}
{{< /tabs >}}

Note that `getDocument()` reads the stored document data and deserializes it into the given class (`BaseDocument`).


## Read a document as Jackson JsonNode

You can also read a document as a Jackson `JsonNode`:

{{< tabs >}}
{{% tab name="java" %}}
```java
System.out.println("Reading document as Jackson JsonNode...");
JsonNode jsonNode = collection.getDocument(key, ObjectNode.class);
System.out.println("Key: " + jsonNode.get("_key").textValue());
System.out.println("Attribute a: " + jsonNode.get("a").textValue());
System.out.println("Attribute b: " + jsonNode.get("b").intValue());
```
{{% /tab %}}
{{< /tabs >}}

After executing this program the console output should be:

{{< tabs >}}
{{% tab name="text" %}}
{{< tabs >}}
{{% tab name="text" %}}
```text
Key: myKey
Attribute a: Foo
Attribute b: 42
```
{{% /tab %}}
{{< /tabs >}}
{{% /tab %}}
{{< /tabs >}}

Please note that `getDocument()` returns the stored document as an instance of `com.fasterxml.jackson.databind.JsonNode`.


## Update a document

{{< tabs >}}
{{% tab name="java" %}}
```java
doc.addAttribute("c", "Bar");
System.out.println("Updating document ...");
collection.updateDocument(key, doc);
```
{{% /tab %}}
{{< /tabs >}}


## Read the document again

To read the document again, run the following:

{{< tabs >}}
{{% tab name="java" %}}
```java
System.out.println("Reading updated document ...");
BaseDocument updatedDocument = collection.getDocument(key, BaseDocument.class);
System.out.println("Key: " + updatedDocument.getKey());
System.out.println("Attribute a: " + updatedDocument.getAttribute("a"));
System.out.println("Attribute b: " + updatedDocument.getAttribute("b"));
System.out.println("Attribute c: " + updatedDocument.getAttribute("c"));
```
{{% /tab %}}
{{< /tabs >}}

After executing this program the console output should look like this:

{{< tabs >}}
{{% tab name="text" %}}
```text
Key: myKey
Attribute a: Foo
Attribute b: 42
Attribute c: Bar
```
{{% /tab %}}
{{< /tabs >}}


## Delete a document

To delete a document, run the following:

{{< tabs >}}
{{% tab name="java" %}}
```java
System.out.println("Deleting document ...");
collection.deleteDocument(key);
```
{{% /tab %}}
{{< /tabs >}}


## Execute AQL queries

First you need to create some documents with the {{< tabs >}}
{{% tab name="Homer" %}}
{{< tabs >}}
{{% tab name="Homer" %}}
```Homer```
{{% /tab %}}
{{< /tabs >}}
{{% /tab %}}
{{< /tabs >}} name in the {{< tabs >}}
{{% tab name="firstCollection" %}}
{{< tabs >}}
{{% tab name="firstCollection" %}}
```firstCollection```
{{% /tab %}}
{{< /tabs >}}
{{% /tab %}}
{{< /tabs >}} collection:

{{< tabs >}}
{{% tab name="java" %}}
```java
for (int i = 0; i < 10; i++) {
    BaseDocument value = new BaseDocument(String.valueOf(i));
    value.addAttribute("name", "Homer");
    collection.insertDocument(value);
}
```
{{% /tab %}}
{{< /tabs >}}

To get all documents with the {{< tabs >}}
{{% tab name="Homer" %}}
{{< tabs >}}
{{% tab name="Homer" %}}
```Homer```
{{% /tab %}}
{{< /tabs >}}
{{% /tab %}}
{{< /tabs >}} name in the {{< tabs >}}
{{% tab name="firstCollection" %}}
{{< tabs >}}
{{% tab name="firstCollection" %}}
```firstCollection```
{{% /tab %}}
{{< /tabs >}}
{{% /tab %}}
{{< /tabs >}} collection and iterate over the result:

{{< tabs >}}
{{% tab name="java" %}}
```java
String query = "FOR t IN firstCollection FILTER t.name == @name RETURN t";
Map<String, Object> bindVars = Collections.singletonMap("name", "Homer");
System.out.println("Executing read query ...");
ArangoCursor<BaseDocument> cursor = db.query(query, bindVars, null, BaseDocument.class);
cursor.forEach(aDocument -> System.out.println("Key: " + aDocument.getKey()));
```
{{% /tab %}}
{{< /tabs >}}

After executing this program the console output should look something like this:

{{< tabs >}}
{{% tab name="text" %}}
```text
Key: 1
Key: 0
Key: 5
Key: 3
Key: 4
Key: 9
Key: 2
Key: 7
Key: 8
Key: 6
```
{{% /tab %}}
{{< /tabs >}}

Please note that:

- the AQL query uses the `@name` placeholder which has to be bound to a value
- `query()` executes the defined query and returns `ArangoCursor` with the given class (here: `BaseDocument`)
- the order is not guaranteed


## Delete a document with AQL

To delete the document created before, execute the following:

{{< tabs >}}
{{% tab name="java" %}}
```java
String query = "FOR t IN firstCollection FILTER t.name == @name "
    + "REMOVE t IN firstCollection LET removed = OLD RETURN removed";
Map<String, Object> bindVars = Collections.singletonMap("name", "Homer");
System.out.println("Executing delete query ...");
ArangoCursor<BaseDocument> cursor = db.query(query, bindVars, null, BaseDocument.class);
cursor.forEach(aDocument -> System.out.println("Removed document " + aDocument.getKey()));
```
{{% /tab %}}
{{< /tabs >}}

After executing this program the console output should look something like this:

{{< tabs >}}
{{% tab name="text" %}}
```text
Removed document: 1
Removed document: 0
Removed document: 5
Removed document: 3
Removed document: 4
Removed document: 9
Removed document: 2
Removed document: 7
Removed document: 8
Removed document: 6
```
{{% /tab %}}
{{< /tabs >}}

## Learn more

- Have a look at the [AQL documentation](../../about-arangodb/) to learn more about the ArangoDB query language.
- Learn more about [databases](../../modeling-data/)
- Read more about [collections](../../getting-started/data-modeling/collections/).
- Explore [documents](../../getting-started/data-modeling/documents/) in our documentation.