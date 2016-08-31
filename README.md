# Example project about how to use GWT 2.8 in Vaadin 7.7 applications
## To build and run this project in < 5 minutes:
Prerequisites: you have [Git](https://git-scm.com/downloads), [JDK 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) and [Maven](https://maven.apache.org/download.cgi) installed.

1. Checkout the sources `git clone https://github.com/Wnt/Vaadin-with-GWT-2.8.git`
2. Run `mvn install` in the widget-addon submodule
3. Run `mvn vaadin:update-widgetset gwt:compile` in the widget-demo submodule
4. And finally run `mvn jetty:run` in the widget-demo submodule to start Jetty on port 8888
5. Open [http://localhost:8888](http://localhost:8888) in your browser
