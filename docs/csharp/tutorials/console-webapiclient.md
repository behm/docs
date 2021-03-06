---
title: REST client
description: REST client
keywords: .NET, .NET Core
author: BillWagner
manager: wpickett
ms.date: 06/20/2016
ms.topic: article
ms.prod: .net-core
ms.technology: .net-core-technologies
ms.devlang: dotnet
ms.assetid: 51033ce2-7a53-4cdd-966d-9da15c8204d2
---

# REST client

## Introduction
This tutorial teaches you a number of features in .NET Core and the C# language. You’ll learn:
*	The basics of the .NET Core Command Line Interface (CLI).
*   An overview of C# Language features.
*	Managing dependencies with NuGet
*   HTTP Communications
*   Processing JSON information
*   Managing configuration with Attributes. 

You’ll build an application that issues HTTP Requests to a REST
service on GitHub. You'll read information in JSON format, and convert
that JSON packet into C# objects. Finally, you'll see how to work with
C# objects.

There are a lot of features in this tutorial. Let’s build them one by one. 
## Prerequisites
You’ll need to setup your machine to run .NET core. You can find the
installation instructions on the [.NET Core](https://www.microsoft.com/net/core)
page. You can run this
application on Windows, Linux, macOS or in a Docker container. 
You’ll need to install your favorite code editor. The descriptions below
use [Visual Studio Code](https://code.visualstudio.com/) which is an open
source, cross platform editor. However, you can use whatever tools you are
comfortable with.
## Create the Application
The first step is to create a new application. Open a command prompt and
create a new directory for your application. Make that the current
directory. Type the command "dotnet new" at the command prompt. This
creates the starter files for a basic “Hello World” application.

Before you start making modifications, let’s go through the steps to run
the simple Hello World application. After creating the application, type
"dotnet restore" at the command prompt. This command runs the NuGet
package restore process. NuGet is a .NET package manager. This command
downloads any of the missing dependencies for your project. As this is a
new project, none of the dependencies are in place, so the first run will
download the .NET Core framework. After this initial step, you will only
need to run dotnet restore when you add new dependent packages, or update
the versions of any of your dependencies. This process also creates the
project lock file (project.lock.json) in your project directory. This file
helps to manage the project dependencies. It contains the local location
of all the project dependencies. You do not need to put the file in source
control; it will be generated when you run “dotnet restore”. 

After restoring packages, you run “dotnet build”. This executes the build
engine and creates your application. Finally, you execute “dotnet run” to
run your application.

## Adding New Dependencies
One of the key design goals for .NET Core is to minimize the size of
the .NET framework installation. The .NET Core Application framework contains
only the most common elements of the .NET full framework. This application
needs more libraries for some of its features. You'll add those
dependencies into your project.json file. You'll need to add the
`System.Net.Http` package so that your application can make HTTP requests.
You'll also need to add the `System.Runtime.Serialization.Json` package
so your application can process JSON responses.

Open your project.json file. Look for the dependencies section. You should
see one line that looks similar to this:

```
"dependencies": {
    "Microsoft.NETCore.App": {
        "type": "platform",
        "version": "1.0.0"
    }
},
```

You'll add two lines to this section to include the two new libraries:

```
"dependencies": {
   "Microsoft.NETCore.App": {
        "type": "platform"
        "version": "1.0.0",
    },
    "System.Runtime.Serialization.Json": "4.0.2",
    "System.Runtime.Serialization.Primitives": "4.1.1"
},
```

Most code editors will provide completion for different versions of these
libraries. You'll usually want to use the latest version of any package
that you add. However, it is important to make sure that the versions
of all packages match, and that they also match the version of the .NET
Core Application framework.

After you've made these changes, you should run "dotnet restore" again so
that those packages are installed on your system.

## Making Web Requests
Now you're ready to start retrieving data from the web. In this
application, you'll read information from the 
[GitHub API](https://developer.github.com/v3/). Let's read information
about the projects under the
[.NET Foundation](http://www.dotnetfoundation.org/) umbrella. You'll
start by making the request to the GitHub API to retrieve information
on the projects. The endpoint you'll use is: [https://api.github.com/orgs/dotnet/repos](https://api.github.com/orgs/dotnet/repos). You want to retrieve all the
information about these projects, so you'll use an HTTP GET request.
Your browser also uses HTTP GET requests, so you can paste that URL into
your browser to see what information you'll be receiving and processing.

You use the `HttpClient` class to make web requests. Like all modern .NET
APIs, `HttpClient` supports only async methods for its long-running APIs.
Start by making an async method. You'll fill in the implementation as you
build the functionality of the application. 

```cs
private static async Task ProcessRepositories()
{
    
}
```

You'll need to add a using statement at the top of your `Main()` method so
that the C# compiler recognizes the `Task` type:

```cs
using System.Threading.Tasks;
```

If you build your project at this point, you'll get a warning generated
for this method, because it does not contain any `await` operators and
will run synchronously. Ignore that for now, you'll add `await` operators
as you fill in the method.

Next, update the `Main()` method to call this method. The
`ProcessRepositories()` method returns a Task, and you shouldn't exit the
program before that task finishes. Therefore, you must use the `Wait()`
method to block and wait for the task to finish:

```cs
public static void Main(string[] args)
{
    ProcessRepositories().Wait();
}
```

Now, you have a program that does nothing, but does it asynchronously. Let's go back to the
`ProcessRepositories()` method and fill in a first version of it:

```cs
private static async Task ProcessRepositories()
{
    var client = new HttpClient();
    client.DefaultRequestHeaders.Accept.Clear();
    client.DefaultRequestHeaders.Accept.Add(
        new MediaTypeWithQualityHeaderValue("application/vnd.github.v3+json"));
    client.DefaultRequestHeaders.Add("User-Agent", ".NET Foundation Repository Reporter");

    var stringTask = client.GetStringAsync("https://api.github.com/orgs/dotnet/repos");

    var msg = await stringTask;
    Console.Write(msg);
}
```

You'll need to also add two new using statements at the top of the file for this to compile:

```cs
using System.Net.Http;
using System.Net.Http.Headers;
```

This first version makes a web request to read the list of all repositories under the dotnet
foundation organization. (The gitHub ID for the .NET Foundation is 'dotnet'). First, you create
a new `HttpClient`. This object handles the request and the responses. The next few lines setup
the `HttpClient` for this request. First, it is configured to accept the GitHub JSON responses.
This format is simply JSON. The next line adds a User Agent header to all requests from this
object. These two headers are checked by the GitHub server code, and are necessary to retrieve
information from GitHub.

After you've configured the `HttpClient`, you make a web request, and retrieve the response. In
this first version, you use the `GetStringAsync` convenience method. This convenience method
starts a task that makes the web request, and then when the request returns, it will read the
response stream, and extract the content from the stream. The body of the response is returned
as a `string`. The string is available when the task completes. 

The final two lines of this method await that task, and then print the response to the console.
Build the app, and run it. The build warning is gone now, because the `ProcessRepositories` now
does contain an `await` operator. You'll see a long display of JSON formatted text.   

## Processing the JSON Result

At this point, you've written the code to retrieve a response from a web server, and display
the text that is contained in that response. Next, let's convert that JSON response into C#
objects.

The JSON Serializer converts JSON data into C# Objects. Your first task is to define a C# class
type to contain the information you use from this response. Let's build this slowly, so start with
a simple C# type that contains the name of the repository:

```cs
namespace WebAPIClient
{
    public class repo
    {
        public string name;
    }
}
``` 

Put the above code in a new file called 'repo.cs'. This version of the class represents the
simplest path to process JSON data. The class name and the member name match the names used
in the JSON packet, instead of following C# conventions. You'll fix that by providing some
configuration attributes later. This class demonstrates another important feature of JSON
serialization and deserialization: Not all the fields in the JSON packet are part of this class.
The JSON serializer will ignore information that is not included in the class type being used.
This feature makes it easier to create types that work with only a subset of the fields in
the JSON packet.

Now that you've created the type, let's deserialize it. You'll need to create a
`DataContractJsonSerializer` object. This object must know the CLR type expected for the
JSON packet it retrieves. The packet from GitHub contains a sequence of repositories, so a
`List<repo>` is the correct type. Add the following line to your `ProcessRepositories` method:

```cs
var serializer = new DataContractJsonSerializer(typeof(List<repo>));
```

You're using two new namespaces, so you'll need to add those as well:

```cs
using System.Collections.Generic;
using System.Runtime.Serialization.Json;
```

Next, you'll use the serializer to convert JSON into C# objects. Replace the call to
`GetStringAsync()` in your `ProcessRepositories` method with the following two lines:

```cs
var streamTask = client.GetStreamAsync("https://api.github.com/orgs/dotnet/repos");
var repositories = serializer.ReadObject(await streamTask) as List<repo>;
```

Notice that you're now using `GetStreamAsync` instead of `GetStringAsync`. The serializer
uses a stream instead of a string as its source. Let's explain a couple features of the C#
language that are being used in the second line above. The argument to `ReadObject` is an
`await` expression. Await expressions can appear almost anywhere in your code, even though
up to now, you've only seen them as part of an assignment statement.

Secondly, the `as` operator converts from the compile time type of `object` to `List<repo>`. 
The declaration of `ReadObject` declares that it returns an object of type `System.Object`.
`ReadObject` will return the type you specified when you constructed it (`List<repo>` in
this tutorial). If the conversion does not succeed, the `as` operator evaluates to `null`,
instead of throwing an exception.

You're almost done with this section. Now that you've converted the JSON to C# objects, let's display
the name of each repository:

```cs
foreach (var repo in repositories)
    Console.WriteLine(repo.name);
```

Compile and run the application. It will print out the names of the repositories that are part of the
.NET Foundation.

## Controlling Serialization

Before you add more features, let's address the `repo` type and make it follow more standard
C# conventions. You'll do this by annotating the `repo` type with *Attributes* that control how
the JSON Serializer works. In your case, you'll use these attributes to define a mapping between
the JSON key names and the C# class and member names. The two attributes used are the `DataContract`
attribute and the `Data Member` attribute. By convention, all Attribute classes end in the suffix
`Attribute`. However, you do not need to use that suffix when you apply an attribute. 

The `DataContract` and `DataMember` attributes are in a different library, so you'll need to add
that library to project.json as a dependency. Add the following line to the dependencies section
of the project.json file (remember to add the comma separator on the line above):

```
"System.Runtime.Serialization.Primitives" : "4.0.1"
```

After you save the file, run 'dotnet restore' to retrieve this package and update the project.json.lock
file.

Next, open the repo.cs file. Let's change the name to use Pascal Case, and fully spell out the name
`Repository`. We still want to map JSON 'repo' nodes to this type, so you'll need to add the 
`DataContract` attribute to the class declaration. You'll set the `Name` property of the attribute
to the name of the JSON nodes that map to this type:

```cs
[DataContract(Name="repo")]
public class Repository
```

The `DataContractAttribute` is a member of the `System.Runtime.Serialization` namespace, so you'll
need to add the appropriate using statement at the top of the file:

```cs
using System.Runtime.Serialization;
```

You changed the name of the `repo` class to `Repository`, so you'll need to make the same name change
in Program.cs (some editors may support a rename refactoring that will make this change automatically:)

```cs
var serializer = new DataContractJsonSerializer(typeof(List<Repository>));

// ...

var repositories = serializer.ReadObject(await streamTask) as List<Repository>;
```

Next, let's make the same change with the `name` field, using the `DataMemberAttribute` class. Make
the following changes to the declaration of the `name` field in repo.cs:

```cs
[DataMember(Name="name")]
public string Name;
```

This change means you need to change the code that writes the name of each repository in program.cs:

```cs
Console.WriteLine(repo.Name);
```

Do a "dotnet build", followed by a "dotnet run" to make sure you've got the mappings correct. You should
see the same output as before. Before we process more properties from the web server, let's make one
more change to the `Repository` class. The `Name` member is a publicly accessible field. That's not
a good object oriented practice, so let's change it to a property. For our purposes, we don't need
any specific code to run when getting or setting the property, but changing to a property makes it
easier to add those changes later without breaking any code that uses the `Repository` class.

Remove the field definition, and replace it with an *auto-implemented property*:

```cs
public string Name { get; set; }
```

The compiler generates the body of the `get` and `set` accessors, as well as a private field to
store the name. It would be similar to the following code that you could type by hand:

```cs
public string Name 
{ 
    get { return this._name; }
    set { this._name = value; }
}
private string _name;
```

Let's make one more change before adding new features. The `ProcessRepositories` method can do the async
work and return a collection of the repositories. Let's return the `List<Repository>` from that method,
and move the code that writes the information into the `Main` method.

Change the signature of `ProcessRepositories` to return a task whose result is a list of `Repository`
objects:

```cs
private static async Task<List<Repository>> ProcessRepositories()
```

Then, just return the repositories after processing the JSON response:

```cs
var repositories = serializer.ReadObject(await streamTask) as List<Repository>;
return repositories;
```

The compiler generates the `Task<T>` object for the return because you've marked this method as `async`.
Then, let's modify the `Main` method so that it captures those results and writes each repository name
to the console. Your `Main` method now looks like this:

```cs
public static void Main(string[] args)
{
    var repositories = ProcessRepositories().Result;

    foreach (var repo in repositories)
        Console.WriteLine(repo.Name);
}
```

Accessing the `Result` property of a Task blocks until the task has completed. Normally, you would prefer
to `await` the completion of the task as in the `ProcessRepositories` method, but that isn't allowed in the
`Main` method.

## Reading More Information

Let's finish this by processing a few more of the properties in the JSON packet that gets sent from the
GitHub API. You won't want to grab everything, but adding a few properties will demonstrate a few more
features of the C# language.

Let's start by adding a few more simple types to the `Repository` class definition. Add these properties
to that class:

```cs
[DataMember(Name="description")]
public string Description { get; set; }

[DataMember(Name="html_url")]
public Uri GitHubHomeUrl { get; set; }

[DataMember(Name="homepage")]
public Uri Homepage { get; set; }

[DataMember(Name="watchers")]
public int Watchers { get; set; }
```

These properties have built in conversions from the string type (which is what the JSON packets contain) to
the target type. The `Uri` type may be new to you. It represents a URI, or in this case, a URL. In the case
of the `Uri` and `int` types, if the JSON packet contains data that does not convert to the target type,
the Serialization action will throw an exception.

Once you've added these, update the `Main` method to display those elements:

```cs
foreach (var repo in repositories)
{
    Console.WriteLine(repo.Name);
    Console.WriteLine(repo.Description);
    Console.WriteLine(repo.GitHubHomeUrl);
    Console.WriteLine(repo.Homepage);
    Console.WriteLine(repo.Watchers);
    Console.WriteLine();
}
```
As a final step, let's add the information for the last push operation. This information is formatted in
this fashion in the JSON response:

```
2016-02-08T21:27:00Z
```

That format does not follow one of the standard .NET DateTime formats. Because of that, you'll need to write
a custom conversion method. You also probably don't want the raw string exposed to uses of the `Repository`
class. Attributes can help control that as well. First, define a `private` property that will hold the
string representation of the date time in your `Repository` class:

```cs
[DataMember(Name="pushed_at")]
private string JsonDate { get; set; }
```

The `DataMember` attribute informs the Serializer that this should be processed, even though it is not
a public member. Next, you need to write a public read only property that converts the string to a
valid `DateTime` object, and returns that `DateTime`:

```cs
[IgnoreDataMember]
public DateTime LastPush
{
    get
    {
        return DateTime.ParseExact(JsonDate, "yyyy-MM-ddTHH:mm:ssZ", CultureInfo.InvariantCulture);
    }
}
```

Let's go over the new constructs above. The `IgnoreDataMember` attribute instructs the serializer
that this type should not be read to or written from any JSON object. This property contains only a
`get` accessor. There is no `set` accessor. That's how you define a *read only* property in C#. (Yes,
you can create *write only* properties in C#, but their value is limited.) The `DateTime.ParseExact`
method parses a string and creates a `DateTime` object to return. If the parse operation fails, the
property accessor throws an exception.

Finally, add one more output statement in the console, and you're ready to build and run this app
again:

```cs
Console.WriteLine(repo.LastPush);
```

Your version should now match the finished version located
[here](https://github.com/dotnet/docs/tree/master/samples/csharp/getting-started/console-webapiclient).
 
## Conclusion

This tutorial showed you how to make web requests, parse the result, and display properties of
those results. You've also added new packages as dependencies in your project. You've seen some of
the features of the C# language that support object oriented techniques.

