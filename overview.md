# Use the Azure Libraries for Java

The open-source Azure libraries for Java simplify provisioning, managing, and using Azure resources from Java application code.

## Important details

* The Azure libraries are how you communicate with Azure services from Java code that you run either locally or in the cloud. (Whether you can run Java code within the scope of a particular service depends on whether that service itself currently supports Java.)
* The libraries support Java 8 and later, and are tested against both the Java 8 baseline as well as the latest Java 'long-term support' release.
* The libraries include full Java module support, which means that they are fully compliant with the requirements of a Java module and export all relevant packages for use.
* The Azure SDK for Java is composed solely of over many individual Java libraries that relate to specific Azure services. There are no other tools in the "SDK".
* There are distinct "management" and "client" libraries (sometimes referred to as "management plane" and "data plane" libraries). Each set serves different purposes and is used by different kinds of code. For more details, see the following sections later in this article:
  * [Provision and manage Azure resources with management libraries.](#provision-and-manage-azure-resources-with-management-libraries)
  * [Connect to and use Azure resources with client libraries.](#connect-to-and-use-azure-resources-with-client-libraries)
* Documentation for the libraries is found on the [Azure for Java Reference](https://docs.microsoft.com/java/api/overview/azure/), which is organized by Azure Service, or the [Java API browser](https://docs.microsoft.com/java/api/), which is organized by package name. At present, you often need to click to a number of layers to get to the classes and methods you care about. Allow us to apologize in advance for this sub-par experience. We're working to improve it!

## Other details

* The Azure libraries for Java build on top of the underlying Azure REST API, allowing you to use those APIs through familiar Java paradigms. However, you can always use the REST API directly from Java code, if desired.
* You can find the source code for the Azure libraries on https://github.com/Azure/azure-sdk-for-java. As an open-source project, contributions are welcome!
* We're currently updating the Azure libraries for Java libraries to share common cloud patterns such as authentication protocols, logging, tracing, transport protocols, buffered responses, and retries.
  * This shared functionality is contained in the [azure-core](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/core/azure-core) library.
* For details on the guidelines we apply to the libraries, see the [Java Guidelines: Introduction](https://azure.github.io/azure-sdk/java_introduction.html).

## Provision and manage Azure resources with management libraries

The SDK's management (or "management plane") libraries, all of which can be found in the `com.azure.resourcemanager` Maven group ID, help you create, provision and otherwise manage Azure resources from Java application code. All Azure services have corresponding management libraries.

With the management libraries, you can write configuration and deployment scripts to perform the same tasks that you can through the [Azure portal](https://portal.azure.com/) or the [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli).

For details on working with each management library, see the README.md file located in the library's project folder in the [SDK GitHub repository](https://github.com/Azure/azure-sdk-for-java). You can also find additional code snippets in the [reference documentation](https://docs.microsoft.com/java/api) and the [Azure Samples](https://docs.microsoft.com/samples/browse/?products=azure&languages=java).

## Connect to and use Azure resources with client libraries

The SDK's client (or "data plane") libraries help you write Java application code to interact with already-provisioned services. Client libraries exist only for those services that support a client API.

For details on working with each client library, see the README.md file located in the library's project folder in the [SDK GitHub repository](https://github.com/Azure/azure-sdk-for-java). You can also find additional code snippets in the [reference documentation](https://docs.microsoft.com/java/api) and the [Azure Samples](https://docs.microsoft.com/samples/browse/?products=azure&languages=java).

## Get help and connect with the SDK team

Visit the [Azure libraries for Java documentation](https://aka.ms/java-docs)
Post questions to the community on [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-sdk-for-java)
Open issues against the SDK on [GitHub](https://github.com/Azure/azure-sdk-for-java/issues)
Mention [@AzureSDK](https://twitter.com/AzureSdk/) on Twitter
