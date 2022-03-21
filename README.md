# UnityDataTools

The UnityDataTool is a set of command line tools showcasing what can be done with the UnityFileSystemApi native dynamic library. The main purpose of these tools is to analyze the content of Unity data files.

The UnityFileSystemApi library is distributed in the Tools folder of the Unity editor (starting in version 2022.1.0a14). For simplicity, it is also included in
this repository. The library is backward compatible, which means that it can read data files generated by any previous version of Unity.

## What is the purpose of the UnityFileSystemApi native library?

The purpose of the UnityFileSystemApi is to expose the functionalities of the WebExtract and binary2text tools, but in a more flexible way. To fully understand
what it means, let's first discuss how Unity generates the data files in a build. The data referenced by the scenes in a build is called the Player Data.
It is a set of files called SerializedFiles containing the assets in a platform-specific format. When using AssetBundles or Addressables, things are slightly
different. Note that as Addressables are AssetBundles and we'll only use the term AssetBundle in the remaining of this document.

AssetBundles are archive files (similar to zip files) that can be mounted at runtime. They contain SerializedFiles like those in the Player Data, but they include what is called a
TypeTree<sup>[1](#footnote1)</sup>. The TypeTree is data structure exposing how objects have been serialized, i.e. the name, type and size of their properties.
It is used by Unity when loading an AssetBundle that was built with a previous Unity version (so you don't necessarily have to update all AssetBundles after
upgrading a project to a newer version of Unity).

The content of a SerializedFile including a TypeTree can be converted to a human-readable format using the binary2text tool that can be found in the Tools
folder of Unity. In the case of AssetBundles, the SerializedFiles must first be extracted using the WebExtrac tool that is also in the Tools folder. For the
Player Data, there is unfortunately no TypeTree so these tools can't be used<sup>[2](#footnote2)</sup>. The text file generated by binary2text can be very
useful to diagnose issues with a build, but they are usually very large and difficult to navigate. Because of this, a tool called the
[AssetBundle Analyzer](https://github.com/faelenor/asset-bundle-analyzer) was created to make it easier to extract useful information from these files in the
form of a SQLite database. The AssetBundle Analyzer has been quite successful but it has several issues. It is extremely slow as it runs WebExtract and
binary2text on all the AssetBundles of a project and has to parse very large text files. It can also easily fail because the syntax used by binary2text is not
standard and can even be impossible to parse in some occasions.

As mentioned earlier, the UnityFileSystemApi has been created to expose WebExtract and binary2text functionalities. This enables the creation of tools that
can read Unity data files with TypeTrees. With it, it becomes very easy to create a binary2text-like tool that can output the data in any format or a new
faster and simpler AssetBundle Analyzer.

## Repository content

The repository contains the following items:
* UnityFileSystem: source code of a .NET class library exposing the functionalities or the UnityFileSystemApi native library.
* UnityFileSystem.Tests: test suite for the UnityFileSystem library.
* UnityFileSystemTestData: the Unity project used to generate the test data.
* [UnityDataTool](UnityDataTool/README.md): a command-line tool providing several features that can be used to analyze the content of Unity data files and using the following libraries.
* [Analyzer](Analyzer/README.md): a library exposing functions that can extract key information from Unity data files and dump it into a SQLite database (similar to the [AssetBundle Analyzer](https://github.com/faelenor/asset-bundle-analyzer)).
* [TextDumper](TextDumper/README.md): a library with functions that can dump SerializedFiles in a human-readable format (similar to binary2text).
* [ReferenceFinder](ReferenceFinder/README.md): a library providing functions that can find reference chains from objects to other objects using a database created by the UnityDataAnalyzer

## How to build

The projects in this solution require the .NET 5.0 SDK. You can use your favorite IDE to build them. They were tested in Visual Studio on Windows and Rider on Mac.

It is also possible to build the projects from the CLI using this command:

`dotnet build -c Release`

Note that on Mac, you need to publish the UnityDataTool project if you want to get an executable file. You can do it from your IDE or execute this command
in the UnityDataTool folder (not from the root folder):

`dotnet publish -c Release -r osx-x64 -p:PublishSingleFile=true -p:UseAppHost=true`

Also on Mac, if you get a warning because "UnityFileSystemApi.dylib" cannot be opened because the developer cannot be verified, click "Cancel" and
then open the System Preferences -> Security & Privacy window. You should be able to allow the file from there. Alternatively, you can copy the file from the
Tools folder of a Unity 2022 installation.

## Disclaimer

This project is provided on an "as-is" basis and is not officially supported by Unity. It is an experimental tool provided as an example of what can be done using the UnityFileSystemApi. You can report bugs and submit pull requests, but there is no guarantee that they will be addressed.

---
*Footnotes*:  
<a name="footnote1">1</a>: AssetBundles include the TypeTree by default but this can be disabled by using the
[DisableWriteTypeTree](https://docs.unity3d.com/ScriptReference/BuildAssetBundleOptions.DisableWriteTypeTree.html) option.  
<a name="footnote2">2</a>: With a custom build of Unity, is it possible to generate Player Data files including TypeTrees.
