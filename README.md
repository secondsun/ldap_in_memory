# IM-LDAP Content for Open Source Community

## Overview
The in-memory LDAP component (IM-LDAP) simplifies the task of standing up an LDAP server for non-production environments. These procedures explain how to easily configure an IM-LDAP directory server to use for testing, for demonstration purposes, or even for simple LDAP data processing tasks. This method has the advantage of not having to install and maintain a server and provides for server-independent functionality.
This implementation uses the UnboundedID Java SDK that includes a fast, powerful, and user-friendly Java API for communicating with LDAP directory servers. The UnboundID Identity Data Platform provides a directory services solution designed to scale for extremely large identity datasets and also provides the data synchronization services necessary to support a low-risk migration from existing directory services solutions.
The UnboundID LDAP SDK for Java is free to use and redistribute in open source or proprietary applications under the GPLv2, LGPLv2.1 and the UnboundID Free Use License. It does not have any third-party dependencies and commercial support is available from UnboundID.

The UnboundedID Java SDK has capability restrictions that require LDAP schema changes to be performed offline.  For more information on LDAP SDK features refer to:
https://www.unboundid.com/products/ldapsdk/docs/getting-started/features.php

*IMPORTANT: The UnboundedID Java SDK only provides an implementation to support plain text passwords (an interface is provided to create other extensions). When LDIF files are imported, the IM-LDAP server will automatically reset the password to "username" + "1234".*

## Configuration
The IM-LDAP self-configures by using the contents of the configuration file ldap-inmemory-config.xml, which is discussed below. The configuration file allows for specification of the root node, binding details (including password), the ability to add individual LDAP entries (without attributes), and the import of specified LDIF export files. 

### LDIF Adjustor
The LDAP schema is loaded from the LDIF file that is defined in the configuration file. A utility class has been added as a tool to simplify the configuration process. LDIF exports must conform to the schema and include the changetype attribute. A helper class has been added which can be run to generate modified LDIF export files to add the missing attribute.
Please refer to the class com.slc.sli.ldap.inmemory.utils.LdifAdjustor for more information. [Add reference] This class supports both IDE and command line execution. 

### Logging
Logging occurs through the SLF4J logging facade which is backed by Log4J. Log levels can be configured from within the log4j.xml. Two handlers have been implemented to allow IM-LDAP events to be exposed by log statements, which could also be captured by specific log file appenders (not included at this time).
Listener
The IM-LDAP allows for multiple listeners to be configured against the same internal server repository distinguished by port and authentication mechanism. However, currently, this implementation only supports use of one listener via the configuration file. Refer to the UnboundedID documentation [link] for more information. 

## Deployment
Deployment occurs by adding a WAR file and resources. To accomplish this, you will copy and modify the resources from test into your classpath to a servlet container such as Jetty or Tomcat. A class is provided within the test package to allow the server to be easily run within an IDE. 
Default host: 127.0.0.1
Port: 10389

### Configuration
inBloom Platform
After SDS artifacts have been built and deployed, you must modify the sli.properties entries. Alternatively, the YML used to generate sli.properties could be modified. However, this procedure is more complex and changes the default "local" behavior.
To generate a value for the property sli.api.ldap.pass that matches the IM-LDAP root password, run the org.slc.sli.encryption.tool.Encryptor class via your IDE. The class contains a main method that can be executed from within the IDE (or modified locally as required). The example below uses a value based on the password “test1234”.
NOTE: sli.simple-idp.ldap.urls should use the appropriate IP address. This implementation will attempt to bind to all interfaces available on the provided address.
Within sli.properties change the following values:
•	sli.simple-idp.ldap.urls = ldap://127.0.0.1:10389
•	sli.api.ldap.pass = FEEBEC071D0C8CA1886224930EEE3F6D
For reference: the original value from git: 
 #sli.api.ldap.pass = F8EEEB5397084B5DD8EA44604D8CC39A
•	sli.api.ldap.user = cn=Admin,dc=slidev,dc=org
ldap-inmemory-config.xml
•	Refer to the ldap-inmemory-config.xml file found within the test/resources. This config file is validated against the ldap-inmemory-config.xsd.
•	A root element must be specified within the configuration. All LDIFs are loaded beneath the root. If an LDIF contains the root node, remove it from the LDIF and add it to the config file. Note that only one root element may be specified.
•	LDIF files are loaded, in order, as defined in the configuration file. One to many LDIF files may be loaded. Optionally, zero to many LDAP entries may be specified (without attributes).
•	At this time, only one listener is supported. If the address is not specified for a listener, the server will attempt to bind to all available local interfaces. (This is considered default behavior for the UnboundedID SDK. For more information, refer to the UnboundedID documentation [link].)


