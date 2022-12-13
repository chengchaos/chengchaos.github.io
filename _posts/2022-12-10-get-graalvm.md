---
title: 安装 GraalVM
key: 2022-12-10
tags: GraalVM java
---

> 建议关键字 java GraalVM graalvm

Summarize

Omitted ...

<!--more-->

## 0x01 官网

<https://www.graalvm.org/downloads/>

## 0x02 安装

```bash
bash <(curl -sL https://get.graalvm.org/jdk)
Downloading graalvm-ce-java17-darwin-amd64-22.3.0.tar.gz...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100  248M  100  248M    0     0  80990      0  0:53:39  0:53:39 --:--:--  465k
Verifying checksum...
Extracting...
Installing GraalVM components...
Downloading: Component catalog from www.graalvm.org
Processing Component: Native Image
Downloading: Component native-image: Native Image from github.com
Installing new component: Native Image (org.graalvm.native-image, version 22.3.0)

+------------------------------------------------------------------------------+
| Prerequisites for GraalVM Native Image                                       |
+------------------------------------------------------------------------------+
| If you are using macOS Catalina and later you may need to remove the         |
| quarantine attribute before use:                                             |
| $ sudo xattr -r -d com.apple.quarantine "/Users/chengchao/graalvm-ce-java17-22.3.0/Contents/Home" |
|                                                                              |
| Please also ensure Xcode is installed:                                       |
| $ xcode-select --install                                                     |
+------------------------------------------------------------------------------+

GraalVM JDK (graalvm-ce-java17-22.3.0) has been downloaded successfully!

Run the following to set your $GRAALVM_HOME, $PATH, and/or $JAVA_HOME:
$ export GRAALVM_HOME="/Users/chengchao/graalvm-ce-java17-22.3.0/Contents/Home"
$ export PATH="/Users/chengchao/graalvm-ce-java17-22.3.0/Contents/Home/bin:$PATH"
$ export JAVA_HOME="/Users/chengchao/graalvm-ce-java17-22.3.0/Contents/Home"

```

```bash
$ gu
GraalVM Component Updater v2.0.0

Usage: 
    gu info [-cClLnprstuvVJ] <param>        print info about a specific component (from a file, a URL or a catalog)
    gu available [-aClvVJ] <expr>           list components available in a catalog
    gu install [-0CcDfiLMnosruvyxY] <param> install a component package
    gu list [-clvJ] <expression>            list installed components, or components from a catalog
    gu remove [-0DfMxv] <id>                uninstall a component
    gu upgrade [-cCnLsuxSd] [<ver>] [<cmp>] upgrade to a recent GraalVM or edition
    gu rebuild-images                       rebuild native executables. Use -h for detailed usage

Common options:
  -A, --auto-yes              respond YES or ACCEPT to all questions.
  -c, --catalog               treat parameters as component IDs from a catalog of GraalVM components. This is the default.
  -C, --custom-catalog <url>  use user-supplied catalog at URL.
  -e, --debug                 debugging. Prints stacktraces, ...
  -E, --no-catalog-errors     do not stop if at least one catalog is working.
  -h, --help                  print help.
  -L, --local-file, --file    treat parameters as local filenames of packaged components.
  -N, --non-interactive       noninteractive mode. Fail when input is required.
  --show-version              print version information and continue.
  -u, --url                   interpret parameters as URLs of packaged components.
  -v, --verbose               be verbose. Print versions and dependency information.
  --version                   print version.

Additonal options:
    --email <address>         email address that confirms acceptance of GDS license. 
                              can only be used with some commands.
    --config <path>           path to configuration file
    --show-ee-token           print current download token
    
Use 
    gu <command> -h
to get specific help.

Runtime options:
  --native                                     Run using the native launcher with limited access to Java libraries
                                               (default).
  --jvm                                        Run on the Java Virtual Machine with access to Java libraries.
  --vm.[option]                                Pass options to the host VM. To see available options, use '--help:vm'.
  --log.file=<String>                          Redirect guest languages logging into a given file.
  --log.[logger].level=<String>                Set language log level to OFF, SEVERE, WARNING, INFO, CONFIG, FINE,
                                               FINER, FINEST or ALL.
  --help                                       Print this help message.
  --help:vm                                    Print options for the host VM.
$ gu list
ComponentId              Version             Component name                Stability                     Origin 
---------------------------------------------------------------------------------------------------------------------------------
graalvm                  22.3.0              GraalVM Core                  Supported                     
native-image             22.3.0              Native Image                  Early adopter                 github.com

```

