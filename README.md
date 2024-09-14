# Purpose
This repository is a collection of Groovy scripts and Gradle tasks that are used with [docToolchain](https://github.com/docToolchain/docToolchain)
to import (_export_ Task) data from 3rd party datasource and publish transformed or generated content to data sinks.

A combination of all task steps could look like
![pipeline][0]


# Requirements
Tested with:
- dtcw 0.51
- docToolchain 3.4.4
- Java 17

# Usage
The easiest way to use the tasks is to configure the additional custom task path in the `docToolchainConfig.groovy`:
````
customTasks = [
  'scripts/convertToDocBook.gradle',
  'scripts/convertToMarkdown.gradle',
  'scripts/convertToMediawiki.gradle',
  'scripts/exportGithubIssues.gradle',
  'scripts/publishToMediawiki.gradle',
  'scripts/publishToOpenProject.gradle',
]
````

# GitHub Issue Exporter
The Gradle task `exportGithubIssues` can be used to connect to [GitHub.com](https://www.github.com) and retrieve issues from a repository.
Because it is an _export_ task it generates asciidoctor source files in the `src` folder that can then be embedded into other asciidoctor
documents. The task searches for open and closed issues and creates three files if these issues exist: `allGitHubIssues.adoc`,
`closedGitHubIssues.adoc` and `openGitHubIssues.adoc`.

Embedded into handwritten documentation this could look like:
![githubissues][1]

## Configure & Run
Beside the registration of the task in the `customTasks` section a custom GitHub configuration needs to be added of the same file:
````
github = [:]
github.with {
    user = "${System.getenv('GITHUB_USER')}"
    password = "${System.getenv('GITHUB_PASSWORD')}"
    root = "https://api.github.com/"
    organization = "<your organization name>"
    repository = "<your repo name>"
    resultsPerPage = 100
}
````

After that you can call the wrapper like:
````
$ ./dtcw exportGithubIssues
````
to generate the snippets.

# Docbook Converter
The Gradle task `convertToDocBook` is used to convert asciidoctor files into OASIS standardized Docbook (version 5.0) format. The task
expects source files in the DocToolchain `${srcDir}` folder (usually `./src`) and converts them into Docbook files, stored in the
DocToolchain `${targetDir}/docbook` folder (usually `./build/docbook`}).

| Asciidoctor (Source) | Docbook (Target) |
|----------------------|------------------|
| ![asciidoc][2]       | ![docbook][3]    |

Notice: Images are not embedded (as base64 encoded data) and are file references in the output document, because further converters in the
DocToolchain pipeline might convert images or not on their own.

## Configure & Run
Beside the registration of the task in the `customTasks` section no additional configuration is required to run the task:
````
$ ./dtcw convertToDocBook
````
to convert all asciidoctor files in `${srcDir}` into Docbook files in `${targetDir}/docbook`.

# Markdown Converter
Existing Docbook files can be converted into various formats, one of them is [Markdown](https://daringfireball.net/projects/markdown) that
is used by a lot of wikis and online authoring tools. To achieve the conversion, the converter uses [pandoc](https://pandoc.org) to convert
all files in `${targetDir}/docbook` to `${targetDir}/md`.

| Docbook (Source) | Markdown (Target) |
|------------------|-------------------|
| ![docbook][3]    | ![markdown][4]    |

Notice: When images in source documents come as file references, the output Markdown file will also refer to a file. If the image is 
embedded, then it is also embedded in Markdown format.

## Configure & Run
Beside the registration of the task in the `customTasks` section no additional configuration is required to run the task:
````
$ ./dtcw convertToMarkdown
````
to convert all Docbook files in `${targetDir}/docbook` into Markdown files in `${targetDir}/md`.

# Mediawiki Converter
Existing Docbook files can also be converted into [Mediawiki format](https://www.mediawiki.org). The converter is very similar to the
[Markdown Converter][] and uses [pandoc](https://pandoc.org) to convert all files in `${targetDir}/docbook` to `${targetDir}/mw`.

| Docbook (Source) | Mediawiki (Target) |
|------------------|--------------------|
| ![docbook][3]    | ![mediawiki][5]    |

Notice: When images in source documents come as file references, the output Mediawiki file will also refer to a file. If the image is
embedded, then it is also embedded in Mediawiki format.

## Configure & Run
Beside the registration of the task in the `customTasks` section no additional configuration is required to run the task:
````
$ ./dtcw convertToMediawiki
````
to convert all Docbook files in `${targetDir}/docbook` into Mediawiki files in `${targetDir}/mw`.

# Publish to Mediawiki
Previously generated Mediawiki documents can be uploaded to a Mediawiki server using the existing Mediawiki web api. The gradle task to do
so, wraps a [python script](https://github.com/SoerenKemmann/mediawiki-utils) (credits to [SoerenKemmann](https://github.com/SoerenKemmann))
that does the heavy lifting.

![mediawikiserver][6]

## Task Requirements
- Python3 must be installed on the machine that is executing the task
- [Python scripts](https://github.com/SoerenKemmann/mediawiki-utils) must be downloaded and placed under `${docDir}/bin`

## Configure & Run
Beside the registration of the task in the `customTasks` section, additional configuration is required to run the task:
````
mediawiki = [:]
mediawiki.with {
    api = "<your mediawiki server>/api.php"
    user = "${System.getenv('MEDIAWIKI_USER')}"
    password = "${System.getenv('MEDIAWIKI_PASSWORD')}"
    extension = ".mw"
    context = "<Name of the context page to store all generated documentation>"
    page = "<Name of the page where the documentation shall be placed in>"
}
````

````
$ ./dtcw publishToMediawiki
````
to upload all previously generated files in `${targetDir}/mw` to the Mediawiki server.

[0]: res/images/dtc-pipeline.drawio.png
[1]: res/images/screenshot.png
[2]: res/images/asciidoctor.png
[3]: res/images/docbook.png
[4]: res/images/markdown.png
[5]: res/images/mediawiki.png
[6]: res/images/mediawikiserver.png