### Supported Platforms
The IM-LDAP runs within a Java Virtual Machine which means it is cross-portable to any OS that has a JDK.
Recommended Skill Sets
Skill set for IM-LDAP out-of-box use:
•	Maven
•	Jetty
Skill set to change or modify how IM-LDAP works:
•	Familiarity with LDAP (entries, Apache Directory Studio, LDIF export format, LDAP schemas, keystones)
•	Maven
•	Jetty/Tomcat
•	XML
•	Java (classpath)
LDAP Best Practices
http://www.ldapguru.info/ldap/ldap-programming-practices.html

### Prerequisites
•	SDS installation/configuration is recommended prior to setting up LDAP. Then, once the LDAP is set up, you can deploy and run it on the configured SDS platform. The links below provide information on SDS installation/configuration:
inBloom platform with the following components running: API, Simple IDP, Ingest, Search Indexer, and in-memory LDAP server running.
•	Small dataset successfully loaded.


Local Deployment
A local deployment can be invoked directly via Maven. When the IM-LDAP server is run in this mode, it will use the "test" classpath and "test" resources.
1.	Access the git repository and navigate to the root directory.
2.	Build the artifacts: "mvn clean install package".
3.	Run the server: mvn jetty:run". This will use the “test” classpath (which means resources found beneath test).
4.	Start the admin railZ app. 
5.	Select the URL: http://local.slidev.org:3001/application_authorizations
6.	Login to inBloom realm using iladmin.
7.	Start the data browser: http://local.slidev.org:3000
8.	Login to "Il daybreak 4529" realm using the user "rrogers". Remember, all passwords are reset to "username" + "1234" when LDIFs are loaded, so the password will be "rrogers1234".
9.	Browse data.
NOTE: It is possible to restart the IM-LDAP server while Simple IDP is running without affecting their respective functionality.

Deployment

A deployment can be invoked directly via Maven. When the IM- LDAP server is run in this mode, it will use the "test" classpath and "test" resources.
1.	Access the git repository and navigate to the root directory.
2.	Build the artifacts: "mvn clean install package".
3.	Copy the WAR () into your servlet container/application server (e.g. Tomcat, Jetty, Resin, Jboss, etc).
4.	Make the required resources available on the classpath: 
a.	ldap-inmemory-config.xml
b.	Resources specified within the XML must exist relative to classpath:
•	ldif/ldap-schema.ldif (Replace with LDIF schema export if customized, one schema for all LDIFs.)
•	ldif/2014_01_16_ldap_export.ldif.modified (Replace with LDIFs, if customized.)
5.	Start the server.
6.	Verify proper functionality using the Apache Directory Studio.

Apache Directory Studio
You can connect to the IM-LDAP by using the Apache Directory Studio.
Create a new connection to the local LDAP:
Host: 127.0.0.1:10389
Simple Authentication:
cn=Admin,dc=slidev,dc=org
test1234
Results
After performing the deployment process,the mvn jetty:run will startup the server without throwing any stack traces.
Being able to successfully connect and browse the component when it starts with the server.

## Examples
N/A.  The in-memory LDAP server is a runnable component.

## Troubleshooting
N/A.  The defaults have been tested and verified to work as-is from a direct checkout.  Any common questions will be added to this section per feedback from the community.


## Source Code
The source code can be obtained from the git repository at the following location: https://github.com/inbloomdev/datastore_ldap_inmemory. 


## Resources
The following list includes items related to this solution:
DS-960 
https://www.unboundid.com/products/ldap-sdk/docs/javadoc/
https://www.unboundid.com/products/ldapsdk/docs/javadoc/com/unboundid/ldap/listener/InMemoryDirectoryServer.html
https://www.unboundid.com/products/ldap-sdk/docs/javadoc/com/unboundid/util/ssl/SSLUtil.html
https://github.com/trevershick/ldap-test-utils

## Support
For support, visit the inBloom Open Source Developer Community Forum, a discussion forum for developers and other members of the inBloom community:
https://forums.inbloom.org/viewforum.php?f=9


