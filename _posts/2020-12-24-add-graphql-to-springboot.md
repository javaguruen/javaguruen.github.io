---
layout: post
title: "add graphql to springboot"
# date: 2020-12-24
tags: [ "SpringBoot", "GraphQL", "API" ]
---
<p>Add GraphQl to Spring-boot
The example in this blog post is implemented using Spring-boot version 2.3.4.RELEASE. The following dependencies are added to my maven project:

<h2>Dependencies</h2>
Note that the versions of these dependencies can have changed by the time your read this.

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

<h2>The schema</h2>

<p>Add the schema as a file named <code>schema.graphqls</code> in the `src/main/resources` folder of your project 

{% highlight graphql %}
type Query {
    "Søk etter whisky. Kan søke på navn og produsent"
    soek(name: String): [Whisky]
}

type Whisky {
    id: ID!,
    datotid: String!,
    "Identificator for a product at vinmonopolet.no"
    varenummer: String!,
    varenavn: String!,
    varetype: String!,
    volum: Float!,
    land: String!,
    distrikt: String,
    underdistrikt: String,
    "Distillery"
    produsent: String!,
    priser: [Pris]!
}

type Pris{
    id: ID!,
    datotid: String!,
    varenummer: String!,
    volum: Float!,
    pris: Float!,
    literpris: Float!
}

{% endhighlight %}
</p>
