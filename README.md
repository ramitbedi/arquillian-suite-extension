The Arquillian Suite Extension
==================================

[![Build Status](https://travis-ci.org/ingwarsw/arquillian-suite-extension.png)](https://travis-ci.org/ingwarsw/arquillian-suite-extension)
[![Coverage Status](https://coveralls.io/repos/ingwarsw/arquillian-suite-extension/badge.png?branch=master)](https://coveralls.io/r/ingwarsw/arquillian-suite-extension?branch=master)

The Extension will force all Classes in a Module into a TestSuite running from the same DeploymentScenario.

Class marked with @ArquillianSuiteDeployment will be used as a 'template' for all other TestClass scenarios.

Deploy will occur only once on first test witch require Arquillian.

So far tested on:
- Jboss 7.1, Jboss 7.2
- Glassfish 3.2.2
- Should work on other servers too

# Basic Usage

Add module to test classpath.

    <dependency>
        <groupId>org.eu.ingwar.tools</groupId>
        <artifactId>arquillian-suite-extension</artifactId>
        <version>1.0.4</version>
        <scope>test</scope>
    </dependency>

Mark one of your test classes with annotation @ArquillianSuiteDeployment along with usual @Deployment annotation on method.

    @ArquillianSuiteDeployment
    public class Deployments {

        @Deployment
        public static WebArchive deploy() {
            return ShrinkWrap.create(WebArchive.class)
                .addAsWebInfResource(EmptyAsset.INSTANCE, "beans.xml")
                .addPackage(Deployments.class.getPackage());
        }
    }

Run test either from IDE or maven.

**Make sure you use servlet protocol!**

To do that add following to arquillian.xml:

    <defaultProtocol type="Servlet 3.0"/>

Or mark your deployments with annotation @OverProtocol("Servlet 3.0")

# Extended usage

## Generic builder

That project contains generic builder with should work for most simple J2EE cases.
You dont need to define deployment yourself but you can use generic builder to do work for you.

Basic usage for EJB module looks like.

        @Deployment
        public static EnterpriseArchive generateAutogeneratedDeployment() {
          EnterpriseArchive ear = EarGenericBuilder.getModuleDeployment(ModuleType.EJB);
            return ear;
        }

## Multi deployment

As with normal arquillian you can define more than one deployment and then force tests to run on one of them.

    @ArquillianSuiteDeployment
    public class Deployments {

        @Deployment(name = "normal")
        public static Archive<?> generateDefaultDeployment() {
            return ShrinkWrap.create(WebArchive.class, "normal.war")
                .addAsWebInfResource(EmptyAsset.INSTANCE, "beans.xml")
                .addClass(Deployments.class)
                .addClass(InjectedObject.class)
                .addPackage(Extension1Test.class.getPackage());
        }
    
        @Deployment(name = "extra")
        public static Archive<?> generateExtraDeployment() {
            Archive<?> ejb = ShrinkWrap.create(WebArchive.class, "extra_ejb.war")
                .addAsWebInfResource(EmptyAsset.INSTANCE, "beans.xml")
                .addClass(Deployments.class)
                .addClass(InjectedObject.class)
                .addPackage(ExtensionExtra1Test.class.getPackage());
            return ejb;
        }
    }
    
Then at test methods you need to add annotation @OperateOnDeployment("name")

    @Test
    @OperateOnDeployment("normal")
    public void testNormal() {
        // Test on normal deployment
    }
    
    @Test
    @OperateOnDeployment("extra")
    public void testExtra() {
        // Test on extra deployment
    }

# Extra info

## Credits

Most work was done by Aslak Knutsen
- https://gist.github.com/aslakknutsen/3975179
- Part was added by ItCrowd team.

I just mixed things up..

## Continuous integration

Travis CI builds the plugin with Oracle and Open JDK 7. All successfully built snapshots are deployed to
Sonatype OSS repository. Jacoco is used to gather coverage metrics and the report is submitted
to Coveralls.


## License

The project is licensed under the Apache License, Version 2.0.


[![Bitdeli Badge](https://d2weczhvl823v0.cloudfront.net/ingwarsw/arquillian-suite-extension/trend.png)](https://bitdeli.com/free "Bitdeli Badge")

