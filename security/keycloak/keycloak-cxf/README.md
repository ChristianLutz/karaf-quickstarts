keycloak-cxf: demonstrates a bundle that registers CXF endpoints that are protected using Keycloak
==========================
Author: Fuse Team  
Level: Intermediate  
Technologies: Keycloak, PAX-WEB, Blueprint, CXF
Summary: a bundle that registers CXF endpoints that are protected using Keycloak
Target Product: Fuse  
Source: <https://github.com/jboss-fuse/karaf-quickstarts/tree/7.x.redhat-7-x/security/keycloak/keycloak-cxf>


What is it?
-----------
This quickstart demonstrates how to create an OSGi bundle that exposes CXF endpoints (both JAX-WS and JAX-RS) protected
by Keycloak authentication mechanisms.

Both kinds of endpoints are exposed using two different methods:

* using embedded Undertow servlet engine (the one that is started by default in Red Hat Fuse and runs hawtio console)
* using separate Undertow servlet engine started and managed by CXF itself


System requirements
-------------------
Before building and running this quick start you need:

* Maven 3.5.0 or higher
* JDK 1.8
* Red Hat Fuse 7


Build and Deploy the Quickstart
-------------------------------

To build the quick start:

1. Change your working directory to `keycloak-cxf` directory.
2. Run `mvn clean install` to build the quickstart.
3. Start Red Hat Fuse 7 by running bin/fuse (on Linux) or bin\fuse.bat (on Windows).
4. In the Red Hat Fuse console, enter the following commands to install required Keycloak features

        feature:repo-add mvn:org.keycloak/keycloak-osgi-features/${version.org.keycloak}/xml/features
        feature:install -v keycloak-pax-http-undertow

5. In the Red Hat Fuse console, enter the following command to install `keycloak-cxf` quickstart:

        install -s mvn:org.jboss.fuse.quickstarts.security/keycloak-cxf/${project.version}


Use the bundle
--------------

After installing `keycloak-cxf` bundle, we'll have 4 CXF endpoints exposed:

* http://localhost:8282/jaxws - SOAP endpoint using separate Undertow servlet engine
* http://localhost:8282/jaxrs - REST endpoint using separate Undertow servlet engine
* http://localhost:8181/cxf/jaxws - SOAP endpoint using embedded Undertow servlet engine
* http://localhost:8181/cxf/jaxrs - REST endpoint using embedded Undertow servlet engine


### Testing separate Undertow servlet engine

We have to create file `etc/jaxws-keycloak.json` **and** `etc/jaxrs-keycloak.json` with the following
content configuring Keycloak integration:
   
       {
         "realm": "fuse7karaf",
         "auth-server-url": "http://localhost:8180/auth",
         "ssl-required": "external",
         "resource": "cxf-external",
         "public-client": true,
         "use-resource-role-mappings": true,
         "confidential-port": 0,
         "principal-attribute": "preferred_username"
       }

Again - this (using two files) is required, because both endpoints are registered in "default" (`/`) context.

