---
layout: post
title: Dependency Injection on Android - Getting Started with Dagger 2
---

To understand what dependency injection is, we have to first understand the dependency inversion principle.

![Dependency Injection]({{ site.baseurl }}/assets/images/dagger.gif "Dependency Injection")

Dependency inversion, the D in the SOLID object-oriented programming principles, refers to the decoupling of dependencies in code by relying on abstractions instead of implementation details.

Dependency injection is the mechanism we use to support dependency inversion. Essentially our code is provided or "injected" with dependencies without having to manage the creation and implementation of those dependencies. In Android we can do this with a library called Dagger 2.

#### Why use Dependency Injection?

Because dependencies are decoupled, there are many benefits to using dependency injection

* we can change out dependency implementations without affecting the entire system
* we can inject mock implementations when testing to make sure we're testing the right parts of our code
* components are modular and can be easily reused

### Setting up Dagger 2

#### Set up the libraries
In this example we'll be adding Realm to our project and injecting it as a dependency.

First we add the Realm plugin to our project-level `build.gradle`

{% highlight php %}
classpath 'io.realm:realm-gradle-plugin:2.3.1'
{% endhighlight %}

At the top of the app-level `build.gradle` apply the realm-android plugin

{% highlight php %}
apply plugin: 'realm-android'
{% endhighlight %}

In the same file add the Dagger dependencies.

{% highlight php %}
compile 'com.google.dagger:dagger:2.9'
annotationProcessor 'com.google.dagger:dagger-compiler:2.9'
{% endhighlight %}

Sync Gradle and you should have both Realm and Dagger ready to use.

#### Modules

We have to tell Dagger how we satisfy dependencies. We do that with the `@Provides` annotation inside of modules (which are denoted with the `@Module` annotation).

First we'll create the `DatabaseModule` class and provide Realm.

{% highlight java %}
@Module
public class DatabaseModule {

    @Provides
    Realm provideRealm(Context context) {
        Realm.init(context);
        RealmConfiguration.Builder builder = new RealmConfiguration.Builder();
        builder.name("daggerdemo.realm");

        RealmConfiguration config = builder.build();
        return Realm.getInstance(config);
    }
}
{% endhighlight %}

You might have noticed that `DatabaseModule` relies on Context. That's because it's a dependency! Let's provide Context in our `ApplicationModule`. It's fairly straightforward. 

{% highlight java %}
@Module
public class ApplicationModule {
    private Application application;

    public ApplicationModule(Application application) {
        this.application = application;
    }

    @Provides
    public Context provideContext() {
        return application;
    }
}
{% endhighlight %}

Note that we could have provided Context in the `DatabaseModule`, but put it in the `ApplicationModule` for modularity.

#### Components

Next we have to tell Dagger where to inject the dependencies. This is done in components using the `@Component` annotation.

{% highlight java %}
@Component(modules = {ApplicationModule.class, DatabaseModule.class})
@Singleton
public interface ApplicationComponent {

    void inject(MainActivity activity);
}
{% endhighlight %}

In this interface we're telling Dagger to inject the modules and the dependencies provided by them into the `MainActivity`. Note that `ApplicationModule` is a singleton and Dagger handles that for us with the `@Singleton` annotation.

In order to create our component we have to use a bit of Dagger's generated code, so build the project now.

Now we can use our newly generated `DaggerApplicationComponent`. Let's wire it up in our Application.

{% highlight java %}
public class DaggerDemoApplication extends Application {

    private ApplicationComponent component;

    public ApplicationComponent getComponent() {
        if (component == null) {
            component = DaggerApplicationComponent.builder()
                    .applicationModule(new ApplicationModule(this))
                    .databaseModule(new DatabaseModule())
                    .build();
        }
        return component;
    }
}
{% endhighlight %}

Here we add the modules to the `DaggerApplicationComponent` using the builder interface's applicationModule() and databaseModule() methods.

The final step to complete injection is done in the `MainActivity`. First define our request for a dependency. This is done with the `@Inject` annotation on the Realm property. Next we have to invoke the inject method we defined in `ApplicationComponent`.


{% highlight java %}
public class MainActivity extends AppCompatActivity {

    @Inject
    Realm realm;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ((DaggerDemoApplication) getApplication()).getComponent()
                .inject(this);
    }
}
{% endhighlight %}

Now we can use the newly injected Realm.

In this post we learned how to use Dagger 2 in Android. We learned that Dagger 2 is a library for dependency injection, which itself is a mechanism to support dependency inversion.

### Further Reading

* [Google's Documentation on Dagger 2][docs]
* [Dagger 2 source on Github][source]
* [Wiki entry for Dependency Inversion Principle][di]
* [Wiki entry for SOLID][solid]

[docs]:	//google.github.io/dagger/users-guide
[source]: //github.com/google/dagger
[di]: //en.wikipedia.org/wiki/Dependency_inversion_principle
[solid]: //en.wikipedia.org/wiki/SOLID_(object-oriented_design)



