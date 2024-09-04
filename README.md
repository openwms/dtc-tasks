# Purpose
This repository is a collection of Groovy scripts and Gradle tasks that are used with [docToolchain](https://github.com/docToolchain/docToolchain)
to import (_export_ Task) data from 3rd party datasource and publish transformed or generated content to data sinks.

# GitHub Issue Exporter
The Gradle task `exportGithubIssues` can be used to connect to [GitHub.com](https://www.github.com) and retrieve issues from a repository.
Because it is an _export_ task it generates asciidoctor source files in the `src` folder that can then be embedded into other asciidoctor
documents. The task searches for open and closed issues and creates three files if these issues exist: `allGitHubIssues.adoc`,
`closedGitHubIssues.adoc` and `openGitHubIssues.adoc`.

Embedded into handwritten documentation this could look like:
![githubissues][1]

## Usage

Tested with:
- dtcw 0.51
- docToolchain 3.4.4
- Java 17

The easiest way to use the tasks is to configure the additional custom task path in the `docToolchainConfig.groovy`:
````
customTasks = [
  'scripts/exportGithubIssues.gradle'
]
````

and also add you custom GitHub configuration to the end of the same file:
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


[1]: res/images/screenshot.png

