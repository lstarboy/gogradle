# Gogradle - a Full-featured Build Tool for Golang

[中文文档](./README_CN.md)

[![Build Status](https://travis-ci.org/blindpirate/gogradle.svg?branch=master)](https://travis-ci.org/blindpirate/gogradle)
[![Build Status](https://ci.appveyor.com/api/projects/status/github/blindpirate/gogradle?branch=master&svg=true)](https://ci.appveyor.com/api/projects/status/github/blindpirate/gogradle?branch=master&svg=true)
[![Coverage Status](https://coveralls.io/repos/github/blindpirate/gogradle/badge.svg?branch=master)](https://coveralls.io/github/blindpirate/gogradle?branch=master)
[![Java 8+](https://img.shields.io/badge/java-8+-4c7e9f.svg)](http://java.oracle.com)
[![Apache License 2](https://img.shields.io/badge/license-APL2-blue.svg)](http://www.apache.org/licenses/LICENSE-2.0.txt)

Gogradle is a gradle plugin which provides support for building golang.

> 2017-02-12 Now Gogradle can build 526 of [Github's top 1000 Go repositories](http://github-rank.com/star?language=Go) **WITHOUT** any extra configuration!

## Feature

- Needless to preinstall anything but `JDK 8+` (including golang itself)
- Support all versions of golang and allow their existence at the same time
- Perfect cross-platform support (as long as `Java` can be run, all tests have passed on OS X 10.11/Ubuntu 12.04/Windows 7)
- Project-scope build, needless to set `GOPATH`
- Full-featured package management
  - Needless to install dependency packages manually, all you need to do is specifying the version
  - Four VCS supported without installation: Git/Svn/Mercurial/Bazaar (Currently only Git is implemented)
  - Transitive dependency management
  - Resolve package conflict automatically
  - Support dependency lock
  - Support importing dependencies managed by various external tools such as glide/glock/godep/gom/gopm/govendor/gvt/gbvendor/trash (Based on [this report](https://github.com/blindpirate/report-of-go-package-management-tool))
  - Support [submodule](https://git-scm.com/book/en/v2/Git-Tools-Submodules)
  - Support [SemVersion](http://semver.org/)
  - Support [vendor](https://docs.google.com/document/d/1Bz5-UB7g2uPBdOx-rw5t9MxJwkfpx90cqG9AFL0JAYo)
  - Support flattening dependencies (Inspired by [glide](https://github.com/Masterminds/glide))
  - Support renaming local packages
  - Support private repository
  - `build`/`test` dependencies are managed separately
  - Support dependency tree visualization
- Support build/test/single-test/wildcard-test/cross-compile
- Modern production-grade support for automatic build, simple to define customized tasks
- Native syntax of gradle
- Additional features for users in mainland China who are behind the [GFW](https://en.wikipedia.org/wiki/Great_Firewall)
- Support shadowsocks proxy 
- IDE support (beta release for IDEA, support for VSCode and Gogland is under development)
- Support incremental build (developing)

## Highlight

- Project-scope build
- Perfect cross-platform support
- Almost all external package management supported
- Fully test coverage
- Long-term support
- Various gradle plugin to enpower your build

## Getting Started

- Install [JDK 8+](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
- Copy `gradle` directory/`gradlew` file/`gradlew.bat` file in **this project** to **the golang project to be built**
- Create a file named `build.gradle` in **the golang project to be built** with the following content:

```groovy
plugins {
    id 'com.github.blindpirate.gogradle' version '0.2.0'
}

golang {
    packagePath = 'your/package/path' // path of project to be built 
}
```

If you are using one of glide/glock/godep/gom/gopm/govendor/gvt/gbvendor/trash, then Gogradle will read the dependency lock file generated by them automatically without any extra configuration. You can generate `gogradle.lock` as its own dependency lock file. Once the file is generated, the original lock file will lose efficacy, and you can delete them. See [Dependency Lock](#dependency-lock) for more details.

## Build a Golang Project

Enter root directory of the project and run:

```
./gradlew build # *nix

gradlew build # Windows
```

Hereinafter, we will use with uniform command form `gradlew <task>` on both `*nix` and `Windows`.

The command is equivalent to `go build` in project root directory with proper `GOPATH`, and Gogradle will do all the stuff such as dependency resolution and installation. Note that Gogradle doesn't touch global `GOPATH` at all, and it will install all the denpendencies into project root directory and set environment variables to proper values automatically - it means that the build will be totally isolated and reproducible.

### Test a Golang Project

Enter root directory of the project and run:

```
gradlew test
```

If you want to specify some tests:

```
gradlew test --tests main_test.go // Specify a single test
gradlew test --tests *_test.go // Wildcard test
```

If you want to let build depend on test, just add the following line to `build.gradle`:

```groovy
build.dependsOn test
```

### Add a dependency

To add a dependency package, just add its name and version into `dependencies` block of `build.gradle`:

```groovy
dependencies {
    build 'github.com/a/b@v1.0.0' 
    test 'github.com/c/d#d3fbe10ecf7294331763e5c219bb5aa3a6a86e80'
}
```

The `build` and `test` are two independent dependency sets. Only `build` dependencies will take effect in build process; both `build` and `test` dependencies will take effect in test process.

To learn more details about dependency, see [Dependency Management](#dependency-management).

### Display Dependencies

```
gradlew dependencies
```

The output is as follows:

```
build:

github.com/gogits/gogs
├── github.com/Unknwon/cae:c6aac99 √
├── github.com/Unknwon/com:28b053d √
├── github.com/Unknwon/i18n:39d6f27 √
│   ├── github.com/Unknwon/com:28b053d √ (*)
│   └── gopkg.in/ini.v1:766e555 -> 6f66b0e
├── github.com/Unknwon/paginater:701c23f √
├── github.com/bradfitz/gomemcache:2fafb84 √
├── github.com/go-macaron/binding:4892016 √
│   ├── github.com/Unknwon/com:28b053d √ (*)
│   └── gopkg.in/macaron.v1:ddb19a9 √
│       ├── github.com/Unknwon/com:28b053d √ (*)
│       ├── github.com/go-macaron/inject:d8a0b86 -> c5ab7bf
│       └── gopkg.in/ini.v1:766e555 -> 6f66b0e (*)
... 

```

This is denpendency tree of [gogs](https://github.com/gogits/gogs) in v0.9.113. A `glide.lock` file exists in its root directory, thus Gogradle imports it automatically. It's simple, isn't it?

Tick mark `√` indicates that the dependency is the final resolved version; arrow `->` indicates that the dependency conflicts with another dependency and is resolved to another version; star `*` indicates this node's descendants are ignored because they have been displayed before.

### Dependency Lock

```
gradlew lock
```

This command will let Gogradle generate a `gogradle.lock` file in project root directory, which records all dependencies of this project. 

`gogradle.lock` is recommended by Gogradle. Locking dependencies play an important role in reproducible build. Similar to [other package management tools](https://github.com/golang/go/wiki/PackageManagementTools), Gogradle can lock the versions of all dependency packages. Moreover, Gogradle can lock dependency packages even when they are in `vendor`!

Gogradle supports transitive dependency/dependency exclusion/customized url, see [Dependency Management](#dependency-management).

Currently, only Git dependencies are supported, support for other vcs is under development.

## Configuration

The following complete configuration is located in `golang` block of `build.gradle`.

```groovy
golang {
    
    // Import path of package to be built
    packagePath = 'github.com/user/project'
    
    // The buid mode. There are two alternatives: DEVELOP/REPRODUCIBLE, DEVELOP by default
    // In DEVELOP mode, the dependency priority is:
    // 1.dependencies declared in build.gradle
    // 2.locked dependencies (in goradle.lock or lock file of other tools)
    // 3.dependencies in vendor directory
    // In REPRODUCIBLE mode, the priority is:
    // 1.dependencies in vendor
    // 2.locked dependencies
    // 3.dependencies declared in build.gradle

    mode = 'REPRODUCIBLE'
    
    // The golang version to use. See https://golang.org/dl/
    // If not specified, the latest table version is used
    goVersion = '1.7.1'
    
    // Default value is "go". Modify this when go is not in $PATH
    goExecutable = '/path/to/go/executable'
    
    // aka. build constraint. See https://golang.org/pkg/go/build/#hdr-Build_Constraints
    buildTags = ['appengine','anothertag']
    
    // Additional feature for users in mainland China,
    // ignore it if you are not
    fuckGfw = true
    
    // cache time for global cache, 24 hours by default
    globalCacheFor 24,'hours'
    
    // Extra command line arguments in build or test
    // Empty list by default
    extraBuildArgs = ['arg1','arg2']
    extraTestArgs = []

    // Location of build output, the default value is ./.gogradle
    // It can be absolute or relative (to project root)
    outputLocation = ''
    // Pattern or output, note that it must be single quote here
    outputPattern = '${os}_${arch}_${packageName}'
    // Specify output platforms in cross compile
    // Go 1.5+ required
    targetPlatform = 'windows-amd64, linux-amd64, linux-386'
}
```

## Dependency Management

Dependency management is a nightmare. Fortunately, the package management mechanism of Gogradle is excellent enough to handle complicated scenarios.

It's well known that golang doesn't manage packages at all. It assumes that all packages are located in one or more [Workspace](https://golang.org/doc/code.html#Workspaces) specified by `GOPATH`. In build, golang will find required package in `GOPATH`. This caused many issues:

- Lack of package version information makes it difficult to do reproducible and stable build.
- There may be multiple builds at the same time, and they may depend on different versions of a package.
- A project may depend on multiple versions of a package due to transitive dependencies

Therefore, golang introduced a mechanism called [vendor](https://docs.google.com/document/d/1Bz5-UB7g2uPBdOx-rw5t9MxJwkfpx90cqG9AFL0JAYo), allowing dependencies to be managed by vcs along with golang project itself. Some questions are solved, but some more arise:

- Existence of redundant code makes project fatter and fatter
- Existence of multiple versions in a project will cause [problems](https://github.com/blindpirate/golang-broken-vendor)
- Various [external package management tools](https://github.com/golang/go/wiki/PackageManagementTools) aren't compatible with each other.

Gogradle makes efforts to improve the situation. It doesn't follow golang's workspace convention, and uses totally isolated and project-grade workspace instead. All resolved packages will be placed into temp directory of project root, thus `GOPATH` is not required. 

### Dependency Declaration

You can declare dependent packages in `dependencies` block of `build.gradle`. Currently only packages managed by Git are supported. Supports for other vcs are under development.

Some examples are as follows:

```groovy
dependencies {
    build 'github.com/user/project'  // No specific version, the latest will be used
    build name:'github.com/user/project' // Equivalent to last line
    
    build 'github.com/user/project@1.0.0-RELEASE' // Specify a version(tag in Git)
    build name:'github.com/user/project', tag:'1.0.0-RELEASE' // Equivalent to last line
    build name:'github.com/user/project', version:'1.0.0-RELEASE' // Equivalent to last line
    
    test 'github.com/user/project#d3fbe10ecf7294331763e5c219bb5aa3a6a86e80' // Specify a commit
    test name:'github.com/user/project', commit:'d3fbe10ecf7294331763e5c219bb5aa3a6a86e80' // Equivalent to last line
}
```

[SemVersion](http://semver.org/) is supported in dependency declaration. In Git, a "version" is just a tag. Gogradle doesn't recommend to use SemVersion since it may break reproducibility of build.

```groovy
dependencies {
    build 'github.com/user/project@1.*'  // Equivalent to >=1.0.0 & <2.0.0
    build 'github.com/user/project@1.x'  // Equivalent to last line
    build 'github.com/user/project@1.X'  // Equivalent to last line

    build 'github.com/user/project@~1.5' // Equivalent to >=1.5.0 & <1.6.0
    build 'github.com/user/project@1.0-2.0' // Equivalent to >=1.0.0 & <=2.0.0
    build 'github.com/user/project@^0.2.3' // Equivalent to >=0.2.3 & <0.3.0
    build 'github.com/user/project@1' // Equivalent to 1.X or >=1.0.0 & <2.0.0
    build 'github.com/user/project@!(1.x)' // Equivalent to <1.0.0 & >=2.0.0
    build 'github.com/user/project@ ~1.3 | (1.4.* & !=1.4.5) | ~2' // Very complicated expression
}
```

You can specify a url in declaration, which is extremely useful in case of private repository. See [Repository Management](#repository-management) for more details.

```groovy
dependencies {
    build name: 'github.com/user/project', url:'https://github.com/user/project.git', tag:'v1.0.0'
    build name: 'github.com/user/project', url:'git@github.com:user/project.git', tag:'v2.0.0'
}
```

Multiple dependencies can be declared at the same time:

```groovy
dependencies {
    build 'github.com/a/b@1.0.0', 'github.com/c/d@2.0.0', 'github.com/e/f#commitId'
    
    build([name: 'github.com/g/h', version: '2.5'],
          [name: 'github.com/i/j', commit: 'commitId'])
}
```

Gogradle provides support for transitive dependencies. For example, the following declaration excludes transitive dependencies of `github.com/user/project`.

```groovy
dependencies {
    build('github.com/user/project') {
        transitive = false
    }
}
```

What's more, you can exclude some specific transitive dependencies. For example, the following declaration excludes all `github.com/c/d` and `github.com/e/f` in a specific version:

```groovy
dependencies {
    build('github.com/a/b') {
        exclude name:'github.com/c/d'
        exclude name:'github.com/c/d', tag: 'v1.0.0'
    }
}
```

If you have some packages in local directory, you can declare them with:

```groovy
dependencies {
    build name: 'a/local/package', dir: 'path/to/local/package' // It must be absolute
}
```

### Build Dependency and Test Dependency

You may notice that there are always `build` or `test` in dependency declarations. It's a term named [Configuration](https://docs.gradle.org/current/userguide/dependency_management.html#sub:configurations) in Gradle. Gogradle predefined `build` and `test` configuration, which you can see as two independent dependencies set. In build, only `build` dependencies will take effect; in test, both of them will take effect and dependencies in `build` have higher priority.

### Dependency Package Management

There are four kinds of dependency package in Gogradle:

- Package managed by vcs
- Package located in local file system
- Package in vendor directory
- Package imported in go source code

There isn't dependency package in golang's world, golang just treat a ordinary directory as a package. In Gogradle, a dependency package is usually root directory or a repo managed by vcs. For example, all go files in a repository managed by Git belong to one dependency package. Gogradle resolve the package path by [the default golang way](https://golang.org/cmd/go/#hdr-Relative_import_paths).

### Dependency Resolution

Dependency resolution is the process in which a dependency is resolved to some concrete code. This process usually relies on vcs such as Git. The ultimate goal of Gogradle is providing support for all four vcs (Git/Mercurial/Svn/Bazaar) with pure Java implementation. Currently only Git is supported. 

### Transitive Dependency

The dependency of a dependency (transitive dependency) can be from:

- dependencies in vendor directory
- dependencies in lock files
- dependencies in `import` of go source code

By default, Gogradle will read dependencies in vendor directory and lock files as transitive dependencies. If the result set is empty, `import` statement in `.go` source code will be scanned as transitive dependencies.

### Dependency Conflict

In practice, the situation may be extremely complicated due to the existence of transitive dependencies.

When a project depends multiple versions of one package, we say they are conflicted. For example, A depends B in version 1 and C, and C depends B in version 2, then version 1 and version 2 of B is conflicted. Golang's vendor mechanism allow them to exist at the same time, which is opposed by Gogradle. It brings [problems](https://github.com/blindpirate/golang-broken-vendor) sooner or later. Gogradle will resolve all conflict and flatten them, i.e., Gogradle assure that there is only one version for a package in the final build. The final dependencies will be placed into a temp directory in project root.

The conflict resolution strategy is:

- First level package always wins: dependencies declared in project to be built have higher priority

- Newer package wins: newer package have priority over old ones

In detail, Gogradle will detect "update time" of every dependency, and use that time to resolve conflicts.

- Update time of package managed by vcs is the commit time.
- Update time of package in local file system is the last modified time of directory
- Update time of package in vendor directory is determined by its "host" dependency.

### Dependency Lock

Gogradle can lock the dependencies in current build. A file named `gogradle.lock` recording all version information of dependency packages is generated in this task. It can make subsequent build stable and reproducible. Under no circumstances should this file be modified manually. Gogradle encourage to check in this file into vcs.

You can use

```
gradlew lock
```

to generate this file.

### Install Dependencies into Vendor

Vendor mechanism is introduced by golang 1.5. It is fully supported but not encouraged by Gogradle. To install dependencies into vendor directory, run:

```
gradlew vendor
```

This task will copy all resolved `build` dependencies into vendor directory. Note that `test` dependencies won't be copied.

## Task in Gogradle

A task unit executed independently is usually called [Task](https://docs.gradle.org/current/userguide/more_about_tasks.html). Gogradle predefined the following tasks:

- prepare
- resolveBuildDependencies
- resolveTestDependencies
- dependencies
- installBuildDependencies
- installTestDependencies
- build
- test
- clean
- check
- lock
- vendor

### prepare

Do some preparation, for example, verifying `build.gradle` and installing golang executables.

### resolveBuildDependencies/resolveTestDependencies

Resolve `build` and `test` dependencies to dependency trees. Conflicts will also be resolved in this task.

### dependencies

Display the dependency tree of current project. It's very useful when you need to resolve package conflict manually.

### installBuildDependencies/installTestDependencies

Flatten resolved `build` and `test` dependencies and install them into `.gogradle` directory respectively so that the future build can use them.

### build

Do build. This task is equivalent to:

```
cd <project path>
export GOPATH=<build dependencies path>
go build -o <output path> 
```

### test

Do test. This task is equivalent to:

```
cd <project path>
export GOPATH=<build dependencies path>:<test dependencies path>
go test
```

### check

This task is usually executed by CI to do some checking, such as test coverage rate. It depends on test task by default.

### clean

Clean temp files in project.

### lock

Generate dependency lock file. See [Dependency Lock](#dependency-lock)

### vendor

Install resolved `build` dependencies into vendor directory. See [Install Dependencies into Vendor](#install-dependencies-into-vendor). 

## Build Output and Cross Compile

By default, Gogradle will place the build output into `${projectRoot}/.gogradle` directory and name it `${os}_${arch}_${packageName}`. You can change this behaviour in configuration, See [Configuration](#Configuration).


Go1.5+ introduce convenient [cross compile](https://dave.cheney.net/2015/08/22/cross-compilation-with-go-1-5), which enable Gogradle to output results available on multiple platform in a single build.

```
golang {
    ...
    targetPlatform = 'windows-amd64, linux-amd64, linux-386'
    ...
}
```

The configuration above indicates that three results should be outputted by current build. 

## Repository Management 

Gogradle supports private repository. You can declare repositories in `repositories` block of `build.gradle`.

By default, Gogradle will read `~/.ssh` when operating on git repositories. If your private key is placed somewhere else, the following configuration can be used:

```
repositories {
    git {
        all()
        credentials {
            privateKeyFile '/path/to/private/key'
        }
    }
}
```

You may want to apply different credentials to some repositories, as follows:

```
repositories{
    git {
        url 'http://my-repo.com/my/project.git'
        credentials {
            username ''
            password ''
        }

    git {
        name 'import/path/of/anotherpackage'
        credentials {
            privateKeyFile '/path/to/private/key'
        }
    }    
}
```

In the DSL above, `name` and `url` can be any object other than string. Gogradle will use built-in Groovy method [`Object.isCase()`](http://mrhaki.blogspot.jp/2009/08/groovy-goodness-switch-statement.html) to test if it matches the declaration.

For example, you can use regular expressions:

```
    git {
        url ~/.*github\.com.*/
        credentials {
            privateKeyFile '/path/to/private/key'
        }
    }
```

If a repository matches a declaration, then the credential in the declaration will be used. Currently only Git repository is supported, you can use username/password or ssh private key as credentials.

Support for other vcs repositories are under development.

## Set Proxy For Build 

To set a proxy, you can add arguments in `gradlew`: 

```./gradlew build -DsocksProxyHost=127.0.0.1 -DsocksProxyPort=1080```

And it is the same for other `gradlew` command.

Also, you can persist the arguments via `~/.gradle/gradle.properties` or `${projectRoot}/gradle.properties`:

```
org.gradle.jvmargs=-DsocksProxyHost=127.0.0.1 -DsocksProxyPort=1080
```

To learn more about environment and proxy, see [Gradle environment](https://docs.gradle.org/current/userguide/build_environment.html#sec:gradle_properties_and_system_properties) and [Java proxy](http://docs.oracle.com/javase/6/docs/technotes/guides/net/proxies.html)

## IDE Integration

There are many IDEs supporting golang since it is static-type, e.g., [VSCode](https://github.com/Microsoft/vscode-go)/[IDEA](https://github.com/go-lang-plugin-org/go-lang-idea-plugin)/[Gogland](https://www.jetbrains.com/go/).

Usually, these IDEs ask user to set `GOPATH` and prepare dependent package there before developing. Gogradle try to make it easier. Ideally, a user should be able to start development after checking out the code immediately, without understanding or setting anything.

This is in plan.

## Contributing

If you like Gogradle, star it please.

Please feel free to submit an [issue](https://github.com/blindpirate/gogradle/issues/new).

Fork and [PR](https://github.com/blindpirate/gogradle/pulls) are always welcomed.
