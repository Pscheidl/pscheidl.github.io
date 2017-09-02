---
title: "Integrating JavaFX 8 with Guice dependency injection"
published: true
categories:
  - Java
tags:
  - Java
  - JavaFX
  - Dependency Injection
  - GUI
---

JavaFX 8 provides an easy way to build GUIs. This post will try to explain how to combine JavaFX 8 with almost any dependency injection framework, providing an example with capable and popular library Google Guice. Making JavaFX 8 cooperate with a dependency injection container is fairly easy.

Complete, functional example can be viewed & downloaded at <a href="https://github.com/Pscheidl/JavaFX8DependencyInjection" target="_blank">Github</a>.

### Basic application without DI

In order to instantiate a JavaFX 8 application without dependency injection, certain steps need to be taken.

1. Extend javafx.application.Application & call launch method on that class from the main method. This is the application's entry point.
2. In application's start method, instantiate FXMLLoader, give it an .fxml file resource. This will create content of the newly created window.
3. Give the Parent object instantiated in previous step to Stage object (usually called primaryStage) supplies as an argument to the start(Stage primaryStage) method.
4. Display the primaryStage by calling it's show() method.

**Example**

{% highlight java %}
package cz.pscheidl.blog.javafxdi;

import java.io.IOException;
import java.io.InputStream;
import javafx.application.Application;
import javafx.fxml.FXMLLoader;
import javafx.scene.Parent;
import javafx.scene.Scene;
import javafx.stage.Stage;

public class JavaFX8App extends Application {

    public static void main(String[] args) {
        JavaFX8App.launch(args);
    }

    @Override
    public void start(Stage primaryStage) throws Exception {
        FXMLLoader loader = new FXMLLoader();

        try (InputStream fxmlInputStream = ClassLoader.getSystemResourceAsStream("cz/pscheidl/blog/gui/gui.fxml")) {
            Parent parent = loader.load(fxmlInputStream);
            primaryStage.setScene(new Scene(parent));
            primaryStage.show();
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }
}
{% endhighlight %}

 ## Adding dependency injection

There is one more obvious step need compared to previous example - to instantiate the dependency injection container. This happens before step two. Then, in step two, a callback to dependency injection container is given to the FXMLLoader instance via it's public void setControllerFactory(Callback<Class<?>,Object> controllerFactory) method. And that's all.

1. Extend javafx.application.Application & call launch method on that class from the main method. This is the application's entry point.
2. Instantiate dependency injection container of your choice. E.g. Google Guice or Weld.
3. In application's start method, instantiate FXMLLoader and set it's controller factory to obtain controllers from the container. Ideally obtain the FXMLLoader from the container itself, using a provider. Then give it an .fxml file resource. This will create content of the newly created window.
4. Give the Parent object instantiated in previous step to Stage object (usually called primaryStage) supplies as an argument to the start(Stage primaryStage) method.
5. Display the primaryStage by calling it's show() method.

In practice, it looks like this.

{% highlight java %}

**Example**

package cz.pscheidl.blog.javafxdi;

import com.google.inject.Guice;
import com.google.inject.Injector;
import cz.pscheidl.blog.javafxdi.guice.GuiceModule;
import java.io.IOException;
import java.io.InputStream;
import javafx.application.Application;
import javafx.fxml.FXMLLoader;
import javafx.scene.Parent;
import javafx.scene.Scene;
import javafx.stage.Stage;

public class JavaFXDI extends Application {

    public static void main(String[] args) {
        //Launch the application - public void start(Stage primaryStage) will be called next.
        JavaFXDI.launch(args);
    }

    @Override
    public void start(Stage primaryStage) throws Exception {
        Injector injector = Guice.createInjector(new GuiceModule());
        FXMLLoader fxmlLoader = new FXMLLoader();
        //The new part. Give fxmlLoader a callback. Controllers will now be instatiated via the container, not FXMLLoader itself.
        fxmlLoader.setControllerFactory(instantiatedClass -> {
            return injector.getInstance(instantiatedClass);
        });

        try (InputStream fxmlInputStream = ClassLoader.getSystemResourceAsStream("cz/pscheidl/blog/javafxdi/JavaFXDI.fxml")) {
            Parent parent = fxmlLoader.load(fxmlInputStream);
            primaryStage.setScene(new Scene(parent));
            primaryStage.setTitle("JavaFX 8 Dependency injection");
            primaryStage.show();
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }
}
{% endhighlight %}

As you can see, there is only a slight difference ! By adding a callback, the FXMLLoader instantiates the controller (place dependencies are and should be declared) by calling the container of your choice. In this case, it's Google Guice. Remember that the CONTAINER HAS TO KNOW HOW TO INSTANTIATE THE CONTROLLER. With Google Guice everything works almost out of the box, since it needn't know anything about your controller in the module. Just about the hypothetical dependencies.

Complete, functional example can be viewed & downloaded at <a href="https://github.com/Pscheidl/JavaFX8DependencyInjection" target="_blank">Github</a>.