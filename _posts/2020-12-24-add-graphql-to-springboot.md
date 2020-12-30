---
layout: post
title: "add graphql to springboot"
# date: 2020-12-24
tags: [ "SpringBoot", "GraphQL", "API" ]
---
<p>Add GraphQl to Spring-boot
The example in this blog post is implemented using Spring-boot version 2.3.4.RELEASE. The following dependencies are added to my maven project:

{% highlight xml %}
        <dependency>
            <groupId>com.graphql-java</groupId>
            <artifactId>graphql-java</artifactId>
            <version>16.1</version>
        </dependency>
        <dependency>
            <groupId>com.graphql-java-kickstart</groupId>
            <artifactId>graphiql-spring-boot-starter</artifactId>
            <version>8.0.0</version>
        </dependency>
        <dependency>
            <groupId>com.graphql-java</groupId>
            <artifactId>graphql-java-spring-boot-starter-webmvc</artifactId>
            <version>2.0</version>
        </dependency>
{% endhighlight %}

<p>

```xml
        <dependency>
            <groupId>com.graphql-java</groupId>
            <artifactId>graphql-java</artifactId>
            <version>16.1</version>
```    
</p>