## 0x03 测试

写一段简单的 Java 代码： 

```

    public class Test {

        public static void main(String[] args) {
            System.out.println("Hello GraalVM !");
        }
    }
```

编译

```bash


$ java -version
openjdk version "17.0.5" 2022-10-18
OpenJDK Runtime Environment GraalVM CE 22.3.0 (build 17.0.5+8-jvmci-22.3-b08)
OpenJDK 64-Bit Server VM GraalVM CE 22.3.0 (build 17.0.5+8-jvmci-22.3-b08, mixed mode, sharing)

$ javac Test.java 

$ java Test
Hello GraalVM !


$ native-image Test
========================================================================================================================
GraalVM Native Image: Generating 'test' (executable)...
========================================================================================================================
[1/7] Initializing...                                                                                    (7.6s @ 0.10GB)
 Version info: 'GraalVM 22.3.0 Java 17 CE'
 Java version info: '17.0.5+8-jvmci-22.3-b08'
 C compiler: cc (apple, x86_64, 14.0.0)
 Garbage collector: Serial GC
[2/7] Performing analysis...  [*****]                                                                   (12.6s @ 0.63GB)
   2,802 (73.54%) of  3,810 classes reachable
   3,360 (50.75%) of  6,621 fields reachable
  12,344 (42.84%) of 28,815 methods reachable
      27 classes,     0 fields, and   313 methods registered for reflection
      58 classes,    59 fields, and    52 methods registered for JNI access
       4 native libraries: -framework Foundation, dl, pthread, z
[3/7] Building universe...                                                                               (1.7s @ 1.10GB)
[4/7] Parsing methods...      [*]                                                                        (1.4s @ 1.73GB)
[5/7] Inlining methods...     [***]                                                                      (1.3s @ 0.60GB)
[6/7] Compiling methods...    [***]                                                                     (10.9s @ 1.53GB)
[7/7] Creating image...                                                                                  (2.2s @ 1.97GB)
   3.90MB (35.44%) for code area:     7,073 compilation units
   6.91MB (62.72%) for image heap:   97,694 objects and 5 resources
 206.47KB ( 1.83%) for other data
  11.01MB in total
------------------------------------------------------------------------------------------------------------------------
Top 10 packages in code area:                               Top 10 object types in image heap:
 658.04KB java.util                                          915.50KB java.lang.String
 324.62KB java.lang                                          870.93KB byte[] for code metadata
 264.72KB java.text                                          839.54KB byte[] for general heap data
 216.40KB java.util.regex                                    614.93KB java.lang.Class
 194.70KB java.util.concurrent                               540.57KB byte[] for java.lang.String
 146.95KB java.math                                          452.67KB java.util.HashMap$Node
 116.72KB java.lang.invoke                                   218.91KB com.oracle.svm.core.hub.DynamicHubCompanion
 114.72KB com.oracle.svm.core.genscavenge                    217.09KB java.util.HashMap$Node[]
  94.83KB java.util.stream                                   159.91KB java.lang.String[]
  94.03KB java.util.logging                                  155.02KB java.util.concurrent.ConcurrentHashMap$Node
   1.68MB for 115 more packages                                1.52MB for 767 more object types
------------------------------------------------------------------------------------------------------------------------
                        1.0s (2.5% of total time) in 17 GCs | Peak RSS: 3.06GB | CPU load: 4.93
------------------------------------------------------------------------------------------------------------------------
Produced artifacts:
 /Users/chengchao/temp/graalvm/test (executable)
 /Users/chengchao/temp/graalvm/test.build_artifacts.txt (txt)
========================================================================================================================
Finished generating 'test' in 39.4s.
$ ls
Test.class               Test.java                test                     test.build_artifacts.txt
$ ./test
Hello GraalVM !
$ time java Test
Hello GraalVM !

real    0m0.145s
user    0m0.099s
sys 0m0.044s
$ time ./test
Hello GraalVM !

real    0m0.011s
user    0m0.004s
sys 0m0.005s

```



EOF

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>


