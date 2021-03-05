---
layout: post
title: "Add graphql to springboot"
# date: 2020-12-24
tags: [ "SpringBoot", "GraphQL", "API" ]
---
<p>Add GraphQl to Spring-boot
The example in this blog post is implemented using Spring-boot version 2.4.2. Adding GraphQl to you Spring-boot is as easy as described in the following sections below. The code snippets shown here are from a "real" pet project of mine where I collect information about whiskies and track their price changes in the Norwegian Vinmonopolet.

<h2>Dependencies</h2>
For both GraphQl and the popular GraphiQl interactive editor, add the following dependencies. Note that the versions of these dependencies can have changed by the time you read this.

{% highlight xml %}
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphql-java</artifactId>
    <version>16.1</version>
</dependency>
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphql-java-spring-boot-starter-webmvc</artifactId>
    <version>2.0</version>
</dependency>
<dependency>
    <groupId>com.graphql-java-kickstart</groupId>
    <artifactId>graphiql-spring-boot-starter</artifactId>
    <version>11.0.0</version>
</dependency>
{% endhighlight %}

<h2>The schema</h2>
GraphQl defines it's own model (Schema) containing Queries, Mutators and the Types representing the data model. For most applications this can be defined in one single file. Add the schema as a file named <code>schema.graphqls</code> in the <code>src/main/resources</code> folder of your project. The following example schema shows two qweries and two types: Whisky and Pris. For more detailed information about the schema content and syntax, see https://graphql.org/learn/schema/.

Note how easy it is to document the parts in the schema. Just add one-line comments in quotes or multi-line comments inside the triple-quotes. Inside triple-quotes markdown formatting is supported.

{% highlight graphql linenos %}
type Query {
    "Søk etter whisky på navn eller destilleri"
    soek(navn: String): [Whisky]

    "Nye whiskyer og whiskyer med prisendring på siste slipp"
    sisteOppdateringer: [Whisky]
}

"""
Whisky
"""
type Whisky {
    id: ID!,
    datotid: String!,
    "vinmonopolet.no sin id på whiskyen"
    varenummer: String!,
    navn: String!,
    volum: Float!,
    land: String!,
    distrikt: String,
    alkohol:Float!,
    destilleri: String!,
    priser: [Pris]!
}

"Prisinformasjon"
type Pris {
    id: ID!,
    datotid: String!,
    varenummer: String!,
    pris: Float!,
    literpris: Float!
}

{% endhighlight %}

<h2>The GraphQl provider</h2>
We need a class to provide the GraphQl bean to spring and do the wiring of the data fetchers.

{% highlight kotlin linenos %}

import graphql.GraphQL
import graphql.schema.GraphQLSchema
import org.springframework.context.annotation.Bean
import java.io.IOException
import javax.annotation.PostConstruct
import graphql.schema.idl.SchemaGenerator
import graphql.schema.idl.RuntimeWiring
import graphql.schema.idl.SchemaParser
import graphql.schema.idl.TypeRuntimeWiring.newTypeWiring
import org.springframework.context.annotation.Configuration

@Configuration
class GraphQLProvider(
  private val graphQLDataFetchers: GraphQLDataFetchers) {

  private var graphQL: GraphQL? = null

  @Bean
  fun graphQL(): GraphQL {
    return graphQL!!
  }

  @PostConstruct
  @Throws(IOException::class)
  fun init() {
    val schema: String = this::class.java.getResource("/schema.graphqls").readText(Charsets.UTF_8)
    val graphQLSchema: GraphQLSchema = buildSchema(schema)
    graphQL = GraphQL.newGraphQL(graphQLSchema).build()
  }

  private fun buildSchema(schema: String): GraphQLSchema {
    val typeRegistry = SchemaParser().parse(schema)
    val runtimeWiring: RuntimeWiring = buildWiring()
    val schemaGenerator = SchemaGenerator()
    return schemaGenerator.makeExecutableSchema(typeRegistry, runtimeWiring)
  }

  private fun buildWiring(): RuntimeWiring {
    return RuntimeWiring.newRuntimeWiring()
      .type(
        newTypeWiring("Query")
          .dataFetcher("soek", graphQLDataFetchers.soek())
      )
      .type(
        newTypeWiring("Whisky")
          .dataFetcher("priser", graphQLDataFetchers.pricesForProduct())
      )
      .build()
  }
}
{% endhighlight %}
It is in the `buildWiring()` I have connected the query to the method that fetches the data. 

<h2>The data fetcher</h2>

The data fetcher fetches data, obvioulsy... Note how arguments/parameters to the query are accessed from the dataFetchingEnvironment. For sub-elements, the link to the parent object can be found the same place, see pricesForProduct().

{% highlight kotlin linenos %}
import no.hamre.polet.dao.Dao
import no.hamre.polet.modell.Product

import graphql.schema.DataFetcher
import graphql.schema.DataFetchingEnvironment
import org.springframework.stereotype.Component

@Component
class GraphQLDataFetchers(private val dao: Dao) {

  fun soek(): DataFetcher<*> {
    return DataFetcher { dataFetchingEnvironment: DataFetchingEnvironment ->
      val name = dataFetchingEnvironment.getArgument<String>("name")
      dao.query(name) //List<Whisky>
    }
  }

  fun pricesForProduct(): DataFetcher<*> {
    return DataFetcher { dataFetchingEnvironment: DataFetchingEnvironment ->
      //Get the parent object (source) for which we will find the prices
      val whisky: Product = dataFetchingEnvironment.getSource()
      val productId = whisky.id
      dao.findPrices(productId = productId) //List<Pris>
    }
  }
}

{% endhighlight %}

<h2>Summary</h2>
The interactive editor <code>graphiql</code> will be available at <code>/graphiql</code> by default when running the application. The GraphQl API itself is exposed on <code>/graphql</code> by default. For configuration options see <a href="https://github.com/graphql-java-kickstart/graphql-spring-boot" target="_new">graphql-spring-boot</a> documentation.


In short, you need to:
<ul>
  <li>Add dependencies</li>
  <li>Define a schema</li>
  <li>Configure Bean that represents the graphql service</li>
  <li>Wire requests to data fetchers</li>
  <li>Fetch data</li>
</ul>
