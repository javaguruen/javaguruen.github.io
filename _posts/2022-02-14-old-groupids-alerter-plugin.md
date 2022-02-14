---
layout: post
title: "Old groupIds alerter (OGA) maven plugin"
# date: 2022-02-14
tags: [ "Maven", "OGA" ]
---

The github page for this project describes the plugin as:
> "A Maven plugin that checks for deprecated groupId+artifactId (e.g. did you
   know that graphql-spring-boot-starter moved from `com.graphql-java` to
   `com.graphql-java-kickstart`?)."

Why are these important to catch? Well, if you check your maven project for
newer versions of your dependencies at a regular interval, you'll miss out
of all new bugfixes, security patches and new features as these are released
as new artifacts with a different groupId and artifactId.

The [README.md](https://github.com/jonathanlermitage/oga-maven-plugin) at github offers a simple overview over possible options and configurations.
I will not copy their README here, but the simple usage is:
Run 
```
mvn biz.lermitage.oga:oga-maven-plugin:check
```
The plugin will fail on the
first error with text like:
```
[ERROR] (dependency) 'org.apache.httpcomponents:httpclient'
    should be replaced by 'org.apache.httpcomponents.client5:httpclient5'
    (context: Maven group id changed to ‘org.apache.httpcomponents.client5’.)
```

As this is a transitive dependency managed by Spring-Boot, I will not change it before the Spring project changes it's dependencies. I can ignore this one by adding
it to a local ignore list, like `ogaIgnore.json`:
```json
{
    "ignoreList": [
        {
            "item": "org.apache.httpcomponents:httpclient"
        }
    ]
}
```

Now rerun the plugin 
```
mvn biz.lermitage.oga:oga-maven-plugin:check 
-DignoreListFile=ogaIgnore.json
```

The plugin also has options for an alternative list of renamed artifacts and
all configuration can be given commandline or in the `plugin` configuration in the `pom.xml` file.