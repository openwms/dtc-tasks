# Purpose
This repository is a collection of Groovy scripts and Gradle tasks that are used with [docToolchain](https://github.com/docToolchain/docToolchain)
to import (_export_ Task) data from 3rd party datasource and publish transformed or generated content to data sinks.

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
DocToolchain `${targetDir}` folder (usually `./build`}).

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
to convert all asciidoctor files in `${srcDir}` into Docbook files in `${targetDir}`.

[1]: res/images/screenshot.png
[2]: res/images/asciidoctor.png
[3]: res/images/docbook.png

