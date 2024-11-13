# Spring-boot-Encache

To implement a caching mechanism in a Spring Boot application for boosting performance and reducing database load, you can use Spring's built-in caching abstraction. This abstraction provides a unified API for caching, making it easy to switch between different caching providers (like EhCache, Caffeine, Redis, etc.) without changing the business logic.

Below is a step-by-step guide on setting up and using Spring's caching mechanism:

1. Add Dependencies

To use caching in a Spring Boot application, you need to add the necessary dependencies to your pom.xml (for Maven) or build.gradle (for Gradle) file. For example, if you plan to use Spring Cache with EhCache, the dependency would look like this:

Maven (pom.xml):
```
<dependencies>
    <!-- Spring Boot Starter for Caching -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>

    <!-- EhCache (optional, you can choose your cache provider) -->
    <dependency>
        <groupId>net.sf.ehcache</groupId>
        <artifactId>ehcache</artifactId>
    </dependency>
</dependencies>
```
Gradle (build.gradle):
```
dependencies {
    // Spring Boot Starter for Caching
    implementation 'org.springframework.boot:spring-boot-starter-cache'

    // EhCache (optional, you can choose your cache provider)
    implementation 'net.sf.ehcache:ehcache'
}
```
2. Enable Caching in Your Application

To enable caching, add the @EnableCaching annotation in your main application class or any configuration class:
```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;

@SpringBootApplication
@EnableCaching  // Enable Spring's caching support
public class CachingApplication {
    public static void main(String[] args) {
        SpringApplication.run(CachingApplication.class, args);
    }
}
```
3. Annotate Methods to Use Caching

Use the @Cacheable, @CachePut, and @CacheEvict annotations to manage caching behavior:

@Cacheable: Used to cache the result of a method.

@CachePut: Updates the cache without skipping the method.

@CacheEvict: Removes an entry from the cache.

```
Here's an example using a Service class to demonstrate:

import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
public class ProductService {

    // Simulated database call for fetching product by ID
    @Cacheable(value = "products", key = "#productId")
    public Product getProductById(Long productId) {
        // Simulating a slow database call
        simulateSlowService();
        return findProductInDatabase(productId);
    }

    private Product findProductInDatabase(Long productId) {
        // Placeholder for database fetch logic
        System.out.println("Fetching product from database with ID: " + productId);
        return new Product(productId, "Product-" + productId, 100.0);
    }

    // Simulates a slow service call by delaying execution
    private void simulateSlowService() {
        try {
            Thread.sleep(2000L); // Simulating 2 seconds delay
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
4. Configure Cache Properties

You can customize cache properties (like expiration, eviction policies, size limits, etc.) depending on the cache provider. For EhCache, you would typically use an XML configuration or properties file.

EhCache Configuration Example (ehcache.xml):
```
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd">
    <cache name="products"
           maxEntriesLocalHeap="1000"
           timeToLiveSeconds="3600">
    </cache>
</ehcache>
```
Spring Boot Configuration (application.properties):

Specify the cache configuration file in the application properties:

# Cache configuration using EhCache
```
spring.cache.type=ehcache
spring.cache.ehcache.config=classpath:ehcache.xml
```
5. Handling Cache Updates and Evictions

To update the cache when data changes (e.g., when a product is updated), use @CachePut. To remove an item from the cache, use @CacheEvict.

Example:
```
import org.springframework.cache.annotation.CachePut;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.stereotype.Service;

@Service
public class ProductService {

    // Updating cache when a product is updated
    @CachePut(value = "products", key = "#product.id")
    public Product updateProduct(Product product) {
        // Update product in the database
        return updateProductInDatabase(product);
    }

    // Evicting a cache entry when a product is deleted
    @CacheEvict(value = "products", key = "#productId")
    public void deleteProduct(Long productId) {
        // Delete product from the database
        deleteProductFromDatabase(productId);
    }

    // Placeholder for database update logic
    private Product updateProductInDatabase(Product product) {
        System.out.println("Updating product in the database: " + product.getId());
        return product;
    }

    // Placeholder for database delete logic
    private void deleteProductFromDatabase(Long productId) {
        System.out.println("Deleting product from the database with ID: " + productId);
    }
}
```
6. Monitor Caching Behavior

To ensure that caching is working correctly:

Use logs to see if data is being fetched from the cache or the database.

Enable spring.cache.cache-names to list cache regions during startup.


Example:

# Enable detailed logging for caching
logging.level.org.springframework.cache=DEBUG

7. Additional Caching Tips
```
Choose the Right Cache Provider: If you need a lightweight cache, use Caffeine; for distributed caching, consider Redis.

Cache Expiration: Set appropriate expiration times for cache entries based on data volatility.

Cache Size: Monitor cache size to prevent excessive memory usage.

Concurrency: Use @Cacheable(sync = true) to avoid cache stampede if multiple threads try to access the same data.

```
Example with Redis Cache (Alternative)

To use Redis instead of EhCache:

1. Add the dependency for Redis:
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

2. Configure Redis in application.properties:
```
spring.cache.type=redis
spring.redis.host=localhost
spring.redis.port=6379

```

By using these techniques, Spring Boot's caching abstraction makes it straightforward to add and manage caching, significantly improving application performance by reducing database load for frequently accessed data.

