---
layout: ballerina-left-nav-pages-swanlake
title: Calling Java Code from Ballerina
description: See how Ballerina offers a straightforward way to call existing Java code from Ballerina and a Java API to call Ballerina code from Java.
keywords: ballerina, programming language, java api, interoperability
permalink: /swan-lake/learn/calling-java-code-from-ballerina/
active: calling-java-code-from-ballerina/
redirect_from:
  - /swan-lake/learn/how-to-use-java-interoperability
  - /swan-lake/learn/how-to-use-java-interoperability/
  - /swan-lake/learn/how-to-call-java-code-from-ballerina/
  - /swan-lake/learn/how-to-call-java-code-from-ballerina
  - /swan-lake/learn/calling-java-code-from-ballerina
---

# Calling Java Code from Ballerina

Ballerina offers a straightforward way to call the existing Java code from Ballerina and also provides a Java API to call Ballerina code from Java.  Although Ballerina is not designed to be a JVM language, the current implementation, which targets the JVM, aka jBallerina, provides Java interoperability by adhering to the Ballerina language semantics. 

- [Ballerina Bindings to Java Code](#ballerina-bindings-to-java-code)
- [The Need to Call Java from Ballerina](#the-need-to-call-java-from-ballerina)
- [Writing Ballerina Bindings](#writing-ballerina-bindings)
- [Using the SnakeYAML Java Library in Ballerina](#using-the-snakeyaml-java-library-in-ballerina)
    - [Step 1 - Writing the Java Code](#step-1---writing-the-java-code)
    - [Step 2 - Setting Up the Ballerina Project](#step-2---setting-up-the-ballerina-project)
        - [Creating a Ballerina Project](#creating-a-ballerina-project)
        - [Adding a Ballerina Module to Your Project](#adding-a-ballerina-module-to-your-project)
        - [Adding a Sample YAML File](#adding-a-sample-yaml-file)
        - [Verifying the Project](#verifying-the-project)
    - [Step 3 - Generating the Ballerina Bindings](#step-3---generating-the-ballerina-bindings)
      - [Copying the SnakeYAML Library to Your Project](#copying-the-snakeyaml-library-to-your-project)
      - [Adding the SnakeYAML Library to the Ballerina TOML File](#adding-the-snakeyaml-library-to-the-ballerina-toml-file)
      - [Generating the Ballerina Bindings](#generating-the-ballerina-bindings)
    - [Step 4 - Writing the Ballerina Code](#step-4---writing-the-ballerina-code)
        - [Creating the `FileInputStream`](#creating-the-fileinputstream)
        - [Creating the SnakeYAML Entry Point](#creating-the-snakeyaml-entry-point)
        - [Loading the YAML Document](#loading-the-yaml-document)
        - [Printing the Returned Map Instance](#printing-the-returned-map-instance)
        - [Completing the Code](#completing-the-code)
- [The Bindgen Tool](#the-bindgen-tool)
- [The `bindgen` Command](#the-bindgen-command)
    - [Generated Bridge Code](#generated-bridge-code)
    - [Mapping Java Code with Ballerina](#mapping-java-code-with-ballerina)
        - [Java Classes](#java-classes)
        - [Constructors](#constructors)
        - [Methods](#methods)
        - [Fields](#fields)
        - [External Interop Functions](#external-interop-functions)
        - [Dependency Objects](#dependency-objects)
        - [Ballerina Object](#ballerina-object)
    - [Type Mappings between Java and Ballerina](#type-mappings-between-java-and-ballerina)
- [Packaging Java Libraries with Ballerina Programs](#packaging-java-libraries-with-ballerina-programs)
    - [Ballerina FFI](#ballerina-ffi)
        - [The External Function Body](#the-external-function-body)
        - [The Handle Type](#the-handle-type)
- [Calling Java Programs from Ballerina](#calling-java-programs-from-ballerina)
    - [Instantiating Java Classes](#instantiating-java-classes)
        - [Dealing with Overloaded Constructors](#dealing-with-overloaded-constructors)
        - [The `paramTypes` Field](#the-paramtypes-field)
    - [Calling Java Methods](#calling-java-methods)
        - [Calling Static Methods](#calling-static-methods)
        - [Calling Instance Methods](#calling-instance-methods)
        - [Mapping Java Classes into Ballerina Objects](#mapping-java-classes-into-ballerina-objects)
        - [Calling Overloaded Java Methods](#calling-overloaded-java-methods)
    - [Java Exceptions as Ballerina Errors](#java-exceptions-as-ballerina-errors)
        - [Java Checked Exceptions](#java-checked-exceptions)
        - [Mapping a Java Exception to a Ballerina Error Value](#mapping-a-java-exception-to-a-ballerina-error-value)
    - [Null Safety](#null-safety)
    - [Mapping Java Types to Ballerina Types and Vice Versa](#mapping-java-types-to-ballerina-types-and-vice-versa)
        - [Mapping Java Types to Ballerina Types](#mapping-java-types-to-ballerina-types)
        - [Mapping Ballerina Types to Java Types](#mapping-ballerina-types-to-java-types)
        - [Access or Mutate Java Fields](#access-or-mutate-java-fields)

### Ballerina Bindings to Java Code
Your task is to write Ballerina code (Ballerina bindings) that lets you call the corresponding Java API as illustrated in the below diagram. 

<img src="/swan-lake/learn/images/interoperability-diagram.png" alt="Ballerina bindings to Java code" width="300" height="350">

This guide teaches you how to write those bindings manually as well as how to generate those bindings automatically but first, let's look at why you want to call Java from Ballerina. 

### The Need to Call Java from Ballerina 
- Ballerina is a relatively new language. Therefore, you may experience a shortage of libraries in [Ballerina Central](https://central.ballerina.io/). In such situations, as a workaround, you can use an existing Java library.
- You are already familiar with a stable Java API that you would like to use in your Ballerina project. 
- You want to take advantage of the strengths of Ballerina but you don’t want to reinvest in the libraries that you or your company have written already. 

There may be other reasons but these are great motivations to use Ballerina bindings. 

### Writing Ballerina Bindings
Writing Ballerina bindings manually is a tedious task. You’ll soon see why. Therefore, we’ve developed a tool called `bindgen` that can generate Ballerina bindings for given Java APIs. The [first section](#how-to-use-the-snakeyaml-java-library-in-ballerina) of this guide shows you how to use it. The [second section](#the-bindgen-tool) is a reference guide to the tool. 

The [third section](#packaging-java-libraries-with-ballerina-programs) explains how to package Java libraries (JAR files) with Ballerina programs. This section is useful because whenever you generate bindings for a Java library, you need to package this Java library and its transitive dependencies to produce a self-contained executable program. 

The [fourth](#ballerina-ffi) and [fifth](#calling-java-code-from-ballerina) sections explain how to write these bindings manually. It is also a useful section for those who want to understand the inner workings of calling Java from Ballerina and for those who want to customize the bindings generated by the `bindgen` tool. 

## Using the SnakeYAML Java Library in Ballerina
SnakeYAML is a YAML parser for Java. In this section, we'll learn how to use this library to parse a YAML document using Ballerina. 

We'll develop a Ballerina program that parses the given YAML file and writes the content to the standard out.

Let's get started.  

### Step 1 - Writing the Java Code
We recommend you to always start by writing the Java code. It gives you an idea of the set of Java classes required to implement your logic. Then, we can use the `bindgen` tool to generate Ballerina bindings for those classes. 

The following Java code uses the SnakeYAML API to parse the given YAML file. Note that this is not the most idiomatic way of writing the Java code for this scenario. 

```java
import org.yaml.snakeyaml.Yaml;

import java.io.FileInputStream;
import java.io.InputStream;
import java.util.Map;

public class SnakeYamlSample {

    public static void main(String... a) {
	String filename = a[0];
	try (InputStream inputStream = new FileInputStream(filename)) {
		Yaml yaml = new Yaml(); 
		Map<String, Object> obj = yaml.load(inputStream);
		System.out.println(obj);
	} catch (Exception e) {
                System.err.println("The file '" + filename + "' cannot be loaded. Reason: " + e.getMessage());;
	}
    }
}
```

Here, we've used four Java classes. 
- `org.yaml.snakeyaml.Yaml`
- `java.io.FileInputStream`
- `java.io.InputStream`
- `java.util.Map`

You can see them in the imported class list. We encourage you to generate Ballerina bindings for these four classes as a start.  

Now, we'll create an environment for our Ballerina program. 

### Step 2 - Setting Up the Ballerina Project
This section assumes that you have already read [How to Structure Ballerina Code](https://ballerina.io/swan-lake//learn/how-to-structure-ballerina-code/). 

#### Creating a Ballerina Project
```sh
> ballerina new yaml-project
Created new Ballerina project at yaml-project

Next:
    Move into the project directory and use `ballerina add <module-name>` to
    add a new Ballerina module.
```
#### Adding a Ballerina Module to Your Project
```sh
> ballerina add yamlparser
Added new ballerina module at 'src/yamlparser’
```
#### Adding a Sample YAML File 
Copy the below content to a file named invoice.yml in the project root directory.
```yaml
invoice: 34843
date   : 2001-01-23
bill-to: &id001
   given  : Chris
   family : Dumars
   address:
       lines: |
           458 Walkman Dr.
           Suite #292
       city    : Royal Oak
       state   : MI
       postal  : 48046
ship-to: *id001
product:
   - sku         : BL394D
     quantity    : 4
     description : Basketball
     price       : 450.00
   - sku         : BL4438H
     quantity    :
     description : Super Hoop
     price       : 2392.00
tax  : 251.42
total: 4443.52
comments: >
   Late afternoon is best.
   Backup contact is Nancy
   Billsmer @ 338-4338.\
```

#### Verifying the Project
```sh
> ballerina build yamlparser 
Compiling source
	sameera/yamlparser:0.1.0

Creating balos
	target/balo/yamlparser-2020r1-any-0.1.0.balo
... 
...

Generating executables
	target/bin/yamlparser.jar
```
```sh
> ballerina run target/bin/yamlparser.jar
Hello World!
```
Great! You are all set for the next step. 

### Step 3 - Generating the Ballerina Bindings 
In this step, we'll use the `bindgen` tool to generate Ballerina bindings for those four classes that we talked about in Step 1. If you want more information about the tool, you can refer [The `bindgen` tool](#the-bindgen-tool).

#### Copying the SnakeYAML Library to Your Project
Download the latest version of the SnakeYAML library and copy it to the project. We need to copy only the SnakeYAML library but for most cases, you may need to copy more than one JAR file. Make sure that you add all the direct and transitive dependencies of them. 

Create a directory in your project root to store all the Java libraries. 
```sh
> mkdir javalibs
> cp <path-to-snakeyaml-lib>/snakeyaml-1.25.jar javalibs
```

#### Adding the SnakeYAML Library to the Ballerina TOML File
Copy and paste the following TOML snippet to the `Ballerina.toml` file in your project’s root directory. This step ensures that the SnakeYAML library is always packaged with the stand-alone executable JAR generated for your Ballerina program. Refer to the [Packaging Java libraries with Ballerina programs](#packaging-java-libraries-with-ballerina-programs) for more details. 

```toml
[platform]
target = "java8"

   [[platform.libraries]]
   path = "./javalibs/snakeyaml-1.25.jar"
   modules = ["yamlparser"]
```

#### Generating the Ballerina Bindings
```sh
> ballerina bindgen -cp ./javalibs/snakeyaml-1.25.jar -o src/yamlparser
 org.yaml.snakeyaml.Yaml java.io.FileInputStream java.io.InputStream java.util.Map

Generating bindings for: 
	java.util.Map
	java.io.FileInputStream
	org.yaml.snakeyaml.Yaml
	java.io.InputStream

Generating dependency bindings for: 
	org.yaml.snakeyaml.introspector.BeanAccess
	java.util.function.BiFunction
	org.yaml.snakeyaml.constructor.BaseConstructor
	java.util.function.Function
	... 
	... 
```
- The `-cp` option specifies the list of Java libraries required to generate bindings. 
- The `-o` option specifies the output directory to which the generated bindings are stored. In this case, we instruct the tool to store bindings inside the `yamlparser` module. 
- The argument list specifies the Java class names. 

The `bindgen` tool generate bindings for 
- The specified Java classes 
- The Java classes exposed in the public APIs of all the specified classes. 


Before we move onto the next step, let’s verify the generated code. 
```sh
> ballerina build yamlparser 
... 
...

Generating executables
	target/bin/yamlparser.jar

> ballerina run target/bin/yamlparser.jar
Hello World!
```

### Step 4 - Writing the Ballerina Code
>**Note:** The `bindgen` tool is still experimental. We are in the process of improving the generated code.

Now, we’ll use the generated bindings and write the Ballerina code, which uses the SnakeYAML library. Here is the Java code. Let’s develop the corresponding Ballerina code step by step. 
```java
public class SnakeYamlSample {

    public static void main(String... a) {
	String filename = a[0];
	try (InputStream inputStream = new FileInputStream(filename)) {
		Yaml yaml = new Yaml(); 
		Map<String, Object> obj = yaml.load(inputStream);
		System.out.println(obj);
	} catch (Exception e) {
                System.err.println("The file '" + filename + "' cannot be loaded. Reason: " + e.getMessage());;
	}
    }
}
```

### Creating the `FileInputStream`
Our goal here is to create a new `java.io.FileInputStream` instance from the filename. In step 3, we generated bindings for the required Java classes. The following is the code snippet that does the job. 

```ballerina
FileInputStream | error fileInputStream = newFileInputStream3(filename);
```

Here, `FileInputStream` is the Ballerina object generated for the `java.io.FileInputStream` class. 
- You can find functions that start with `newFileInputStream` in the generated code. Each such function creates a new `java.io.FileInputStream` instance. Ballerina does not support function overloading. Therefore, the bindgen tool generates a separate Ballerina function for each overloaded method or constructor. We will improve the function names of the generated bindings in a future release. 
- All the methods in the `java.io.FileInputStream` class are mapped to methods in the generated Ballerina object.  

Next, we’ll handle the error using a type guard.
```ballerina

if fileInputStream is error {
	// The type of fileInputStream is error within this block
       io:println("The file '" + filename + "' cannot be loaded. Reason: " + fileInputStream.reason());
} else {
	// The type of fileInputStream is FileInputStream within this block
}
```
### Creating the SnakeYAML Entry Point
The `org.yaml.snakeyaml.Yaml` Class is the entry point to the SnakeYAML API.  The corresponding generated Ballerina object is `Yaml`. The `newYaml5()`  function is mapped to the default constructor of the Java class.   
```ballerina
Yaml yaml = newYaml5();
```
###  Loading the YAML Document
We'll be using the `org.yaml.snakeyaml.Yaml.load(InputStream is)` method to get a Java Map instance from the given InputStream. 
```ballerina

InputStream inputStream = new (fileInputStream.jObj);
Object mapObj = yaml.load2(inputStream);
```
The generated code does not handle Java subtyping at the moment. Therefore, `InputStream inputStream = newFileInputStream3(filename) `will not compile. We will improve this in a future release. As a workaround, you can create a new `java.io.InputStream` as above. 

The `org.yaml.snakeyaml.Yaml.load(InputStream is)` is a generic method. The bindgen tool does not support Java generics at the moment. That is why the corresponding Ballerina method returns an Object.  

###  Printing the Returned Map Instance
You can print the content of the Map instance in the the standard out as follows. 
```ballerina
io:println(mapObj);
```
### Completing the Code 
Here, is the complete code. You can replace the contents in `src/yamlparser/main.bal` with the following code.
```ballerina
import ballerina/io;
 
public function main(string... args) returns error? {
   string filename = args[0];
   FileInputStream | error fileInputStream = newFileInputStream3(filename);
   if fileInputStream is error {
       io:println("The file '" + filename + "' cannot be loaded. Reason: " + fileInputStream.reason());
   } else {
       Yaml yaml = newYaml5();
       InputStream inputStream = new (fileInputStream.jObj);
       Object mapObj = yaml.load2(inputStream);
       io:println(mapObj);
   }
}
```

Let's build and run this code. 
```sh
> ballerina build yamlparser 
Compiling source
	sameera/yamlparser:0.1.0

Creating balos
	target/balo/yamlparser-2020r1-any-0.1.0.balo
... 
...

Generating executables
	target/bin/yamlparser.jar
```

Now, we need to pass the YAML file name as the first argument. 
```sh
> ballerina run target/bin/yamlparser.jar invoice.yaml
{invoice=34843, date=Mon Jan 22 16:00:00 PST 2001, bill-to={given=Chris, family=Dumars, address={lines=458 Walkman Dr.
Suite #292
, city=Royal Oak, state=MI, postal=48046}}, ship-to={given=Chris, family=Dumars, address={lines=458 Walkman Dr.
Suite #292
, city=Royal Oak, state=MI, postal=48046}}, product=[{sku=BL394D, quantity=4, description=Basketball, price=450.0}, {sku=BL4438H, quantity=null, description=Super Hoop, price=2392.0}], tax=251.42, total=4443.52, comments=Late afternoon is best. Backup contact is Nancy Billsmer @ 338-4338.\}
```
In this section, we explained how to use the `bindgen` tool to generate Ballerina bindings for Java classes and how to use those generated ones. 

The next sections provide more details on various aspects related to Java interoperability in Ballerina. 


## The `bindgen` Tool

The following subsections explain how the `bindgen` tool works.

- [The `bindgen` Command](#the-bindgen-command)
    - [Generated Bridge Code](#generated-bridge-code)
    - [Mapping Java Code with Ballerina](#mapping-java-code-with-ballerina)
        - [Java Classes](#java-classes)
        - [Constructors](#constructors)
        - [Methods](#methods)
        - [Fields](#fields)
        - [External Interop Functions](#external-interop-functions)
        - [Dependency Objects](#dependency-objects)
        - [Ballerina Object](#ballerina-object)
    - [Type Mappings between Java and Ballerina](#type-mappings-between-java-and-ballerina)
- [Packaging Java Libraries with Ballerina Programs](#packaging-java-libraries-with-ballerina-programs)
    - [Ballerina FFI](#ballerina-ffi)
        - [The External Function Body](#the-external-function-body)
        - [The Handle Type](#the-handle-type)

>**Note:** The `bindgen` tool is still experimental. We are in the process of improving the generated code.*

The `bindgen` is a CLI tool, which generates Ballerina bindings for Java classes.

### The `bindgen` Command

```sh
ballerina bindgen [(-cp|--classpath) <classpath>...]
                  [(-o|--output) <output>]
                  (<class-name>...)
```

`(-cp|--classpath) <classpath>...`
This optional parameter could be used to specify one or more comma-delimited classpaths for retrieving the required Java libraries needed by the bindgen tool execution. The classpath could be provided as comma-separated paths of JAR files or as comma-separated paths of directories containing all the relevant Java libraries. If the Ballerina bindings are to be generated from a standard Java library or from a library available inside the Ballerina SDK, then you need not specify the classpath explicitly.

`(-o|--output) <output>`
This optional parameter could be used to specify the directory path into which the Ballerina bindings should be inserted. If this path is not specified, the output will be written onto the same directory from where the command is run. You can point the path of a Ballerina module to generate the code inside a Ballerina module. 

`<class-name>...`
One or more space-separated fully-qualified Java class names for which the Ballerina bridge code is to be generated. Please note that these class names should be provided at the end of the command.

### Generated Bridge Code

When the tool is run, a `.bal` file will be created to represent each Java class. This would contain the respective Ballerina object along with the required Java interoperability mappings. These `.bal` files would reside inside sub directories representing the package structure. Apart from creating bindings for the specified Java classes, the command would also generate empty Ballerina objects for the dependent Java classes. A Java class would be considered dependent if it is used inside one of the Ballerina objects generated. A set of additional utility files will also be generated in order to support the auto-generated Ballerina bindings. The folder structure of the generated bindings will be as follows.

	<ballerina_bindings>
		├── <package-name>
			└── <class-name>.bal
			└── ...
		├── ... 
		└── <dependencies>
			├── <utils>
				├── ArrayUtils.bal
				├── Constants.bal
				└── JObject.bal
			├── <package-name>
				└── <class-name>.bal
				└── ...
			└── ...

### Mapping Java Code with Ballerina

#### Java Classes
A Java class will be mapped onto a Ballerina object. This Ballerina object will have the same name as that of the Java class. 

E.g., Generated Ballerina object of the `java.utils.ArrayDeque` class will be as follows.
```ballerina
public type ArrayDeque object {
 
	...
};
```
>**Note:** If there are multiple classes with the same name, you should change the names manually. 

#### Constructors
Constructors of Java classes will be mapped onto public functions outside the Ballerina object. These function names are comprised of the constructor name prefixed with the `new` keyword. If there exists multiple constructors, they will be suffixed with an auto increment number.

E.g., Generated constructors of the `java.utils.ArrayDeque` class will be as follows.
```ballerina
public function newArrayDeque1() returns ArrayDeque {

   ...
}
public function newArrayDeque2(int arg0) returns ArrayDeque {

   ...
}
public function newArrayDeque3(Collection arg0) returns ArrayDeque {

   ...
}
```

#### Methods
All public methods will be exposed through Ballerina bindings. Instance methods will reside inside the Ballerina object and these would take the name of the Java method. However, if there exists overloaded methods, a numeric suffix will be appended at the end of the name.

E.g., Some of the generated instance methods of the `java.utils.ArrayDeque` class will be as follows.
```ballerina
public type ArrayDeque object {
   ...
 
   public function add(Object arg0) returns boolean {
   
       ...
   }
   public function isEmpty() returns boolean {
   
       ...
   }
};
```
Static methods would reside outside the Ballerina object as public functions, which take the name of the Java method with the Java class name appended at the beginning as a prefix.
 
E.g., A generated static method of the `java.utils.UUID` class will be as follows.
```ballerina
public function UUID_randomUUID() returns UUID {
 
   ...
}
```

#### Fields
All public fields of a Java class will be exposed through Ballerina bindings in the form of getters and setters. Instance fields will have the respective getters and setters inside the Ballerina object, whereas the static fields will have getters and setters outside the Ballerina object as public functions. 

The getters and setters of an instance field will take the name of the field prefixed with a ‘get’ or ‘set’ at the beginning. For a static field, getters and setters (if the field is not final) will take the name of the field with a ‘get’ or ‘set’ prefix along with the Java class name appended at the beginning.

#### External Interop Functions
These interop functions take the fully-qualified Java method name as the function name. However, if there exists overloaded methods, a numeric suffix will be appended at the end.

#### Dependency Objects
When there are dependent Java classes present inside generated Ballerina bindings (as parameters or return types), the bindgen tool generates an empty Ballerina object to represent each one of these classes. These would represent a Java class mapping without the constructor, method, or field bindings. If one of these classes are required later, the bindgen tool could be re-run to generate the Ballerina bindings. 

>**Note:** If the complete implementation of a dependency object is generated using the bindgen tool, the existing empty Ballerina object should be deleted manually as instructed in the command output.

E.g., Generated dependency object representing `java.util.List` will be as follows.
```ballerina
public type List object {
 
   *JObject;
  
   public function __init(handle obj) {
   
       self.jObj = obj;
   }
};
```

#### Ballerina Object
A Ballerina object representing a Java class will always store the handle reference of the Java class in it’s `jObj` field. To explain the implementation further, a Ballerina object representing a Java class would always be implemented using the following `JObject` abstract class. Hence, the handle reference could be accessed when needed.

```ballerina
public type JObject abstract object {
 
   public handle jObj;
};
```

A Ballerina object could be initialized using the `__init` function or a constructor. If the first approach is used, you should pass on a handle reference of the Java class to the `__init` function. This could be used if you have already obtained a handle reference through some means other than using the constructor. If not, you could use the second approach to create a new object using a constructor.

E.g., Generated `__init` function of a Ballerina object mapping a Java class.
```ballerina
public function __init(handle obj) {

   self.jObj = obj;
}

```

#### Type Mappings between Java and Ballerina
When using the Ballerina bindings, you could use the Ballerina primitive types, Ballerina string type, and the generated Ballerina objects. They will be mapped internally onto the respective Java primitives, Java String object, and the respective handle references of the objects.

## Packaging Java Libraries with Ballerina Programs
This section assumes that you have already read the guide [How to Structure Ballerina Code](https://ballerina.io/swan-lake//learn/how-to-structure-ballerina-code/). When you compile a Ballerina program with `ballerina build <root-module>`, the compiler creates an executable JAR file and when you compile a Ballerina module with `ballerina build -c <module>`, the compiler creates a BALO file. In both cases, the Ballerina compiler produces self-contained archives. There are situations in which you need to package JAR files with these archives. The most common example would be packing the corresponding JDBC driver.

There are two kinds of Ballerina projects: 
1. Produces executable programs 
	* Contains one or more Ballerina modules and at least one of them has to be a root module.
	* A root module has a `main` method and/or one or more services.
	* Build the project with `ballerina build <root-module>` or use `ballerina build -a` if there is more than one root module.
	* The best practice is to maintain a single root module in a Ballerina project.
2. Produces Ballerina library modules
	* Contains one or more Ballerina library modules. 
	* Build the modules with `ballerina build -c <module>` or `ballerina build -c -a` to build all modules.
	* Usually, the compiled library modules are pushed to Ballerina central.

How you package JAR files with compiled archives is the same in both kinds of projects. Therefore, a sample Ballerina project, which produces an executable is used here.

Here, is a Ballerina project layout of a microservice called "order management". The module `ordermgt` - the root module - contains a RESTFul service, which exposes resource functions to create, retrieve, update, and cancel orders. The `dbutils` module offers utility functions, which use a MySQL database to store orders. 

```
ordermgt_service/
├── Ballerina.toml
└── javalibs/
    └── mysql-connector-java-<version>.jar
└── src/
    └── ordermgt/
    └── dbutils/
```    
    
The Java MySQL connector is placed inside the `javalibs` directory. You are free to store the JAR files anywhere in your file system. This example places those JAR files inside the project directory. As a best practice, maintain Java libraries inside the project.
The `Ballerina.toml` file, which marks a directory as a Ballerina project lives at the root of the project. It is also a manifest file that contains project information, dependent Ballerina module information, and platform-specific library information. Java libraries are considered as platform-specific libraries. 
Here, is how you can specify a JAR file dependency in the`Ballerina.toml`.

```toml
[platform] 
target = "java8" 

[[platform.libraries]] 
# Absolute or relative path to the JAR file
path = "<path-to-jar-file-1>" 
# A comma-separated list of Ballerina module names, which depends on this JAR
modules = ["<ballerina-module-1>"]

[[platform.libraries]] 
path = "<path-to-jar-file-2>" 
modules = ["<ballerina-module-1>","<ballerina-module-2>"]
```

Now, let’s look at the contents of the `Ballerina.toml` file in this project.
```toml
[platform] 
target = "java8" 

	[[platform.libraries]] 
	path = "./javalibs/mysql-connector-java-<version>.jar" 
	modules = ["ordermgt"]
```

If your project has only one root module, then you can attach all the JAR file dependencies to your root module as the best practise. 

If your project is a Ballerina library module project, then you should specify the JAR file dependencies in each Ballerina module if that module depends on the JAR file. 

Now, use `ballerina build ordermgt` to build an executable JAR. This command packages all JARs specified in your `Ballerina.toml` with the executable JAR file. 

## Ballerina FFI
Let's look at the list of language features that enable Ballerina developers to call foreign code written in other programming languages. E.g., while the jBallerina compiler allows you to call any `Java` code, the nBallerina compiler will allow you to call any `C` Code. 

### The External Function Body
Usually, the body or the implementation of a function is specified in the same source file. The part, which is enclosed by curly braces is called the function body.

```ballerina
function doSomething(int i) returns string {
	...
}
```

Ballerina also allows you to define a function without a function body and marks it with the `external` keyword to express that the implementation is not provided by the Ballerina source file. 

```ballerina
function doSomething(int i) returns string = external;
```

Now, let’s see how you can link this function with a foriegn function. 

```ballerina
import ballerina/java;

function doSomething(int i) returns string = @java:Method {
	name: "doSomethingInJava"
	class: "a.b.c.Foo"
} external;
```

The `@java:Method` annotation instructs the jBallerina compiler to link with the `doSomethingInJava` static method in the Java class `a.b.c.Foo`. There exists a set of annotations and other utilities available in the `ballerina/java` module to make Java interoperability work.  This guide covers most of them.

### The Handle Type
The handle type describes a reference to an externally-managed storage. These values can only be created by a Ballerina function with an external function body. Within the context of jBallerina, a `handle` type variable can refer to any Java reference type value: a Java object, an array, or the null value.

Consider the `randomUUID` method in the Java UUID class, which gives you a UUID object. This is the Java method signature.

```java
static UUID randomUUID()
```

Here, is the corresponding Ballerina function that returns a value of the handle type.

```ballerina
import ballerina/java;

function randomUUID() returns handle = @java:Method {
    name: "randomUUID",
    class: "java.util.UUID"
} external;
```

In Java, you can assign the `null` value to any variable of a reference type. Therefore, a `handle` type variable may also refer to the Java `null`.

The following section describes various aspects of Java interoperability in Ballerina. You can copy and paste following examples into a .bal file and run it using the `ballerina run <file_name.bal>` command.

## Calling Java Programs from Ballerina
The following subsections explain how to call Java code from Ballerina. 

- [Instantiating Java Classes](#instantiating-java-classes)
    - [Dealing with Overloaded Constructors](#dealing-with-overloaded-constructors)
    - [The `paramTypes` Field](#the-paramtypes-field)
- [Calling Java Methods](#calling-java-methods)
     - [Calling Static Methods](#calling-static-methods)
     - [Calling Instance Methods](#calling-instance-methods)
     - [Mapping Java Classes into Ballerina Objects](#mapping-java-classes-into-ballerina-objects)
     - [Calling Overloaded Java Methods](#calling-overloaded-java-methods)
- [Java Exceptions as Ballerina Errors](#java-exceptions-as-ballerina-errors)
    - [Java Checked Exceptions](#java-checked-exceptions)
    - [Mapping a Java Exception to a Ballerina Error Value](#mapping-a-java-exception-to-a-ballerina-error-value)
- [Null Safety](#null-safety)
- [Mapping Java Types to Ballerina Types and Vice Versa](#mapping-java-types-to-ballerina-types-and-vice-versa)
    - [Mapping Java Types to Ballerina Types](#mapping-java-types-to-ballerina-types)
    - [Mapping Ballerina Types to Java Types](#mapping-ballerina-types-to-java-types)
    - [Access or Mutate Java Fields](#access-or-mutate-java-fields)

### Instantiating Java Classes
Let's look at how you can create Java objects in a Ballerina program. The `@java:Constructor` annotation instructs the compiler to link a Ballerina function with a Java constructor.

The `ArrayDeque` class in the `java.util` package has a default constructor. The following Ballerina code creates a new `ArrayDeque` object. As you can see, the `newArrayDeque` function is linked with the default constructor. This function returns a handle value and it refers the constructed `ArrayDeque` instance.

```ballerina
import ballerina/java;

public function main() {
    handle arrayDeque = newArrayDeque(); 
}

function newArrayDeque() returns handle = @java:Constructor {
    class: "java.util.ArrayDeque"
} external;
```

You can also create a wrapper Ballerina object for Java classes as follows.

```ballerina
import ballerina/java;

public function main() {
    ArrayDeque ad = new; 
}

type ArrayDeque object {
    private handle jObj;

    function __init(){
        self.jObj = newArrayDeque();          
    }
};

function newArrayDeque() returns handle = @java:Constructor {
    class: "java.util.ArrayDeque"
} external;

```

>**Note:** that these `@java:*` annotations cannot be attached to Ballerina object methods at the moment.

#### Dealing with Overloaded Constructors
When there are two constructors with the same number of arguments available, you need to specify the exact constructor that you want to link with the Ballerina function. The `ArrayDeque` class contains three constructors and the last two are overloaded ones.

```ballerina
public ArrayDeque();
public ArrayDeque(int numElements);
public ArrayDeque(Collection<? extends E> c);
```

Here, is the updated Ballerina code.

```ballerina
import ballerina/java;

function newArrayDeque() returns handle = @java:Constructor {
    class: "java.util.ArrayDeque"
} external;

function newArrayDequeWithSize(int numElements) returns handle = @java:Constructor {
    class: "java.util.ArrayDeque",
    paramTypes: ["int"]
} external;

function newArrayDequeWithCollection(handle c) returns handle = @java:Constructor {
    class: "java.util.ArrayDeque",
    paramTypes: ["java.util.Collection"]
} external;
```

##### The `paramTypes` Field
You can use the `paramTypes` field to resolve the exact overloaded method. This field is defined as follows.

```ballerina
# The `Class` type represents a fully-qualified Java class name.
public type Class string;

# The `ArrayType` represents a Java array type. It is used to specify parameter
# types in the `Constructor` and `Method` annotations.
#
# + class - Element class of the array type
# + dimensions - Dimensions of the array type
public type ArrayType record {|
    Class class;
    byte dimensions;
|};

(Class | ArrayType)[] paramTypes?;
```

As per the above definition, `paramTypes` field takes an array of Java classes or array types. The following table contains more details.


Java Type | Description | Example
--------- | ----------- | -------
Primitive | The Java class name of a primitive type is the same as the name of the primitive type.  | The `boolean.class.getName()` expression evaluates to “boolean”. Similarly, the `int.class.getName()` expression evaluates to “int”.
Class | Fully-qualified class name | “java.lang.String”
Array | Use the `ArrayType` record defined above to specify Java array types in overloaded methods. | Method signature: `void append(boolean[] states, long l, String[][] args);` The corresponding value of the `paramField`: `paramField: [{class:”boolean”, dimensions: 1}, “long” {class:”java.lang.String”, dimensions: 2}]`

For more details, look at the following example.

```java
public Builder(Person[][] list, int index);
public Builder(Student[][] list, int index);
```

Here, is the corresponding Ballerina code.

```ballerina
import ballerina/java;

function builderWithPersonList(handle list, int index) returns handle = @java:Constructor {
    class: "a.b.c.Builder",
    paramTypes: [{class: "a.b.c.Person", dimensions:2}, "int"]
} external;

function builderWithStudentList(handle list, int index) returns handle = @java:Constructor {
    class: "a.b.c.Builder",
    paramTypes: [{class: "a.b.c.Student", dimensions:2}, "int"]
} external;
```

### Calling Java Methods
You can use the `java:@Method` annotation to link Ballerina functions with Java static and instance methods. There is a small but important difference in calling Java static methods vs calling instance methods. 

#### Calling Static Methods
Let’s first look at how to call a static method. The “java.util.UUID” class  has a static method with the `static UUID randomString()` signature. 

```ballerina
import ballerina/java;
import ballerina/io;

function randomUUID() returns handle = @java:Method {
   name: "randomUUID",
   class: "java.util.UUID"
} external;

public function main() {
   handle uuid = randomUUID();
   io:println(uuid);
}
```

The `name` field is optional here. If the Ballerina function name is the same as the Java method name, you don’t have to specify the `name` field.

```ballerina
function randomUUID() returns handle = @java:Method {
   class: "java.util.UUID"
} external;
```

#### Calling Instance Methods
Now, let’s look at how to call Java instance methods using the same `ArrayDeque` class in the `java.util` package. It can be used as a stack with its `pop` and `push` instance methods with the following method signatures. 

```java
E pop();
void push(E e);
```

Here, are the corresponding Ballerina functions that are linked to these methods. 

```ballerina
function pop(handle arrayDequeObj) returns handle = @java:Method {
   class: "java.util.ArrayDeque"
} external;

function push(handle arrayDequeObj, handle e) = @java:Method {
   class: "java.util.ArrayDeque"
} external; 
```

If you compare these functions with the Java method signatures, you would notice the additional `handle arrayDequeObj` parameter in Ballerina functions. Let’s look at a sample usage to understand the reason. 

```ballerina
public function main() {
   // Create a new instance of `ArrayDeque`.
   handle arrayDequeObj = newArrayDeque();

   // Convert a Ballerina string to a Java string.
   string str = “Ballerina”
   handle handleStr = java:fromString(str);

   push(arrayDequeObj, handleStr);
   handle e = pop(arrayDequeObj);
}
```

As you can see, you need to first construct an instance of the `ArrayDeque` class. The `arrayDequeObj` variable refers to an `ArrayDeque` object. Then, you need to pass this variable to both the `pop` and `push` functions because the corresponding Java methods are instance methods of the`ArrayDeque` class. Therefore, you need an instance of the `ArrayDeque` class in order to invoke its instance methods. You can think of the `arrayDequeObj` variable as the method receiver.

#### Mapping Java Classes into Ballerina Objects
The following pattern is useful if you want to present a clearer Ballerina API, which calls to the underneath Java code. This pattern creates wrapper Ballerina objects for each Java class that you want to expose via your API. 

Imagine that you want to design an API to manipulate a stack of string values by using the Java `ArrayDeque` utility. You can create a Ballerina object type as follows. 

```ballerina
public type StringStack object {
   private handle jObj;

   public function __init() {
       self.jObj = newArrayDeque();
   }

   public function push(string element) {
       push(self.jObj, java:fromString(element));
   }

   public function pop() returns string {
       handle handleEle = pop(self.jObj);
       // Let's talk about error handling and null satefy later in this guide
       // This example uses an empty string for now.
       return java:toString(handleEle) ?: "";

   }
};

function newArrayDeque() returns handle = @java:Constructor {
   class: "java.util.ArrayDeque"
} external;

function pop(handle receiver) returns handle = @java:Method {
   class: "java.util.ArrayDeque"
} external;

function push(handle receiver, handle element) = @java:Method {
   class: "java.util.ArrayDeque"
} external;
```

This object presents a much clearer API compared to the previous API. Here, is a sample usage of this object. 

```ballerina
public function main() {
   StringStack stack = new();
   stack.push("Ballerina");   
   string element = stack.pop();
}
```

#### Calling Overloaded Java Methods
The “Instantiate Java classes” section presented about how to deal with overloaded constructors in the. You need to use the same approach to deal with overloaded Java methods. Let’s try to call the overloaded `append` methods in the `java.lang.StringBuffer class. Here, is a subset of those methods. 

```java
StringBuffer append(boolean b);
StringBuffer append(int i);
StringBuffer append(String str);
StringBuffer append(StringBuffer sb);
StringBuffer append(char[] str);
```

Here, is the set of Ballerina functions that are linked with the above Java methods. Notice the usage of the `paramTypes` annotation field. You can find more details of this field in the “Instantiate Java classes” section. 

```ballerina
function appendBool(handle sbObj, boolean b) returns handle = @java:Method {
   name: "append",
   paramTypes: ["boolean"],
   class: "java.lang.StringBuffer"
} external;

function appendInt(handle sbObj, int i) returns handle = @java:Method {
   name: "append",
   paramTypes: ["int"],
   class: "java.lang.StringBuffer"
} external;

function appendCharArray(handle sbObj, handle str) returns handle = @java:Method {
   name: "append",
   paramTypes: [{class: "char", dimensions: 1}],
   class: "java.lang.StringBuffer"
} external;

function appendString(handle sbObj, handle str) returns handle = @java:Method {
   name: "append",
   paramTypes: ["java.lang.String"],
   class: "java.lang.StringBuffer"
} external;

function appendStringBuffer(handle sbObj, handle sb) returns handle = @java:Method {
   name: "append",
   paramTypes: ["java.lang.StringBuffer"],
   class: "java.lang.StringBuffer"
} external;
```

### Java Exceptions as Ballerina Errors
A function call in Ballerina may complete abruptly by returning an error or by raising a panic. Panics are rare in Ballerina. The best practise is to handle errors in your normal control flow. Raising a panic is similar to throwing a Java exception. The `trap` action will stop a panic and give you the control back in Ballerina and the `try-catch` statement does the same in Java. 

Errors in Ballerina belong to the built-int type `error`. The error type can be considered as a distinct type from all other types: The `error` type does not belong to the `any` type, which is the supertype of all other Ballerina types. Therefore, errors are explicit in Ballerina programs and it is almost impossible to ignore them. For more details, see BBEs. 

How do Java exceptions are mapped to Ballerina errors? 
A Java function call may complete abruptly by throwing either a checked exception or an unchecked exception. Unchecked exceptions are usually not part of the Java method signature unlike the checked exceptions.

Java interoperability layer in Ballerina handles checked exceptions differently from unchecked exceptions as explained below.
Java unchecked exceptions
If the linked Java method throws an unchecked exception, then the corresponding Ballerina function will complete abruptly by raising a panic.
 
The following example tries to pop an element out of an empty queue. The `pop` method in the `ArrayDeque` class throws an unchecked  `java.util.NoSuchElementException` exception in such cases. This exception will cause the Ballerina `pop` function  to raise a panic. 

```ballerina
import ballerina/java;

function newArrayDeque() returns handle = @java:Constructor {
   class: "java.util.ArrayDeque"
} external;

function pop(handle receiver) returns handle = @java:Method {
   class: "java.util.ArrayDeque"
} external;

public function main() {
   handle arrayDeque = newArrayDeque();
   handle element = pop(arrayDeque);
}
```

Here, is the output:

```
error: java.util.NoSuchElementException 
	at array_deque:pop(array_deque.bal:65535)
	   array_deque:main(array_deque.bal:13)
```

You can use the `trap` action to stop the propagation of the panic and to get an `error` value. 

```ballerina
public function main() {
   handle arrayDeque = newArrayDeque();
   handle | error element = trap pop(arrayDeque);
   if element is error {
       io:println(element.reason());
       io:println(element.detail());
       io:println(element.stackTrace().callStack);
   } else {
       // .....
   }
} 
```
#### Java Checked Exceptions
Let’s see how you can call a Java method that throws a checked exception. As illustrated in the following example, the corresponding Ballerina function should have the `error` type as part of it’s return type. 

The `java.util.zip.ZipFile` class is used to read entries in a ZIP file. There are many constructors in this class. Here, the constructor that takes the file name as an argument is used. 

```java
public ZipFile(String name) throws IOException
```

Since this Java constructor throws a checked exception,  the `newZipfile` Ballerina function returns `ZipFile` instances or an error. 

```ballerina
import ballerina/java;

function newZipFile(handle filename) returns handle | error = @java:Constructor {
   class: "java.util.zip.ZipFile",
   paramTypes: ["java.lang.String"]
} external;

public function main() {
   handle|error zipFile = newZipFile(java:fromString("some_file.zip"));
}
```

#### Mapping a Java Exception to a Ballerina Error Value
Now, let’s briefly look at how a Java exception is converted to a Ballerina error value at runtime. A Ballerina error value contains three components: a reason, a detail, and stack trace. 

The `reason`:
	* This is a string identifier for the category of error.
	* In this case, reason value is set to the fully-qualified Java class name of the exception. 
		* Unchecked: Class name of of the thrown unchecked exception
		* Checked: Class name of the exception that is declared in the method signature
The `detail`:
	* The `message` field is set to `e.getMessage()`.
	* The `cause` field is set to the Ballerina error that represents this Java exception’s cause.

### Null Safety
Ballerina provides strict null safety compared to Java with optional types.  The Java null reference can be assigned to any reference type. However, in Ballerina, you cannot assign the nil value to a variable unless the variable’s type is an optional type. 

As explained above, Ballerina handle values cannot be created in Ballerina code. They are created and returned by foriegn functions and a variable of the handle type refers to a Java reference value. Since Java null is also a valid reference value, this variable can refer to a Java null value.

Let’s look at an example, which deals with Java null. The following code uses the `peek` method in the `ArrayDeque` class. `Peek` retrieves but does not remove the head of the queue or returns null if the queue is empty. 

```ballerina
import ballerina/java;

function newArrayDeque() returns handle = @java:Constructor {
   class: "java.util.ArrayDeque"
} external;

function peek(handle receiver) returns handle = @java:Method {
   class: "java.util.ArrayDeque"
} external;

// Linked with the `java.lang.Object.toString()` method in Java.
function toString(handle objInstance) returns handle = @java:Method {
   class: "java.lang.Object"
} external;

public function main() {
   handle arrayDeque = newArrayDeque();
   handle element = peek(arrayDeque);
   Handle str = toString(element);
}
```

Since the queue is empty in this case, `peek` should return null i.e., `element` should refer to Java null.  The output of this program will be as follows.

```ballerina
 error: org.ballerinalang.jvm.values.ErrorValue message={ballerina}JavaNullReferenceError
	at array_deque:toString(array_deque.bal:19)
	    array_deque:main(array_deque.bal:27)
```

This is equivalent to a Java NPE. In such situations, you should check for null using the `java:isNull()` function. Here, is the modified example. 

```ballerina
public function main() {
   handle arrayDeque = newArrayDeque();
   handle element = peek(arrayDeque);   
  
   if java:isNull(element) {
       // handle this case
   } else {
       handle str = toString(element);
   }
}
```

There are situations in which you need to pass a Java null to a method or store in a data structure. In such situations, you can create a handle value that refers to a Java null as follows. 

```ballerina
handle nullValue = java:createNull();
```

### Mapping Java Types to Ballerina Types and Vice Versa
#### Mapping Java Types to Ballerina Types
The following table summarizes how Java types are mapped to corresponding Ballerina types. This is applicable when mapping a return type of a Java method to a Ballerina type. 

Java type | Ballerina type | Notes
--------- | -------------- | -----
Any reference type including “null type” | handle |
boolean | boolean |
byte | byte, int, float | widening conversion when byte -> int and byte -> float
short | int, float | widening conversion 
char  | int, float | widening conversion 
int | int, float | widening conversion 
long | int, float | widening conversion when long -> float
float | float | widening conversion 
double | float | 

#### Mapping Ballerina Types to Java Types
The following table summarizes how Ballerina types are mapped to corresponding Java types. These rules are applicable when mapping a Ballerina function argument to a Java method/constructor parameter.

Ballerina type | Java type | Notes
-------------- | --------- | -----
handle | Any reference type | As specified by the Java method/constructor signature
boolean | boolean | 
byte | byte, short, char, int, long, float, double | Widening conversion from byte -> short, char, int, long, float, double
int | byte, char, short, int, long | Narrowing conversion when int -> byte, char, short and int
float | byte, char, short, int, long, float, double | Narrowing conversion when float -> byte, char, short, int, long, float


### Access/ or Mutate Java Fields
The `@java:FieldGet` and `@java:FieldSet` annotations allow you to read and update the value of a Java static or instance field respectively. The most common use case is to read a value of a Java static constant. 

```ballerina
import ballerina/java;

public function pi() returns float = @java:FieldGet {
   name:"PI",
   class:"java/lang/Math"
} external;

public function main() {
   float r = 4;
   float l = 2 * pi() * r;
}
```

In this example, the `pi()` function returns the value of the `java.lang.Math.PI` static field. This uses the annotation field `name` to specify the name of the field. Likewise, if you want to access an instance field, you need to pass the relevant object instance as discussed in the instance methods section.

The `@java:FieldSet` annotation has the same structure as the above. 
