# Package
>Package is the collection of source code that is compiled together. This is the Go's way to structure your code

Meaning, package is the *directory/folder of source code*. As the name implies, it helps to pack your files to organise your project.

There are 2 types of packages:
1. Main: package that contains a main function that is an entry point of a program
2. Non-main: the rest of packages, that should be ***focused on specific functionality*** 



# Module
>Modules are how Go manages packages and dependencies
>Module is the collection of [packages](#package) that are released, *versioned and distributed **together*** .

>Module stores related packages for a discrete and useful set of function.
>For example, you can create a module with packages that contains financial analytics functionality, so others can use your module in their financial application. Basically it's like a library or a package in a flutter

*Before modules there was a $GOPATH that forces us to locate all in the project to a specific path* 

Module provides an information about dependency requirements and dependency versioning
- dependency requirements: specifies what dependency needed and where to find it by specifying a path that consist of: host, the user and a project name
- dependency versioning: clarifying the requirements by specifying the specific version

### creating a module that others can use
creating module by `go mod init [path]`.
If you publish your module, then your path ***must*** be a path from which your module can be downloaded by Go tools. That would be your code's repository, for example in [github](https://github.com)

https://go.dev/doc/tutorial/create-module

If a module contains a folder `foo`, then this package can be imported by `modulepath/foo`

### module defining  guidelines

>It's recommended to use a module path that corresponds to a repository you plan or will publish your module to, so when you do, `go get` will be able to automatically fetch, build and install your module. 
>For example you may choose a module path `github.com/bob/hello`, so when you publish your module, everyone can get it by simply using `import "github.com/bob/hello"` in their app.

> Also note that you don't need to publish your code to a remote repo before you can build it. 
> But it's still recommended to follow this pattern so you'll have less work to make it work in the future if you decide to publish it. Nothing to lose here.

---
## go.mod
go.mod describes:
1. the path to its own project, called [[module path]]
2. the Go version
3. the path to the dependencies and the versions

## go.sum
Auto-generated file that contains:
1. dependencies of the dependencies that we imported
2. checksum of the integrity of dependencies

> **NOTE**: dependencies will be automatically added to go.mod and go.sum, when you add a external modules with `go get` command
  
# reference
[the most information that i've taken from](https://www.youtube.com/watch?v=ihZ9tQoqX-A)