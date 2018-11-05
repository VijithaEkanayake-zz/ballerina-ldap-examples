# ballerina-ldap-examples
This repository contains examples to demonstrate LDAP integration scenarios with Ballerina

### Basic LDAP example

##### sample LDAP auth store configurations
```ballerina
// Configuration options to integrate Ballerina service
// with an LDAP server
auth:LdapAuthProviderConfig ldapAuthProviderConfig = {
    // Unique name to identify the user store
    domainName: "ballerina.io",
    // Connection URL to the LDAP server
    connectionURL: "ldap://localhost:9389",
    // The username used to connect to the LDAP server
    connectionName: "uid=admin,ou=system",
    // Password for the ConnectionName user
    connectionPassword: "secret",
    // DN of the context or object under which the user entries are stored in the LDAP server
    userSearchBase: "ou=Users,dc=ballerina,dc=io",
    // Object class used to construct user entries
    userEntryObjectClass: "identityPerson",
    // The attribute used for uniquely identifying a user entry
    userNameAttribute: "uid",
    // Filtering criteria used to search for a particular user entry
    userNameSearchFilter: "(&(objectClass=person)(uid=?))",
    // Filtering criteria for searching user entries in the LDAP server
    userNameListFilter: "(objectClass=person)",
    // DN of the context or object under which the group entries are stored in the LDAP server
    groupSearchBase: ["ou=Groups,dc=ballerina,dc=io"],
    // Object class used to construct group entries
    groupEntryObjectClass: "groupOfNames",
    // The attribute used for uniquely identifying a group entry
    groupNameAttribute: "cn",
    // Filtering criteria used to search for a particular group entry
    groupNameSearchFilter: "(&(objectClass=groupOfNames)(cn=?))",
    // Filtering criteria for searching group entries in the LDAP server
    groupNameListFilter: "(objectClass=groupOfNames)",
    // Define the attribute that contains the distinguished names (DN) of user objects that are in a group
    membershipAttribute: "member",
    // To indicate whether to cache the role list of a user
    userRolesCacheEnabled: true,
    // Define whether LDAP connection pooling is enabled
    connectionPoolingEnabled: false,
    // Timeout in making the initial LDAP connection
    ldapConnectionTimeout: 5000,
    // The value of this property is the read timeout in milliseconds for LDAP operations
    readTimeout: 60000,
    // Retry the authentication request if a timeout happened
    retryAttempts: 3
};
```

##### How basic ldap example works.

- Embedded directory server starts with the below ldap schema.
```ballerina
version: 1

dn: dc=ballerina,dc=io
objectClass: extensibleObject
objectClass: domain
objectClass: top
dc: ballerina

dn: ou=Users,dc=ballerina,dc=io
objectClass: organizationalUnit
objectClass: top
ou: Users

dn: ou=Groups,dc=ballerina,dc=io
objectClass: organizationalUnit
objectClass: top
ou: Groups

dn: uid=admin,ou=Users,dc=ballerina,dc=io
objectClass: organizationalPerson
objectClass: person
objectClass: extensibleObject
objectClass: uidObject
objectClass: inetOrgPerson
objectClass: top
cn: Test User
givenName: admin
sn: User
uid: admin
mail: admin@ballerina.io
ou: Users
userpassword: ballerina

dn: uid=user2,ou=Users,dc=ballerina,dc=io
objectClass: organizationalPerson
objectClass: person
objectClass: extensibleObject
objectClass: uidObject
objectClass: inetOrgPerson
objectClass: top
cn: Test User
givenName: Demo
sn: User
uid: user2
mail: user2@ballerina.io
ou: Users
userpassword: ballerina

dn: uid=user1,ou=Users,dc=ballerina,dc=io
objectClass: organizationalPerson
objectClass: person
objectClass: extensibleObject
objectClass: uidObject
objectClass: inetOrgPerson
objectClass: top
cn: Test User
givenName: User1
sn: User
uid: user1
mail: user1@ballerina.io
ou: Users
userpassword: ballerina

dn: cn=admin,ou=Groups,dc=ballerina,dc=io
objectClass: groupOfNames
objectClass: top
cn: admin
member: uid=user1,ou=Users,dc=ballerina,dc=io
member: uid=admin,ou=Users,dc=ballerina,dc=io
member: uid=user2,ou=Users,dc=ballerina,dc=io
```

