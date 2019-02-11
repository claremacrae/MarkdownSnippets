<!--
This file was generate by the MarkdownSnippets.
Source File: \readme.source.md
To change this file edit the source file and then re-run the generation using either the dotnet global tool (https://github.com/SimonCropp/MarkdownSnippets#githubmarkdownsnippets) or using the api (https://github.com/SimonCropp/MarkdownSnippets#running-as-a-unit-test).
-->

# <img src="https://raw.githubusercontent.com/SimonCropp/MarkdownSnippets/master/src/icon.png" height="40px"> MarkdownSnippets

Extract code snippets from any language to be used when building documentation

Loosely based on some code from  https://github.com/shiftkey/scribble


## Using Snippets

The keyed snippets can then be used in any documentation `.md` file by adding the text `snippet: KEY`.

Then snippets with the key (all versions) will be rendered in a tabbed manner. If there is only a single version then it will be rendered as a simple code block with no tabs.

For example

<pre>
Some blurb about the below snippet
snippet&#58; MySnippetName
</pre>

The resulting markdown will be will be:

    Some blurb about the below snippet
    ```
    My Snippet Code
    ```


## Defining Snippets


### Using comments

Any code wrapped in a convention based comment will be picked up. The comment needs to start with `startcode` which is followed by the key. The snippet is then terminated by `endcode`.

```
// startcode MySnippetName
My Snippet Code
// endcode
```


### Using regions

Any code wrapped in a named C# region will be picked up. The name of the region is used as the key.

```
#region MySnippetName
My Snippet Code
#endregion
```


### Code indentation

The code snippets will do smart trimming of snippet indentation.

For example given this snippet:

<pre>
&#8226;&#8226;#region MySnippetName
&#8226;&#8226;Line one of the snippet
&#8226;&#8226;&#8226;&#8226;Line two of the snippet
&#8226;&#8226;#endregion
</pre>

The leading two spaces (&#8226;&#8226;) will be trimmed and the result will be:

```
Line one of the snippet
••Line two of the snippet
```

The same behavior will apply to leading tabs.


### Do not mix tabs and spaces

If tabs and spaces are mixed there is no way for the snippets to work out what to trim.

So given this snippet:

<pre>
&#8226;&#8226;#region MySnippetNamea
&#8226;&#8226;Line one of the snippet
&#10137;&#10137;Line one of the snippet
&#8226;&#8226;#endregion
</pre>

Where &#10137; is a tab.

The resulting markdown will be will be

<pre>
Line one of the snippet
&#10137;&#10137;Line one of the snippet
</pre>

Note none of the tabs have been trimmed.


### Ignore paths

When scanning for snippets the following are ignored:

 * All directories and files starting with a period `.`
 * All binary files as defined by https://github.com/sindresorhus/binary-extensions/
 * Any of the following directory names: `bin`, `obj`

To change these conventions manipulate lists `MarkdownSnippets.Exclusions.ExcludedDirectorySuffixes` and `MarkdownSnippets.Exclusions.ExcludedFileExtensions`.


## The NuGet package [![NuGet Status](http://img.shields.io/nuget/v/MarkdownSnippets.svg?style=flat)](https://www.nuget.org/packages/MarkdownSnippets/)

https://nuget.org/packages/MarkdownSnippets/

    PM> Install-Package MarkdownSnippets


## Api Usage


### Reading snippets from files

<!-- snippet: ReadingFilesSimple -->
```cs
var files = Directory.EnumerateFiles(@"C:\path", "*.cs", SearchOption.AllDirectories);

var snippets = FileSnippetExtractor.Read(files);
```
<sup>[snippet source](/src/Tests/Snippets/Usage.cs#L8-L14)</sup>
<!-- endsnippet -->


### Reading snippets from a directory structure

<!-- snippet: ReadingDirectorySimple -->
```cs
// extract snippets from files
var snippetExtractor = new DirectorySnippetExtractor(
    // all directories except bin and obj
    directoryFilter: dirPath => !dirPath.EndsWith("bin") && !dirPath.EndsWith("obj"),
    // all js and cs files
    fileFilter: filePath => filePath.EndsWith(".js") || filePath.EndsWith(".cs"));
var snippets = snippetExtractor.ReadSnippets(@"C:\path");
```
<sup>[snippet source](/src/Tests/Snippets/Usage.cs#L34-L44)</sup>
<!-- endsnippet -->


### Full Usage

<!-- snippet: markdownProcessingSimple -->
```cs
// setup version convention and extract snippets from files
var snippetExtractor = new DirectorySnippetExtractor(
    directoryFilter: x => true,
    fileFilter: s => s.EndsWith(".js") || s.EndsWith(".cs"));
var snippets = snippetExtractor.ReadSnippets(@"C:\path");

// Merge with some markdown text
var markdownProcessor = new MarkdownProcessor(
    snippets: snippets,
    appendSnippetGroup: SimpleSnippetMarkdownHandling.AppendGroup);

using (var reader = File.OpenText(@"C:\path\inputMarkdownFile.md"))
using (var writer = File.CreateText(@"C:\path\outputMarkdownFile.md"))
{
    var result = markdownProcessor.Apply(reader, writer);
    // snippets that the markdown file expected but did not exist in the input snippets
    var missingSnippets = result.MissingSnippets;

    // snippets that the markdown file used
    var usedSnippets = result.UsedSnippets;
}
```
<sup>[snippet source](/src/Tests/Snippets/Usage.cs#L49-L73)</sup>
<!-- endsnippet -->


## GitHubMarkdownSnippets

A [.NET Core Global Tool](https://docs.microsoft.com/en-us/dotnet/core/tools/global-tools) for merging snippets into GitHub markdown document.


### Target directory

The target directory can be defined via one of the following:

 * Passed in as a single argument; or
 * The current directory is used. Only if it exists with a git repository, that is a directory tree that contains a directory names `.git`.


### Behavior

 * Recursively scan the [target directory](#target-directory) for alln on [ignored files](#ignore-paths) for snippets.
 * Recursively scan the [target directory](#target-directory) for all `*.source.md` files.
 * Merge the snippets with the `.source.md` to produce `.md` files. So for example `readme.source.md` would be merged with snippets to produce `readme.md`. Note that this process will overwrite any existing `.md` files that have matching `.source.md` pairs.


### Installation

To install use:

```ps
dotnet tool install -g GitHubMarkdownSnippets
```

To update use:

```ps
dotnet tool update -g GitHubMarkdownSnippets
```

To uninstall use:

```ps
dotnet tool uninstall -g GitHubMarkdownSnippets
```


### Usage

For running in the current directory:

```ps
mdsnippets
```

For running against a target directory:

```ps
mdsnippets C:\Code\TheTargetDirectory
```


### Running as a unit test

The above functionality can also be achieved via a unit test via using the API.

For the git repository containing the unit test file:

<!-- snippet: GitHubMarkdownProcessorRunForFilePath -->
```cs
GitHubMarkdownProcessor.RunForFilePath();
```
<sup>[snippet source](/src/Tests/Snippets/Usage.cs#L19-L23)</sup>
<!-- endsnippet -->

For a specific directory:

<!-- snippet: GitHubMarkdownProcessorRun -->
```cs
GitHubMarkdownProcessor.Run("targetDirectory");
```
<sup>[snippet source](/src/Tests/Snippets/Usage.cs#L25-L29)</sup>
<!-- endsnippet -->


## Icon

Icon courtesy of [The Noun Project](http://thenounproject.com) and is licensed under Creative Commons Attribution as: 

> ["Down"](https://thenounproject.com/AlfredoCreates/collection/arrows-5-glyph/) by [Alfredo Creates](https://thenounproject.com/AlfredoCreates) from The Noun Project