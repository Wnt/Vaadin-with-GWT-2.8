# Example project about how to use GWT 2.8 in Vaadin 7.7 applications
## To build and run this project in < 5 minutes:
Prerequisites: you have [Git](https://git-scm.com/downloads), [JDK 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) and [Maven](https://maven.apache.org/download.cgi) installed.

1. Checkout the sources `git clone https://github.com/Wnt/Vaadin-with-GWT-2.8.git`
2. Run `mvn install` in the widget-addon submodule
3. Run `mvn vaadin:update-widgetset gwt:compile` in the widget-demo submodule
4. And finally run `mvn jetty:run` in the widget-demo submodule to start Jetty on port 8888
5. Open [http://localhost:8888](http://localhost:8888) in your browser

# Tutorial on how to create a project like this:
## How to use GWT 2.8 in Vaadin applications today
GWT 2.8 brings a bunch of interesting new features and improvements for Vaadin client-side developers. Some of the most interesting features are Java 8 support and a matured [JsInterop](https://docs.google.com/document/d/10fmlEYIHcyead_4R1S5wKGs1t2I7Fnp_PaNaa7XTEk0/view). You can catch up with other new developments from the [GWT Release Notes](http://www.gwtproject.org/release-notes.html). At the time of writing this, GWT 2.8 is in its release candidate phase.

In this tutorial I’m going to show you how to use GWT 2.8 today when doing Vaadin client-side development. I’m going to use the Vaadin archetype widget as our example project. I’m using the newly released Vaadin 7.7 as that has changed the [way Vaadin depends on GWT](https://vaadin.com/download/prerelease/7.7/7.7.0/7.7.0.rc1/release-notes.html#gwtdep) and thus makes using a custom GWT version simpler than ever before.

The sample project is available on [GitHub](https://github.com/Wnt/Vaadin-with-GWT-2.8). The first five commits correspond to the sections in this tutorial so you can see exactly what was changed in each step.

## Creating a project to work with

Let’s start by creating a project to work with. You can use the Maven command line:

    mvn -B archetype:generate \
        -DarchetypeGroupId=com.vaadin \
        -DarchetypeArtifactId=vaadin-archetype-widget \
        -DarchetypeVersion=7.7.0 \
        -DgroupId=org.test \
        -DartifactId=widget \
        -Dversion=1.0-SNAPSHOT

or your IDE of choice to materialize a project.

## Change GWT dependencies

If we take a look at the addon sub-module’s dependency tree using `mvn dependency:tree`, we can see some GWT 2.7 dependencies are brought in by `com.vaadin:vaadin-client`

    $ mvn dependency:tree

    --- maven-dependency-plugin:2.8:tree (default-cli) @ widget ---
    org.test:widget:jar:1.0-SNAPSHOT
    +- com.vaadin:vaadin-server:jar:7.7.0:compile
    |  +- com.vaadin:vaadin-sass-compiler:jar:0.9.13:compile
    |  |  +- org.w3c.css:sac:jar:1.3:compile
    |  |  +- com.vaadin.external.flute:flute:jar:1.3.0.gg2:compile
    |  |  \- com.yahoo.platform.yui:yuicompressor:jar:2.4.8:compile
    |  |     \- rhino:js:jar:1.7R2:compile
    |  +- com.vaadin:vaadin-shared:jar:7.7.0:compile
    |  \- org.jsoup:jsoup:jar:1.8.3:compile
    +- com.vaadin:vaadin-client:jar:7.7.0:provided
    |  \- com.vaadin.external.gwt:gwt-elemental:jar:2.7.0.vaadin3:provided
    |     \- com.vaadin.external.gwt:gwt-user:jar:2.7.0.vaadin3:provided
    |        +- javax.validation:validation-api:jar:1.0.0.GA:provided
    |        \- javax.validation:validation-api:jar:sources:1.0.0.GA:provide
    \- junit:junit:jar:4.8.1:test
    ------------------------------------------------------------------------

These are practically the same as official GWT modules, but built by us to a different groupId. We need to exclude these dependencies and introduce equivalent GWT 2.8 dependencies. This can be done by adding an exclude rule to the `pom.xml` in the `com.vaadin:vaadin-client` section:

    <dependency>
        <groupId>com.vaadin</groupId>
        <artifactId>vaadin-client</artifactId>
        <version>${vaadin.version}</version>
        <scope>provided</scope>
        <exclusions>
                 <exclusion>
                       <groupId>com.vaadin.external.gwt</groupId>
                       <artifactId>gwt-elemental</artifactId>
                 </exclusion>
        </exclusions>
    </dependency>

And let’s introduce an equivalent GWT 2.8 dependency:

    <dependency>
        <groupId>com.google.gwt</groupId>
        <artifactId>gwt-elemental</artifactId>
        <version>2.8.0-rc2</version>
        <scope>provided</scope>
    </dependency>

Now we can build & install the project (run `mvn clean install` in the addon module) and move on to the demo project where the actual widget set compilation happens. Again we can run `mvn dependency:tree` to identify dependencies we need to change. Add the following exclude rules to the pom:

    <dependency>
        <groupId>com.vaadin</groupId>
        <artifactId>vaadin-client-compiler</artifactId>
        <scope>provided</scope>
        <exclusions>
              <exclusion>
                      <groupId>com.vaadin.external.gwt</groupId>
                      <artifactId>gwt-dev</artifactId>
                  </exclusion>
                  <exclusion>
                      <groupId>com.vaadin.external.gwt</groupId>
                      <artifactId>gwt-elemental</artifactId>
                  </exclusion>
        </exclusions>
    </dependency>

And add equivalent GWT 2.8 dependencies:

    <dependency>
        <groupId>com.google.gwt</groupId>
        <artifactId>gwt-dev</artifactId>
        <version>2.8.0-rc2</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>com.google.gwt</groupId>
        <artifactId>gwt-elemental</artifactId>
        <version>2.8.0-rc2</version>
        <scope>provided</scope>
    </dependency>

## Change to use the GWT 2.8 compiler

Normally Vaadin Maven projects use the `vaadin-maven-plugin` to compile the widgetset. We want to use the upstream `gwt-maven-plugin` instead. Let’s first configure the `vaadin-maven-plugin` to generate the widget set in a directory where the `gwt-maven-plugin` can find it. Add the following configuration section to `vaadin-maven-plugin`:

    <groupId>com.vaadin</groupId>
    <artifactId>vaadin-maven-plugin</artifactId>
    <version>${vaadin.plugin.version}</version>
    <configuration>
       <generatedWidgetsetDirectory>${basedir}/src/main/java</generatedWidgetsetDirectory>
    </configuration>

Before generating the widget set, make sure that there is no existing `*.gwt.xml` files in the demo project, as the Vaadin plugin will only create a new widget set into `src/main/java` if there is no existing widget set it can update. To generate the widget set just run `mvn vaadin:update-widgetset`.

Next we can remove or comment out the compile goal from `vaadin-maven-plugin` as we’ll use the upstream GWT compiler for that:

    <goal>resources</goal>
    <!-- <goal>compile</goal> -->
    <goal>update-widgetset</goal>
    Last modification we need to do to the pom.xml is to add the gwt-maven-plugin build plugin:
    <plugin>
       <groupId>org.codehaus.mojo</groupId>
       <artifactId>gwt-maven-plugin</artifactId>
       <version>2.8.0-rc2</version>
       <configuration>
           <webappDirectory>${basedir}/target/classes/VAADIN/widgetsets</webappDirectory>
       </configuration>
       <executions>
           <execution>
               <goals>
                   <goal>compile</goal>
               </goals>
           </execution>
       </executions>
    </plugin>

## Test GWT compilation and run Jetty

Now you are ready to run `mvn clean gwt:compile` to compile the widget set. After the widget set is compiled you can run the demo UI using `mvn jetty:run` and access the test UI at [http://localhost:8080/](http://localhost:8080/).

## Use new GWT 2.8 features and recompile the widget set

Now for the fun part! We can start to utilize new GWT 2.8 features e.g. by simply changing the `ClickHandler` in `MyComponentConnector` from

    getWidget().addClickHandler(new ClickHandler() {
        public void onClick(ClickEvent event) {
           final MouseEventDetails mouseDetails = MouseEventDetailsBuilder
                   .buildMouseEventDetails(event.getNativeEvent(),
                           getWidget().getElement());
           // When the widget is clicked, the event is sent to server with ServerRpc
           rpc.clicked(mouseDetails);
        }
    });

into a lambda expression:

    getWidget().addClickHandler(event -> {
        final MouseEventDetails mouseDetails = MouseEventDetailsBuilder
               .buildMouseEventDetails(event.getNativeEvent(),
                       getWidget().getElement());
        rpc.clicked(mouseDetails);
    });

After making changes to the addon project, you follow the normal workflow of `mvn clean install` in the addon project followed by a `mvn clean gwt:compile` in the demo project.

I hope this tutorial helped you to get started with GWT 2.8 development. The next major Vaadin version, Vaadin 8, will use [GWT 2.8 by default](https://github.com/vaadin/vaadin/blob/master/pom.xml#L28). This means that any Java 8 and JsInterop code you write today should be reusable in Vaadin 8 without the tricks listed in this tutorial.

The sample project with all the modifications described in this post is available on [GitHub](https://github.com/Wnt/Vaadin-with-GWT-2.8). Go ahead and import that to your IDE and start your own GWT 2.8 adventure!
