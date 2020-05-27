# Git Changelog Plugin

[![Build Status](https://ci.jenkins.io/job/Plugins/job/git-changelog-plugin/job/master/badge/icon)](https://ci.jenkins.io/job/Plugins/job/git-changelog-plugin)

Creates a changelog, or release notes, based on Git commits between 2 revisions.

This can also be done with a [command line tool](https://github.com/tomasbjerre/git-changelog-command-line).

# Usage

You can use this plugin either in a **pipeline** or as a **post-build action**.

There is a complete running example available here: https://github.com/tomasbjerre/jenkins-configuration-as-code-sandbox

## Pipeline

The plugin is compatible with the [pipeline plugin](https://jenkins.io/doc/book/pipeline/getting-started/) and can be configured to support many use cases. You probably want to adjust it using the [Snippet Generator](https://jenkins.io/doc/book/pipeline/getting-started/#snippet-generator).

The `gitChangelog` step can return:

 * **Context** - An object that contains all the information needed to create a changelog. Can be used to gather information from Git like committers, emails, issues and much more.
 * **String** - A string that is a rendered changelog, ready to be published. Can be used to publish on build summary page, on a wiki, emailed and much more.

The template and context is [documented here](https://github.com/tomasbjerre/git-changelog-lib).

It can integrate with issue management systems to get titles of issues and links. You will probably want to avoid specifying credentials in plain text in your script. One way of doing that is using the [credentials binding plugin](https://jenkins.io/doc/pipeline/steps/credentials-binding/). The supported integrations are:

 * GitLab
 * GitHub
 * Jira

You can [create a file](https://jenkins.io/doc/pipeline/examples/) or maybe publish the changelog with:

 * [HTML Publisher Plugin](https://plugins.jenkins.io/htmlpublisher)
 * [Confluence Publisher Plugin](https://plugins.jenkins.io/confluence-publisher)
 * [Email Extension](https://plugins.jenkins.io/email-ext)

You can filter out a subset of the commits by:

 * Specifying specific **from**/**to** **references**/**commits**.
 * Adding filter based on message.
 * Adding filter based on commit time.
 * Filter tags based on tag name.
 * Filter commits based on commit time.
 * Ignore commits that does not contain an issue.

You can make the changelog prettier by:

 * Transforming tag name to something more readable.
 * Changing date display format.
 * Creating virtual tag, that contains all commits that does not belong to any other tag. This can be named something like *Unreleased*.
 * Creating virtual issue, that contains all commits that does not belong to any other issue.
 * Remove issue from commit message. This can be named something like *Wall of shame* and list all committers that did not commit on an issue.

Check the [Snippet Generator](https://jenkins.io/doc/book/pipeline/getting-started/#snippet-generator) to see all features!

### Pipeline with context

Here is an example that clones a repo, gathers all jiras and adds a link to jira in the description of the job. The context contains much more then this and is [documented here](https://github.com/tomasbjerre/git-changelog-lib).

```groovy
node {
 deleteDir()
 sh """
 git clone git@github.com:jenkinsci/git-changelog-plugin.git .
 """
    
 def changelogContext = gitChangelog returnType: 'CONTEXT',
  from: [type: 'REF', value: 'git-changelog-1.50'],
  to: [type: 'REF', value: 'master'],
  jira: [issuePattern: 'JENKINS-([0-9]+)\\b', password: '', server: '', username: '']

 Set<String> issueIdentifiers = new TreeSet<>()
 changelogContext.issues.each { issue ->
  if (issue.name == 'Jira') {
   issueIdentifiers.add(issue.issue)
  }
 }
 currentBuild.description = "http://jira.com/issues/?jql=key%20in%20%28${issueIdentifiers.join(',')}%29"
}
```

### Pipeline with string

Here is an example that clones a repo and publishes the changelog on job page. The template and context is [documented here](https://github.com/tomasbjerre/git-changelog-lib).

```groovy
node {
 deleteDir()
 sh """
 git clone git@github.com:jenkinsci/git-changelog-plugin.git .
 """
    
 def changelogString = gitChangelog returnType: 'STRING',
  from: [type: 'REF', value: 'git-changelog-1.50'],
  to: [type: 'REF', value: 'master'],
  template: """
  <h1> Git Changelog changelog </h1>

<p>
Changelog of Git Changelog.
</p>

{{#tags}}
<h2> {{name}} </h2>
 {{#issues}}
  {{#hasIssue}}
   {{#hasLink}}
<h2> {{name}} <a href="{{link}}">{{issue}}</a> {{title}} </h2>
   {{/hasLink}}
   {{^hasLink}}
<h2> {{name}} {{issue}} {{title}} </h2>
   {{/hasLink}}
  {{/hasIssue}}
  {{^hasIssue}}
<h2> {{name}} </h2>
  {{/hasIssue}}


   {{#commits}}
<a href="https://github.com/tomasbjerre/git-changelog-lib/commit/{{hash}}">{{hash}}</a> {{authorName}} <i>{{commitTime}}</i>
<p>
<h3>{{{messageTitle}}}</h3>

{{#messageBodyItems}}
 <li> {{.}}</li> 
{{/messageBodyItems}}
</p>


  {{/commits}}

 {{/issues}}
{{/tags}}
  """

 currentBuild.description = changelogString
}
```

## Post-build action

When the plugin is installed, it will add a new post build action in Jenkins job configuration.

 * **Git Changelog** - Implements features from [git-changelog-lib](https://github.com/tomasbjerre/git-changelog-lib).

### Git Changelog

A couple of revisions are configured along with some other optional features. A editable template is available for the user to tweak. 

![Select references](/doc/imgs/git-changelog-references.png)

The changelog is created from parsing Git and rendering the template with a context derived from the configured revisions.

![Tweak template](/doc/imgs/git-changelog-file.png)

# Development

This plugin can be built and started with maven and Jenkins' hpi plugin:

```
./run.sh
```

The functionality is implemented in [git-changelog-lib](https://github.com/tomasbjerre/git-changelog-lib). Pull requests are welcome!