To test the endpoints running on separate Undertow servlet engine, we can run the unit tests inside `keycloak-cxf`
directory:

    $ mvn test -Pqtest -Dtest=JaxRsClientTest#helloExternalAuthenticated,JaxWsClientTest#helloExternalAuthenticated
    [INFO] Scanning for projects...
    [INFO] 
    [INFO] ----------< org.jboss.fuse.quickstarts.security:keycloak-cxf >----------
    [INFO] Building Red Hat Fuse :: Quickstarts :: Security :: Keycloak :: CXF ${project.version}
    [INFO] -------------------------------[ bundle ]-------------------------------
    [INFO] 
    [INFO] --- maven-resources-plugin:3.0.2:resources (default-resources) @ keycloak-cxf ---
    [INFO] Using 'UTF-8' encoding to copy filtered resources.
    [INFO] Copying 1 resource
    [INFO] 
    [INFO] --- maven-compiler-plugin:3.7.0:compile (default-compile) @ keycloak-cxf ---
    [INFO] Changes detected - recompiling the module!
    [INFO] Compiling 2 source files to /data/sources/github.com/jboss-fuse/karaf-quickstarts/distribution/target/classes/security/keycloak/keycloak-cxf/target/classes
    [INFO] 
    [INFO] --- maven-bundle-plugin:3.5.1:manifest (bundle-manifest) @ keycloak-cxf ---
    [INFO] 
    [INFO] --- maven-resources-plugin:3.0.2:testResources (default-testResources) @ keycloak-cxf ---
    [INFO] Using 'UTF-8' encoding to copy filtered resources.
    [INFO] Copying 1 resource
    [INFO] 
    [INFO] --- maven-compiler-plugin:3.7.0:testCompile (default-testCompile) @ keycloak-cxf ---
    [INFO] Changes detected - recompiling the module!
    [INFO] Compiling 2 source files to /data/sources/github.com/jboss-fuse/karaf-quickstarts/distribution/target/classes/security/keycloak/keycloak-cxf/target/test-classes
    [INFO] 
    [INFO] --- maven-surefire-plugin:2.20.1:test (default-test) @ keycloak-cxf ---
    [INFO] 
    [INFO] -------------------------------------------------------
    [INFO]  T E S T S
    [INFO] -------------------------------------------------------
    [INFO] Running org.jboss.fuse.quickstarts.security.keycloak.cxf.JaxRsClientTest
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-0 >> "POST /auth/realms/fuse7karaf/protocol/openid-connect/token HTTP/1.1[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-0 >> "Authorization: Basic Y3hmLWV4dGVybmFsOjdlMjBhZGRkLTg3ZmMtNDUyOC04MDhjLWU5YzdjOTUwZWYyMw==[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-0 >> "Content-Length: 52[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-0 >> "Content-Type: application/x-www-form-urlencoded[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-0 >> "Host: localhost:8180[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-0 >> "Connection: Keep-Alive[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-0 >> "User-Agent: Apache-HttpClient/4.5.5 (Java/1.8.0_241)[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-0 >> "[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-0 >> "grant_type=password&username=admin&password=passw0rd"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-0 << "HTTP/1.1 200 OK[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-0 << "Connection: keep-alive[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-0 << "Cache-Control: no-store[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-0 << "Set-Cookie: KC_RESTART=; Version=1; Expires=Thu, 01-Jan-1970 00:00:10 GMT; Max-Age=0; Path=/auth/realms/fuse7karaf/; HttpOnly[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-0 << "Pragma: no-cache[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-0 << "Content-Type: application/json[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-0 << "Content-Length: 3988[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-0 << "Date: Thu, 16 Jan 2020 08:56:48 GMT[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-0 << "[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-0 << "{"access_token":<token>,"token_type":"bearer","not-before-policy":0,"session_state":"a011b925-998c-4a6f-9ce1-f70c79155ea8","scope":"profile email"}"
    09:56:48 INFO [org.jboss.fuse.quickstarts.security.keycloak.cxf.JaxRsClientTest] : token: <token>
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-1 >> "GET /jaxrs/service/hello/hi HTTP/1.1[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-1 >> "Authorization: Bearer <token>[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-1 >> "Host: localhost:8282[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-1 >> "Connection: Keep-Alive[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-1 >> "User-Agent: Apache-HttpClient/4.5.5 (Java/1.8.0_241)[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-1 >> "[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-1 << "HTTP/1.1 200 OK[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-1 << "Connection: keep-alive[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-1 << "Transfer-Encoding: chunked[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-1 << "Content-Type: application/json[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-1 << "Date: Thu, 16 Jan 2020 08:56:48 GMT[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-1 << "[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-1 << "11[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-1 << "{"result":"[hi]"}[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-1 << "0[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-1 << "[\r][\n]"
    09:56:48 INFO [org.jboss.fuse.quickstarts.security.keycloak.cxf.JaxRsClientTest] : response: {"result":"[hi]"}
    [INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.943 s - in org.jboss.fuse.quickstarts.security.keycloak.cxf.JaxRsClientTest
    [INFO] Running org.jboss.fuse.quickstarts.security.keycloak.cxf.JaxWsClientTest
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-2 >> "POST /auth/realms/fuse7karaf/protocol/openid-connect/token HTTP/1.1[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-2 >> "Authorization: Basic Y3hmLWV4dGVybmFsOjdlMjBhZGRkLTg3ZmMtNDUyOC04MDhjLWU5YzdjOTUwZWYyMw==[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-2 >> "Content-Length: 52[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-2 >> "Content-Type: application/x-www-form-urlencoded[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-2 >> "Host: localhost:8180[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-2 >> "Connection: Keep-Alive[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-2 >> "User-Agent: Apache-HttpClient/4.5.5 (Java/1.8.0_241)[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-2 >> "[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-2 >> "grant_type=password&username=admin&password=passw0rd"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-2 << "HTTP/1.1 200 OK[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-2 << "Connection: keep-alive[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-2 << "Cache-Control: no-store[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-2 << "Set-Cookie: KC_RESTART=; Version=1; Expires=Thu, 01-Jan-1970 00:00:10 GMT; Max-Age=0; Path=/auth/realms/fuse7karaf/; HttpOnly[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-2 << "Pragma: no-cache[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-2 << "Content-Type: application/json[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-2 << "Content-Length: 3988[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-2 << "Date: Thu, 16 Jan 2020 08:56:48 GMT[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-2 << "[\r][\n]"
    09:56:48 DEBUG [org.apache.http.wire] : http-outgoing-2 << "{"access_token":<token>,"expires_in":300,"refresh_expires_in":1800,"refresh_token":<token>,"token_type":"bearer","not-before-policy":0,"session_state":"c4412d02-623e-4f0d-aa59-574bda4c6a11","scope":"profile email"}"
    09:56:48 INFO [org.jboss.fuse.quickstarts.security.keycloak.cxf.JaxWsClientTest] : token: <token>
    09:56:49 INFO [org.apache.cxf.wsdl.service.factory.ReflectionServiceFactoryBean] : Creating Service {urn:fuse:cxf:1}JaxWsServiceService from class org.jboss.fuse.quickstarts.security.keycloak.cxf.JaxWsService
    09:56:49 INFO [org.jboss.fuse.quickstarts.security.keycloak.cxf.JaxWsClientTest] : Result: [Hi]
    [INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.754 s - in org.jboss.fuse.quickstarts.security.keycloak.cxf.JaxWsClientTest
    [INFO] 
    [INFO] Results:
    [INFO] 
    [INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0
    [INFO] 
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time:  5.074 s
    [INFO] Finished at: 2020-01-16T09:56:49+01:00
    [INFO] ------------------------------------------------------------------------


### Testing embedded Undertow servlet engine

In case of endpoints deployed using embedded Undertow servlet engine, the configuration is slightly different.
The `/cxf` servlet is already registered and we have to _alter_ it to be able to delegate to Keycloak authentication
mechanism.

[PAX-WEB](https://ops4j1.jira.com/wiki/spaces/paxweb/pages/354025473/HTTP+Context+processing) provides a mechanism
where existing servlets can be _reconfigured_ to change (for example) the authentication mechanisms used.

We have to create `\${karaf.etc}/org.ops4j.pax.web.context-cxf.cfg` factory PID configuration which will be
processed by PAX-WEB that will alter `/cxf` servlet registered by CXF bundles.

The content of the file is:

    bundle.symbolicName = org.apache.cxf.cxf-rt-transports-http
    context.id = default
    
    context.param.keycloak.config.resolver = org.keycloak.adapters.osgi.HierarchicalPathBasedKeycloakConfigResolver
    
    login.config.authMethod = KEYCLOAK
    login.config.realmName = _does_not_matter
    
    security.constraint.1.url = /cxf/jaxws/*
    security.constraint.1.roles = admin
    security.constraint.2.url = /cxf/jaxrs/*
    security.constraint.2.roles = admin

Because we've used `HierarchicalPathBasedKeycloakConfigResolver` instead of `PathBasedKeycloakConfigResolver`,
we can separately secure two endpoints. We can do it by creating `etc/cxf-jaxrs-keycloak.json` and
`etc/cxf-jaxws-keycloak.json` with the following content configuring Keycloak integration:
                                                
    {
      "realm": "fuse7karaf",
      "auth-server-url": "http://localhost:8180/auth",
      "ssl-required": "external",
      "resource": "cxf",
      "use-resource-role-mappings": true,
      "confidential-port": 0,
      "principal-attribute": "preferred_username",
      "public-client": true
    }

With `PathBasedKeycloakConfigResolver` we'd have to use single `etc/cxf-keycloak.json` file that would affect *all*
the CXF endpoints.

To test the endpoints running on embedded Undertow servlet engine, we can run the unit tests inside `keycloak-cxf`
directory:

    $ mvn test -Pqtest -Dtest=JaxRsClientTest#helloEmbeddedAuthenticated,JaxWsClientTest#helloEmbeddedAuthenticated
    [INFO] Scanning for projects...
    [INFO] 
    [INFO] ----------< org.jboss.fuse.quickstarts.security:keycloak-cxf >----------
    [INFO] Building Red Hat Fuse :: Quickstarts :: Security :: Keycloak :: CXF ${project.version}
    [INFO] -------------------------------[ bundle ]-------------------------------
    [INFO] 
    [INFO] --- maven-resources-plugin:3.0.2:resources (default-resources) @ keycloak-cxf ---
    [INFO] Using 'UTF-8' encoding to copy filtered resources.
    [INFO] Copying 1 resource
    [INFO] 
    [INFO] --- maven-compiler-plugin:3.7.0:compile (default-compile) @ keycloak-cxf ---
    [INFO] Nothing to compile - all classes are up to date
    [INFO] 
    [INFO] --- maven-bundle-plugin:3.5.1:manifest (bundle-manifest) @ keycloak-cxf ---
    [INFO] 
    [INFO] --- maven-resources-plugin:3.0.2:testResources (default-testResources) @ keycloak-cxf ---
    [INFO] Using 'UTF-8' encoding to copy filtered resources.
    [INFO] Copying 1 resource
    [INFO] 
    [INFO] --- maven-compiler-plugin:3.7.0:testCompile (default-testCompile) @ keycloak-cxf ---
    [INFO] Nothing to compile - all classes are up to date
    [INFO] 
    [INFO] --- maven-surefire-plugin:2.20.1:test (default-test) @ keycloak-cxf ---
    [INFO] 
    [INFO] -------------------------------------------------------
    [INFO]  T E S T S
    [INFO] -------------------------------------------------------
    [INFO] Running org.jboss.fuse.quickstarts.security.keycloak.cxf.JaxRsClientTest
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-0 >> "POST /auth/realms/fuse7karaf/protocol/openid-connect/token HTTP/1.1[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-0 >> "Authorization: Basic Y3hmOmYxZWM3MTZkLTIyNjItNDM0ZC04ZTk4LWJmMzFiNmI4NThkNg==[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-0 >> "Content-Length: 52[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-0 >> "Content-Type: application/x-www-form-urlencoded[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-0 >> "Host: localhost:8180[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-0 >> "Connection: Keep-Alive[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-0 >> "User-Agent: Apache-HttpClient/4.5.5 (Java/1.8.0_241)[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-0 >> "[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-0 >> "grant_type=password&username=admin&password=passw0rd"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-0 << "HTTP/1.1 200 OK[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-0 << "Connection: keep-alive[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-0 << "Cache-Control: no-store[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-0 << "Set-Cookie: KC_RESTART=; Version=1; Expires=Thu, 01-Jan-1970 00:00:10 GMT; Max-Age=0; Path=/auth/realms/fuse7karaf/; HttpOnly[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-0 << "Pragma: no-cache[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-0 << "Content-Type: application/json[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-0 << "Content-Length: 3976[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-0 << "Date: Thu, 16 Jan 2020 08:59:15 GMT[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-0 << "[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-0 << "{"access_token":<token>,"expires_in":300,"refresh_expires_in":1800,"refresh_token":<token>,"token_type":"bearer","not-before-policy":0,"session_state":"89e6d239-c5c0-43f7-a727-6e3ec850b216","scope":"profile email"}"
    09:59:15 INFO [org.jboss.fuse.quickstarts.security.keycloak.cxf.JaxRsClientTest] : token: <token>
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-1 >> "GET /cxf/jaxrs/service/hello/hi HTTP/1.1[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-1 >> "Authorization: Bearer <token>[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-1 >> "Host: localhost:8181[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-1 >> "Connection: Keep-Alive[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-1 >> "User-Agent: Apache-HttpClient/4.5.5 (Java/1.8.0_241)[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-1 >> "[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-1 << "HTTP/1.1 200 OK[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-1 << "Expires: 0[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-1 << "Cache-Control: no-cache, no-store, must-revalidate[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-1 << "X-Powered-By: Open Source[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-1 << "Server: Pax-HTTP-Undertow[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-1 << "Pragma: no-cache[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-1 << "Date: Thu, 16 Jan 2020 08:59:15 GMT[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-1 << "Connection: keep-alive[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-1 << "Transfer-Encoding: chunked[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-1 << "Content-Type: application/json[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-1 << "[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-1 << "11[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-1 << "{"result":"[hi]"}[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-1 << "0[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-1 << "[\r][\n]"
    09:59:15 INFO [org.jboss.fuse.quickstarts.security.keycloak.cxf.JaxRsClientTest] : response: {"result":"[hi]"}
    [INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.827 s - in org.jboss.fuse.quickstarts.security.keycloak.cxf.JaxRsClientTest
    [INFO] Running org.jboss.fuse.quickstarts.security.keycloak.cxf.JaxWsClientTest
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-2 >> "POST /auth/realms/fuse7karaf/protocol/openid-connect/token HTTP/1.1[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-2 >> "Authorization: Basic Y3hmOmYxZWM3MTZkLTIyNjItNDM0ZC04ZTk4LWJmMzFiNmI4NThkNg==[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-2 >> "Content-Length: 52[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-2 >> "Content-Type: application/x-www-form-urlencoded[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-2 >> "Host: localhost:8180[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-2 >> "Connection: Keep-Alive[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-2 >> "User-Agent: Apache-HttpClient/4.5.5 (Java/1.8.0_241)[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-2 >> "[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-2 >> "grant_type=password&username=admin&password=passw0rd"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-2 << "HTTP/1.1 200 OK[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-2 << "Connection: keep-alive[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-2 << "Cache-Control: no-store[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-2 << "Set-Cookie: KC_RESTART=; Version=1; Expires=Thu, 01-Jan-1970 00:00:10 GMT; Max-Age=0; Path=/auth/realms/fuse7karaf/; HttpOnly[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-2 << "Pragma: no-cache[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-2 << "Content-Type: application/json[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-2 << "Content-Length: 3976[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-2 << "Date: Thu, 16 Jan 2020 08:59:15 GMT[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-2 << "[\r][\n]"
    09:59:15 DEBUG [org.apache.http.wire] : http-outgoing-2 << "{"access_token":<token>,"expires_in":300,"refresh_expires_in":1800,"refresh_token":<token>,"token_type":"bearer","not-before-policy":0,"session_state":"ce0ce5f8-33ca-4e8b-9e6b-1628b0faa762","scope":"profile email"}"
    09:59:15 INFO [org.jboss.fuse.quickstarts.security.keycloak.cxf.JaxWsClientTest] : token: <token>
    09:59:15 INFO [org.apache.cxf.wsdl.service.factory.ReflectionServiceFactoryBean] : Creating Service {urn:fuse:cxf:1}JaxWsServiceService from class org.jboss.fuse.quickstarts.security.keycloak.cxf.JaxWsService
    09:59:16 INFO [org.jboss.fuse.quickstarts.security.keycloak.cxf.JaxWsClientTest] : Result: [Hi]
    [INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.623 s - in org.jboss.fuse.quickstarts.security.keycloak.cxf.JaxWsClientTest
    [INFO] 
    [INFO] Results:
    [INFO] 
    [INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0
    [INFO] 
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time:  3.965 s
    [INFO] Finished at: 2020-01-16T09:59:16+01:00
    [INFO] ------------------------------------------------------------------------

We have to create `etc/cxf-keycloak.json` file, so http://localhost:8181/cxf URL is accessible (when handling context processed with pax-web using `etc/org.ops4j.pax.web.context-cxf.cfg`). It is however **not** protected and we can access
a list of CXF deployed endpoints without a need to authenticate via Keycloak.
We can also set a page with the list of CXF endpoints as protected, accessibble only after successful OAuth2 authentication.
`\${karaf.etc}/org.ops4j.pax.web.context-cxf.cfg` would have to contain additional lines:

    security.constraint.3.url = /cxf
    security.constraint.3.roles = admin


Undeploy the Bundle
-------------------

To stop and undeploy the bundle in Fuse:

1. Enter `bundle:list -s` command to retrieve your bundle id and symbolic name
2. To stop and uninstall the bundle enter

        bundle:uninstall <id>

    or (uninstall by symbolic name)

        bundle:uninstall keycloak-cxf
