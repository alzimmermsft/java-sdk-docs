# Patterns & best practices

> (Topics include: Introduction to Maven BOM, Creating service clients, etc)

The Java Azure SDK is composed of many Java client libraries - one for each service - rather than a single monolithic library for all of Azure. Despite this, the Java client libraries are designed with consistency in mind, and therefore a number of patterns have been established that users of the library can benefit from as they move from one library to another.

## Library installation

All Java client libraries for Azure are made available in Maven Central in the `com.azure` group ID. This familiar home for developers means that inclusion of a library is as simple as including the appropriate Maven group ID, artifact ID, and version number for the libraries that developers wish to use. Even better, the Azure SDK team ships a Maven BOM file that enables developers to avoid the need to specify a particular version of each Java client library - instead developers may depend on a particular BOM release and have that be used to infer the version of all Java client libraries.

> **TODO** link to correct location for details on all released libraries. Also include details on including the BOM and tracking its version.

