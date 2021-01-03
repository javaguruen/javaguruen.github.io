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

Add the schema as a file named <code>schema.graphqls</code> in the <code>src/main/resources</code> folder of your project. The examples given here are a subset of code from an actual project of mine, so please excuse the mix of Norwegian and English.

{% highlight graphql linenos %}
type Query {
    "Søk etter whisky. Kan søke på navn og produsent"
    soek(name: String): [Whisky]
}

type Whisky {
    id: ID!,
    "Identificator for a product at vinmonopolet.no"
    varenummer: String!,
    varenavn: String!,
    land: String!,
    distrikt: String,
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

<h2>The data fetcher</h2>

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
      dao.query(name) //List<Product>
    }
  }

  fun pricesForProduct(): DataFetcher<*> {
    return DataFetcher { dataFetchingEnvironment: DataFetchingEnvironment ->
      //Get the parent object (source) for which we will find the prices
      val whisky: Product = dataFetchingEnvironment.getSource()
      val productId = whisky.id
      dao.findPrices(productId = productId) //List<Price>
    }
  }
}

{% endhighlight %}

<h2>The object model</h2>
It can seem odd that the classes are named Product and Price in the code but Whisky and Pris in the GraphQl schema. The name of the containing class is not important, only the name of the fields within.

{% highlight kotlin linenos %}
// Data for a whisky
data class Product(
    val id: Long,
    val varenummer: String,
    val varenavn: String,
    val land: String,
    val distrikt: String? = null,
    val produsent: String,
    val prices: List<Price> = listOf()
)

data class Price(
    val id: Long,
    val datotid: LocalDateTime,
    val varenummer: String,
    val volum: Double,
    val pris: Double,
    val literpris: Double
)

{% endhighlight %}

<h2>Summary</h2>
The interactive editor <code>graphiql</code> will be available at <code>/graphiql</code> when running the application. The GraphQl API itself is exposed on <code>/graphql</code> by default. For configuration options see <a href="https://github.com/graphql-java-kickstart/graphql-spring-boot" target="_new">graphql-spring-boot</a> documentation.