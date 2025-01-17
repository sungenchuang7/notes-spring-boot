# notes-spring-boot

# Spring vs Spring Boot
`Spring` is an umbrella project that covers several sub-projects such as `Spring Boot`, *Spring AI*, *Spring Framework* etc..

Spring Boot is an extension of *Spring Framework*, which is more complicated and requires more configuration than Spring Boot. Spring Boot is used to create stand-alone, production-grade Spring-based applications with minimal effort. 

An Spring Boot application runs inside an *Inversion of Control* (IoC) **container**. Spring Boot uses *Dependency Injection* and manages class instances (objects) inside the container. 

When using DI, developers won't have to use `new` keyword to create class objects. Instead, **annotations** (`"@"`) are used to tell Spring Boot what classes need to be instantiated for the application. 

To tell Spring Boot a class should be managed inside the container, `@Component` should be placed above the class declaration. For example, 
```
public interface Computer {
    public void compile();
}

@Component("customBeanNameForDesktop")
public class Desktop implements Computer {
    public void compile() {
        System.out.println("desktop is compiling some programs");
    }
}
```

The entry point of a Spring Boot application is `SpringApplication.run(MyAppApplication.class, args);` This method returns a `ConfigurableApplicationContext`, which can be used to refer to the container and extract objects from it. 

Consider the following code, say we have a `Dev` class, modelling a software developer:
```
@Component
public class Dev {
    @Autowired // field injection
    @Qualifier("customBeanNameForDesktop")
    Computer computer;

    public void build() {
        computer.compile();
        System.out.println("I'm a dev. I'm building sth");
    }
}
```
A Dev can `build()` stuff. Say we want to have a Dev in our Spring Boot app to build stuff, we can do:

```
@SpringBootApplication
public class MyAppApplication {
	public static void main(String[] args) {
		ApplicationContext context = SpringApplication.run(MyAppApplication.class, args);
		Dev dev1 = context.getBean(Dev.class);
		dev1.build();
	}
}
```

Here, we extract the `ApplicationContext`, which is a reference to the container managing all our objects of classes labelled as `@Component`. Notice inside `Dev.build()` we actually need an instance of `Computer` to call its `compile()` method. So we need to use `@Autowire` to tell Spring Boot that an object of `Computer` needs to also be injected and managed since `Dev` depends on it. 

Since `Computer` is an interface, Spring Boot will try to find any implementation of `Computer` to instantiate and inject. Since we've annotated the `Laptop` class with `@Component`, Spring Boot will use it. 

In the case of ambiguity, say, there are multiple classes implementing `Computer`, Spring Boot will be confused about which implemention to instantiate and inject. We can use `@Qualifier(<beanName>)` to qualify our intention. Every component's default bean name is the class name with the first letter lowercased. For example, our `Desktop` component's default bean name is `desktop`. However, we can also overwrite it by specifying the custom bean name in the annotation, as seen in the code `@Component("customBeanNameForDesktop")`. 

Another way to avoid ambiguity is to attach a `@Primary` annotation to one of the implementation components that we want to be managed by the container, but we need to be careful to annotate **only one** component as the primary, or Spring Boot will get confused again. 

Another thing to talk about is the different ways for **Dependency Injection**.
There are three main ways to tell Spring Boot to inject dependencies (that a component depends on) for us. 
* Field Injection
* Constructor Injection
* Setter Injection

Consider the following code: 
``` 
@Component
public class Dev {
    @Autowired // field injection
    @Qualifier("customBeanNameForDesktop")
    Computer computer;

    public void build() {
        computer.compile();
        System.out.println("I'm a dev. I'm building sth");
    }
}
```
We know the `ApplicationContext` (the container) will already inject and manage `Dev` class object(s) for us. However, since `Dev` depend on `Computer`, so we need to annotate it with `@Autowire` to tell Spring Boot to further inject objects of type `Computer`. In the `Dev` class, `Computer computer` is a field, and when we add `@Autowire` above the field, it's called **Field Injection**. It's actually not recommended to do it this way.

The preferred way is to use **Constructor Injection**, which will look like this:
``` 
@Component
public class Dev {
    @Qualifier("customBeanNameForDesktop")
    Computer computer;

    public Dev(Computer computer) { // constructor injection
        this.computer = computer;
    }

    public void build() {
        computer.compile();
        System.out.println("I'm a dev. I'm building sth");
    }
}
```
The nice thing about Spring Boot is that if a class only has a single constructor defined, we don't need to use the `@Autowire` anontation. Spring Boot will just know to further inject any dependencies the constructor requires. 

The third way to inject a dependency is **Setter Injection**. An example of that is:
```
@Component
public class Dev {

    @Qualifier("customBeanNameForDesktop")
    Computer computer;

    @Autowired // setter injection
    public void setComputer(Computer computer) {
        this.computer = computer;
    }

    public void build() {
        computer.compile();
        System.out.println("I'm a dev. I'm building sth");
    }
}
```