- Ballerina service is implemented in way to use LDAP user store to do the authentications/authorizations.
```ballerina
import ballerina/auth;
import ballerina/http;

// Configuration options to integrate Ballerina service
// with an LDAP server
auth:LdapAuthProviderConfig ldapAuthProviderConfig = {
    // Unique name to identify the user store
    domainName: "ballerina.io",
    // Connection URL to the LDAP server
    connectionURL: "ldap://localhost:9389",
    // The username used to connect to the LDAP server
    connectionName: "uid=admin,ou=system",
    // Password for the ConnectionName user
    connectionPassword: "secret",
    // DN of the context or object under which
    // the user entries are stored in the LDAP server
    userSearchBase: "ou=Users,dc=ballerina,dc=io",
    // Object class used to construct user entries
    userEntryObjectClass: "identityPerson",
    // The attribute used for uniquely identifying a user entry
    userNameAttribute: "uid",
    // Filtering criteria used to search for a particular user entry
    userNameSearchFilter: "(&(objectClass=person)(uid=?))",
    // Filtering criteria for searching user entries in the LDAP server
    userNameListFilter: "(objectClass=person)",
    // DN of the context or object under which
    // the group entries are stored in the LDAP server
    groupSearchBase: ["ou=Groups,dc=ballerina,dc=io"],
    // Object class used to construct group entries
    groupEntryObjectClass: "groupOfNames",
    // The attribute used for uniquely identifying a group entry
    groupNameAttribute: "cn",
    // Filtering criteria used to search for a particular group entry
    groupNameSearchFilter: "(&(objectClass=groupOfNames)(cn=?))",
    // Filtering criteria for searching group entries in the LDAP server
    groupNameListFilter: "(objectClass=groupOfNames)",
    // Define the attribute that contains the distinguished names
    // (DN) of user objects that are in a group
    membershipAttribute: "member",
    // To indicate whether to cache the role list of a user
    userRolesCacheEnabled: true,
    // Define whether LDAP connection pooling is enabled
    connectionPoolingEnabled: false,
    // Timeout in making the initial LDAP connection
    ldapConnectionTimeout: 5000,
    // The value of this property is the read timeout in
    // milliseconds for LDAP operations
    readTimeout: 60000,
    // Retry the authentication request if a timeout happened
    retryAttempts: 3
};

http:AuthProvider basicAuthProvider = {
    id: "basic01",
    scheme: http:AUTHN_SCHEME_BASIC,
    authStoreProvider: http:AUTH_PROVIDER_LDAP,
    authStoreProviderConfig: ldapAuthProviderConfig
};

// The endpoint used here is `http:SecureListener`, which by default tries to
// authenticate and authorize each request. The developer has the option to
// override the authentication and authorization at the service level and
// resource level.
endpoint http:SecureListener ep {
    port: 9090,
    authProviders: [basicAuthProvider],
    secureSocket: {
        keyStore: {
            path: "${ballerina.home}/bre/security/ballerinaKeystore.p12",
            password: "ballerina"
        }
    }
};

@http:ServiceConfig {
    basePath: "/ldapAuth",
    authConfig: {
        authentication: { enabled: true }
    }
}
// Auth configuration comprises of two parts - authentication & authorization.
// Authentication can be enabled by setting the `authentication:{enabled:true}`
// annotation attribute.
service<http:Service> helloService bind ep {

    @http:ResourceConfig {
        methods: ["GET"],
        path: "/resourceOne",
        authConfig: {
            scopes: ["admin"]
        }
    }
    // The authentication and authorization settings can be overridden at
    // resource level.
    // The enableAuthz resource would inherit the
    // `authentication:{enabled:true}` flag from the service level,
    // and override the scope defined in the service level with test scope.
    resourceOne(endpoint caller, http:Request req) {
        _ = caller->respond("Hello, World!!!");
    }

    @http:ResourceConfig {
        methods: ["GET"],
        path: "/resourceTwo",
        authConfig: {
            scopes: ["test"]
        }
    }
    resourceTwo(endpoint caller, http:Request req) {
        _ = caller->respond("Hello, World!!!");
    }
}
```

### How to try out the example

- Clone the git repository to a desired location.
- Navigate ldap-examples/ldap-server directory and execute following command to start the embedded ldap server.
```bash
java -jar embedded-ldap-server-1.0.0-jar-with-dependencies.jar
```
- From another terminal window navigate to ldap-examples/basic-examples directory and execute the following command 
to start the Ballerina service.
```ballerina
ballerina run ldap_auth_store_example.bal
```
- Invoke the service with below curl command.

With valid user credenatils
```ballerina
curl -kv https://localhost:9090/ldapAuth/resourceOne -H "Authorization: Basic YWRtaW46YmFsbGVyaW5h"
```
As an out put you should get a successful response as "Hello, World!!!" from the Ballerina service.

With invalid user credentials
```ballerina
curl -kv https://localhost:9090/ldapAuth/resourceOne -H "Authorization: Basic dmlqaXRoYTpiYWxsZXJpbmE"
```
Out put of above command should result an "Authentication failure"

With valid user authentication credentials, trying to access unauthorized resource.
```ballerina
curl -kv https://localhost:9090/ldapAuth/resourceTwo -H "Authorization: Basic YWRtaW46YmFsbGVyaW5h"
```
Out put of above command should result an "Authorization failure"
