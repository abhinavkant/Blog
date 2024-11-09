---
layout: post
title: "Inversion of Control and Dependency Injection"
---
> In 1995, when the Gang of Four wrote Design Patterns,2 this was already common knowledge: *Program to an interface, not an implementation.*

> Inversion of control is a common characteristic of frameworks, so saying that these lightweight containers are special because they use inversion of control is like saying my car is special because it has wheels.

> Inversion of Control is too generic a term, and thus people find it confusing. As a result with a lot of discussion with various IoC advocates we settled on the name Dependency Injection.

ref: https://martinfowler.com/articles/injection.html#InversionOfControl

So the idea is to create a **loosely coupled** code. Design the code such that
* separating concerns
* binary references of its dependents has been reduced.
* code maintainability without adding complexity.
* increase testability.

> Loose couping enables you to write code which is [open for extensibility, but closed for modification]. This is called the **OPEN/CLOSED PRINCIPLE**.

Example of a tight coupled code

```csharp
public class Service
{
    public void Serve()
    {
        Console.WriteLine("Service Invoked");
    }
}

public class Client
{
    public Service _service;

    public Client()
    {
        _service = new Service();
    }

    public void StartService()
    {
        Console.WriteLine("Service Started");
        _service.StartService();
        //To Do: Some Stuff
    }
}

class Program
{
    static void Main(string[] args)
    {
        var c = new Client();
        c.Start();
    }
}
```

Above code we are bound to face issues:
* Client class needs to be aware of "Service" class.
* Any changes in Service method may result in changing the class consuming the it.
* The code is almost impossible to testable.
* It is difficult to switch out Service class and replace it with new implementation.

Let's refactor the code:

```csharp
public interface IService
{
    void Serve();
}

public class Service : IService
{
    public void Serve()
    {
        Console.WriteLine("Service Invoked");
    }
}

public class Client
{
    private readonly IService _service;

    public Client(IService service)
    {
        _service = service;
    }

    public void StartService()
    {
        Console.WriteLine("Service Started");
        this._service.Serve();
        //To Do: Some Stuff
    }
}

class Program
{
    static void Main(string[] args)
    {
        var c = new Client(new Service());
        c.StartService();
    }
}
```

Now lets see what have we achieved:
* We have made sure Client class only refers to the signature of IService rather than the concrete implementation.
* Code is more flexible, we can switch the implementation of IService when Client class is initialized a
* Client does not have to change.
* Code is testable.

Benefits gained from loose coupling:
* Late binding
* Extensibility
* Parallel development
* Maintainability
* Testability

## Ways to achieve IoC/DI:

### Constructor Injection

```csharp
public interface IService
{
    void Serve();
}

public class Service : IService
{
    public void Serve()
    {
        Console.WriteLine("Service Called");
        //To Do: Some Stuff
    }
}

public class Client
{
    private IService _service;

    public Client(IService service)
    {
        this._service = service;
    }

    public void StartService()
    {
        Console.WriteLine("Service Started");
        this._service.Serve();
        //To Do: Some Stuff
    }
}
class Program
{
    static void Main(string[] args)
    {
        var client = new Client(new Service());
        client.StartService();

        Console.ReadKey();
    }
}
```
### Property Injection

```csharp
public interface IService
{
    void Serve();
}

public class Service : IService
{
    public void Serve()
    {
        Console.WriteLine("Service Called");
        //To Do: Some Stuff
    }
}

public class Client
{
    private IService _service;

    public IService Service
    {
        set
        {
            this._service = value;
        }
    }

    public void StartService()
    {
        Console.WriteLine("Service Started");
        this._service.Serve();
        //To Do: Some Stuff
    }
}

class Program
{
    static void Main(string[] args)
    {
        Client client = new Client();
        client.Service = new Service();
        client.StartService();

        Console.ReadKey();
    }
}
```

### Method Injection

```csharp
public interface IService
{
    void Serve();
}

public class Service : IService
{
    public void Serve()
    {
        Console.WriteLine("Service Called");
        //To Do: Some Stuff
    }
}

public interface IClient
{
    void Start();
}

public class Client
{
    public void StartService(IService service)
    {
        Console.WriteLine("Service Started");
        service.Serve();
        //To Do: Some Stuff
    }
}
class Program
{
    static void Main(string[] args)
    {
        Client client = new Client();
        client.StartService(new Service());

        Console.ReadKey();
    }
}
```

The most convenient way to achieve DI/IoC is to use constructor injection
##  IoC/DI Container

A framework to create dependencies and inject them automatically when required. It automatically creates objects based on request and inject them when required. DI Container helps us to manage dependencies with in the application in a simple and easy way.

### IoC/DI Frameworks:
* Autofac

    1. Register the instance

    ```csharp
    var builder = new ContainerBuilder();
    builder.RegisterType<Service>().As<IService>();
    builder.RegisterType<Client>().As<IClient>();
    var container = builder.Build();
    ```

    2. Resolve and instance

    ```csharp
    var client = resolver.Resolve<IClient>();
    client.Start();
    ```

* StructureMap
* Unity
* NInject
* Create your own (for understanding only)
```csharp
public class IoC
{
    readonly Dictionary<Type, Type> types = new Dictionary<Type, Type>();

    public void Register<TContract, TImplementation>()
    {
        types[typeof(TContract)] = typeof(TImplementation);
    }

    public T Resolve<T>()
    {
        return (T)Resolve(typeof(T));
    }

    public object Resolve(Type contract)
    {
        Type implementation = types[contract];

        ConstructorInfo constructor = implementation.GetConstructors()[0];
        ParameterInfo[] constructorParameters = constructor.GetParameters();

        if (constructorParameters.Length == 0)
        {
            return Activator.CreateInstance(implementation);
        }

        List<object> parameters = new List<object>(constructorParameters.Length);
        foreach (ParameterInfo parameterInfo in constructorParameters)
        {
            parameters.Add(Resolve(parameterInfo.ParameterType));
        }

        return constructor.Invoke(parameters.ToArray());
    }
}
```
