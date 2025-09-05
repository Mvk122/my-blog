---
blog_title: Getting Started with Spark and Iceberg Locally (Without Docker)
blog_path: ./blogs/getting-started-with-spark-and-iceberg-locally.html
blog_description: There seems to be a lack of documentation online on how to do this, so I'm making it.
---

{% include "header.jinja" %}

# Getting Started with Spark and Iceberg Locally

[The official quickstart guide](https://iceberg.apache.org/spark-quickstart/) for using Apache Spark with Apache Iceberg recommends using [a docker image](https://hub.docker.com/r/tabulario/spark-iceberg) to get started. While that's great for getting a playground environment to play with SparkSQL, it isn't representative of how you'd actually start a real project.

So here's how you'd actually create a Maven Java 17 project that's actually modular.

Firstly we want to get the relevant dependencies, Spark and Iceberg so lets just add this to our `pom.xml`:

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-core_2.13</artifactId>
        <version>3.5.5</version>
    </dependency>

    <dependency>
        <groupId>org.apache.iceberg</groupId>
        <artifactId>iceberg-core</artifactId>
        <version>1.9.2</version>
        <scope>runtime</scope>
    </dependency>
</dependencies>

<repositories>
    <repository>
        <id>central</id>
        <name>Central Repository</name>
        <url>https://repo1.maven.org/maven2</url>
    </repository>
</repositories>
```

Some Java code TODO yap

```java
package sparkplayground;

import org.apache.spark.sql.SparkSession;

public class Main {
    public static SparkSession createSparkSession() {
        return SparkSession.builder()
                .config("spark.sql.extensions", "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions")
                .config("spark.sql.catalog.hadoop_catalog", "org.apache.iceberg.spark.SparkCatalog")
                .config("spark.sql.catalog.hadoop_catalog.type", "hadoop")
                .config("spark.sql.catalog.hadoop_catalog.warehouse", "file:///database")
                .config("spark.master", "local")
                .getOrCreate();
    }

    public static void main(String[] args) {
        SparkSession spark = createSparkSession();
        spark.sql("CREATE TABLE local.db.test (id BIGINT, name STRING) USING ICEBERG");
        System.out.println("Hello World");
    }
}
```

Running this on Java 17 will give us an exception:
```
Exception in thread "main" java.lang.IllegalAccessError: class org.apache.spark.storage.StorageUtils$ (in unnamed module @0x1a75e76a) cannot access class sun.nio.ch.DirectBuffer (in module java.base) because module java.base does not export sun.nio.ch to unnamed module @0x1a75e76a
```

We need to export `sun.nio.ch` in our Java config by adding VM options:

```
--add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.lang.invoke=ALL-UNNAMED --add-opens=java.base/java.lang.reflect=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.base/java.net=ALL-UNNAMED --add-opens=java.base/java.nio=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/java.util.concurrent=ALL-UNNAMED --add-opens=java.base/java.util.concurrent.atomic=ALL-UNNAMED --add-opens=java.base/sun.nio.ch=ALL-UNNAMED --add-opens=java.base/sun.nio.cs=ALL-UNNAMED --add-opens=java.base/sun.security.action=ALL-UNNAMED --add-opens=java.base/sun.util.calendar=ALL-UNNAMED --add-opens=java.security.jgss/sun.security.krb5=ALL-UNNAMED
```

{% include "footer.jinja" %}