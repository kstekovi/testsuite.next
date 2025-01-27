[![TC Build](https://ci.wildfly.org/app/rest/builds/buildType:(id:hal_TestSuiteDevelopment)/statusIcon.svg)](https://ci.wildfly.org/viewType.html?buildTypeId=hal_TestSuiteDevelopment&guest=1)

# Testsuite

Testsuite for the HAL management console based on [Drone & Graphene](http://arquillian.org/guides/functional_testing_using_graphene/) Arquillian extensions.

## Profiles

The testsuite uses various profiles to decide how and which tests to run. The following profiles are available:

- `chrome` | `firefox` | `safari`: Defines the browser to run the tests (mutual exclusive)
- `basic`, `rbac`, `transaction`, `multihosts`, `keycloak`: Defines which tests to run (can be combined)
- `standalone` | `domain` | `microprofile` | `domain-hc-dc`: Defines the operation mode (mutual exclusive)
- `standalone` | `domain` | `domain-hc-dc`: Defines the operation mode (mutual exclusive). 

Combine multiple profiles to define your setup. Choose at least one profile from each line. Please note that you cannot combine profiles which are marked as mutual exclusive.

`microprofile` profile defines tests which are runnable with expansion pack (XP) which include microprofile.
`domain-hc-dc` means domain mode with master (dc) and slave (hc) host.

Examples of valid combinations:

- `chrome,basic,standalone`
- `firefox,basic,rbac,domain`
- `firefox,basic,domain-hc-dc`
- `safari,rbac,transaction,standalone`

Examples of invalid combinations:

- `safari,firefox`
- `basic,transaction`
- `standalone,domain`
- `standalone,domain-hc-dc`
- `domain,domain-hc-dc`
- `chrome,basic,standalone,domain`

## Run Tests

In order to run tests you need a running WildFly / JBoss EAP server with an insecure management interface.

The easiest way is to use the provided scripts

- `start-wildfly.sh` and
- `stop-wildfly.sh`

They use the docker image [halconsole/hal-wildfly](https://hub.docker.com/r/halconsole/hal-wildfly/) which is based
on [jboss/wildfly](https://hub.docker.com/r/jboss/wildfly/), the latest HAl console and standalone configurations with
insecure management interfaces.

If you rather want to use a custom WildFly instance, use the following commands to remove the security realm from the
management interface:

Standalone

```
/core-service=management/management-interface=http-interface:undefine-attribute(name=security-realm)
:reload
```

Domain

```
/host=master/core-service=management/management-interface=http-interface:undefine-attribute(name=security-realm)
/host=master:reload
```

Domain hc dc (on master)

```
/host=master/core-service=management/management-interface=http-interface:undefine-attribute(name=security-realm)
/host=master:reload
```

### Run all tests:

To run the tests you need to set the ``JBOSS_HOME`` property pointing to the WildFly directory. If have run the ``start-wildfly`` script, you can see the WildFly directory printed out in the console.

```bash
mvn test -P<profiles>
```

### Run all tests including product (EAP) specific ones:

```bash
mvn test -P<profiles> -Deap
```

### Run a single test:

```bash
mvn test -P<profiles> -Dtest=<fully qualified classname>
```

To debug the test(s) use the `maven.surefire.debug` property:

```bash
mvn test -P<profiles> -Dtest=<fully qualified classname> -Dmaven.surefire.debug
```

The tests will automatically pause and await a remote debugger on port 5005. You can then attach to the running tests using your IDE.
