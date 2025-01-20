# System-design

Here is a detailed study guide with expanded descriptions, examples, and implementation details for each topic. The structure will include:
	1.	Definition: A detailed explanation of the concept.
	2.	Examples: Practical real-world use cases.
	3.	Implementation: Code examples where applicable (e.g., C#).
	4.	Commonly Used Solutions: Popular tools or frameworks.

1. Scalability

Definition:

Scalability is the system’s ability to handle an increasing number of requests or a growing dataset by adding resources. It ensures the system performs efficiently as demand grows.
	•	Vertical Scaling: Adding more resources (CPU, RAM) to a single machine.
	•	Horizontal Scaling: Adding more machines or nodes to a system.

Examples:
	•	Vertical Scaling: Upgrading a database server to a higher-tier instance.
	•	Horizontal Scaling: Adding more instances of a web server behind a load balancer.

Implementation:

```csharp
// Example: Horizontal Scaling with a Load Balancer in ASP.NET
public class LoadBalancer
{
    private List<string> _servers = new List<string>();
    private int _currentIndex = 0;

    public void AddServer(string server)
    {
        _servers.Add(server);
    }

    public string GetServer()
    {
        if (_servers.Count == 0) throw new Exception("No servers available");
        string server = _servers[_currentIndex];
        _currentIndex = (_currentIndex + 1) % _servers.Count; // Round-robin
        return server;
    }
}
```

```csharp
// Usage
var loadBalancer = new LoadBalancer();
loadBalancer.AddServer("Server1");
loadBalancer.AddServer("Server2");

Console.WriteLine(loadBalancer.GetServer()); // Output: Server1
Console.WriteLine(loadBalancer.GetServer()); // Output: Server2
```

Commonly Used Solutions:
	•	Vertical Scaling: AWS EC2 instance resizing, Azure VM scaling.
	•	Horizontal Scaling: Kubernetes, Docker Swarm.

2. Availability

Definition:

Availability is the percentage of time a system is operational and accessible. High Availability (HA) systems minimize downtime through redundancy.

Examples:
	•	Deploying services across multiple regions or availability zones.
	•	Using failover mechanisms to switch to backup systems during failure.

Implementation:
	•	SQL Server Always On Availability Groups:

```sql
-- Configuring a secondary replica for failover in SQL Server
ALTER AVAILABILITY GROUP [MyAG]
ADD REPLICA ON 'SecondaryServer' 
WITH (
    ENDPOINT_URL = 'TCP://SecondaryServer:5022',
    FAILOVER_MODE = AUTOMATIC,
    AVAILABILITY_MODE = SYNCHRONOUS_COMMIT
);
```

Commonly Used Solutions:
	•	Database: Amazon RDS Multi-AZ, SQL Server Always On.
	•	Redundancy: Load Balancers, DNS Failover.

3. CAP Theorem

Definition:

CAP Theorem states that in a distributed system, you can only guarantee two out of three:
	1.	Consistency: Every read receives the most recent write or an error.
	2.	Availability: Every request receives a response, even if it’s stale.
	3.	Partition Tolerance: The system continues to operate despite network partition.

Examples:
	•	AP (Availability + Partition Tolerance): DynamoDB.
	•	CP (Consistency + Partition Tolerance): MongoDB with Replica Sets.

Implementation:
	•	Example: Configuring MongoDB for CP:
```json
{
  "replicaSet": {
    "name": "rs0",
    "members": [
      { "_id": 0, "host": "mongo1:27017" },
      { "_id": 1, "host": "mongo2:27017" },
      { "_id": 2, "host": "mongo3:27017" }
    ]
  }
}
```

Commonly Used Solutions:
	•	AP Systems: DynamoDB, Cassandra.
	•	CP Systems: MongoDB, HBase.

4. ACID Transactions

Definition:

ACID guarantees reliable transactions in databases:
	1.	Atomicity: All operations complete or none.
	2.	Consistency: Data remains valid after the transaction.
	3.	Isolation: Concurrent transactions do not affect each other.
	4.	Durability: Completed transactions persist.

Examples:
	•	Bank transfers: Ensuring funds are deducted from one account and added to another.

Implementation:
```csharp
using (var transaction = connection.BeginTransaction())
{
    try
    {
        var command1 = new SqlCommand("UPDATE accounts SET balance = balance - 100 WHERE id = 1", connection, transaction);
        var command2 = new SqlCommand("UPDATE accounts SET balance = balance + 100 WHERE id = 2", connection, transaction);

        command1.ExecuteNonQuery();
        command2.ExecuteNonQuery();

        transaction.Commit(); // Both operations succeed
    }
    catch
    {
        transaction.Rollback(); // Revert changes on error
    }
}
```

Commonly Used Solutions:
	•	Relational Databases: PostgreSQL, MySQL, Oracle.

5. BASE Transactions

Definition:

BASE (Basically Available, Soft state, Eventually consistent) is a distributed system design that prioritizes availability over consistency.

Examples:
	•	Shopping cart updates in e-commerce systems, where changes propagate asynchronously.

Implementation:
	•	DynamoDB Example:

{
  "Table": "Orders",
  "EventualConsistency": true
}

Commonly Used Solutions:
	•	NoSQL Databases: Cassandra, CouchDB.

6. Consistency Patterns

Definition:
	•	Strong Consistency: Ensures all replicas reflect the latest write.
	•	Eventual Consistency: Updates propagate over time but may be stale momentarily.

Examples:
	•	Strong Consistency: Bank account balances.
	•	Eventual Consistency: Social media likes count.

Implementation:
	•	Eventual Consistency Example:

// Write to a replica and eventually sync with master
var replica = new RedisReplica("localhost");
replica.Write("key", "value");
// Background job to sync with master

Commonly Used Solutions:
	•	Strong: PostgreSQL, MySQL.
	•	Eventual: DynamoDB, Cassandra.

7. Rate Limiting

Definition:

Rate limiting controls the number of requests a system accepts over a specific time window.

Examples:
	•	API limiting: 100 requests per minute per user.

Implementation:
```csharp
public class RateLimiter
{
    private int _limit = 100;
    private TimeSpan _window = TimeSpan.FromMinutes(1);
    private Dictionary<string, List<DateTime>> _requestLog = new();

    public bool AllowRequest(string userId)
    {
        if (!_requestLog.ContainsKey(userId))
            _requestLog[userId] = new List<DateTime>();

        var now = DateTime.UtcNow;
        _requestLog[userId] = _requestLog[userId].Where(t => t > now - _window).ToList();

        if (_requestLog[userId].Count < _limit)
        {
            _requestLog[userId].Add(now);
            return true;
        }
        return false;
    }
}
```

Commonly Used Solutions:
	•	Nginx Rate Limiting.
	•	Cloudflare, AWS API Gateway.

8. Fault Tolerance
Definition:
Fault tolerance is the ability of a system to continue functioning in the event of component failures. It ensures minimal disruption to the user experience.

Examples:
Redundant servers in a cluster so that if one server goes down, others take over.
Automatic retries for failed API calls.
Implementation:
```csharp
// Simple retry mechanism for fault tolerance
public async Task<T> RetryAsync<T>(Func<Task<T>> action, int maxAttempts = 3)
{
    int attempts = 0;
    while (attempts < maxAttempts)
    {
        try
        {
            return await action();
        }
        catch (Exception ex) when (attempts < maxAttempts - 1)
        {
            attempts++;
            Console.WriteLine($"Retrying... Attempt {attempts}");
        }
    }
    throw new Exception("Max retry attempts exceeded.");
}
```
Commonly Used Solutions:
Kubernetes for automatic pod restarts.
Netflix Hystrix for circuit-breaking.
9. Single Point of Failure (SPOF)
Definition:
A single point of failure is a part of the system that, if it fails, causes the entire system to fail. Eliminating SPOFs improves reliability.

Examples:
A single database server without replication.
A monolithic application where all functionalities are in a single server.
Implementation:
Deploy multiple instances of services with a load balancer to distribute traffic.
Commonly Used Solutions:
Load Balancers: Nginx, AWS Elastic Load Balancer.
Multi-master databases: MongoDB, Cassandra.
10. Disaster Recovery
Definition:
Disaster recovery involves processes and mechanisms to restore functionality after catastrophic events (e.g., data center failure, natural disaster).

Examples:
Offsite backups stored in a different region.
Active-passive disaster recovery setup with a secondary data center.
Implementation:
Automate database backups:
```csharp
public void BackupDatabase(string databaseName, string backupPath)
{
    string command = $"BACKUP DATABASE {databaseName} TO DISK = '{backupPath}'";
    using (var connection = new SqlConnection("YourConnectionString"))
    {
        var sqlCommand = new SqlCommand(command, connection);
        connection.Open();
        sqlCommand.ExecuteNonQuery();
    }
}
```
Commonly Used Solutions:
AWS Backup, Azure Site Recovery.
Cross-region replication in AWS or Azure.
11. Content Delivery Network (CDN)
Definition:
A Content Delivery Network (CDN) caches content (static and dynamic) at edge locations closer to users to reduce latency and load on the origin server.

Examples:
Use CDN for delivering images, videos, or CSS files.
Accelerating website load times globally.
Implementation:
Use a CDN service like AWS CloudFront or Cloudflare.
Commonly Used Solutions:
Cloudflare, AWS CloudFront, Akamai.
12. Proxy vs Reverse Proxy
Definition:
Proxy: Acts on behalf of the client to forward requests to servers.
Reverse Proxy: Sits between clients and servers, forwarding requests to the appropriate backend server.
Examples:
Proxy: Used in corporate environments for web filtering.
Reverse Proxy: Nginx used for load balancing.
Implementation:
```json
# Reverse Proxy Example
server {
    listen 80;
    server_name example.com;
    location / {
        proxy_pass http://backend_server;
    }
}
```
Commonly Used Solutions:
Proxy: Squid.
Reverse Proxy: Nginx, HAProxy.
13. Domain Name System (DNS)
Definition:
DNS translates human-readable domain names (e.g., www.example.com) into IP addresses.

Examples:
DNS routing traffic to the closest data center.
Using DNS failover to redirect users during a data center outage.
Implementation:
Configure DNS records (e.g., A, CNAME, MX) in a provider like AWS Route 53.
Commonly Used Solutions:
AWS Route 53, Google Cloud DNS, Cloudflare.
14. Caching Strategies (LRU, LFU)
Definition:
Caching stores frequently accessed data in memory for faster retrieval.

LRU (Least Recently Used): Evicts the least recently used item.
LFU (Least Frequently Used): Evicts the least frequently accessed item.
Examples:
LRU: Browser cache evicting old pages.
LFU: Content Delivery Network evicting less popular assets.
Implementation:
```csharp

// LRU Cache Example in C#
public class LRUCache<K, V>
{
    private readonly int _capacity;
    private readonly LinkedList<(K Key, V Value)> _list = new();
    private readonly Dictionary<K, LinkedListNode<(K, V)>> _cache = new();

    public LRUCache(int capacity) => _capacity = capacity;

    public V Get(K key)
    {
        if (_cache.TryGetValue(key, out var node))
        {
            _list.Remove(node);
            _list.AddFirst(node);
            return node.Value.Value;
        }
        throw new KeyNotFoundException();
    }

    public void Put(K key, V value)
    {
        if (_cache.ContainsKey(key))
            _list.Remove(_cache[key]);
        else if (_cache.Count >= _capacity)
        {
            var lru = _list.Last.Value.Key;
            _list.RemoveLast();
            _cache.Remove(lru);
        }
        _cache[key] = _list.AddFirst((key, value));
    }
}
```
Commonly Used Solutions:
Redis, Memcached.
15. Distributed Caching
Definition:
Distributed caching spreads cached data across multiple servers to handle large-scale data access.

Examples:
Caching user session data across multiple servers using Redis Cluster.
Implementation:
Configure a Redis Cluster for caching.
Commonly Used Solutions:
Redis Cluster, Hazelcast.
16. Load Balancing (Round Robin, Weighted)
Definition:
Load balancing distributes traffic across servers to ensure reliability and performance.

Round Robin: Requests are distributed sequentially.
Weighted: Servers receive requests based on assigned weights.
Examples:
Round Robin: Basic load balancing among equal-capacity servers.
Weighted: Favor servers with higher CPU/RAM capacity.
Implementation:
```csharp

// Round Robin Load Balancer
public class RoundRobinLoadBalancer
{
    private readonly List<string> _servers = new();
    private int _currentIndex = 0;

    public void AddServer(string server) => _servers.Add(server);

    public string GetNextServer()
    {
        if (_servers.Count == 0) throw new Exception("No servers available");
        var server = _servers[_currentIndex];
        _currentIndex = (_currentIndex + 1) % _servers.Count;
        return server;
    }
}

// Usage
var loadBalancer = new RoundRobinLoadBalancer();
loadBalancer.AddServer("Server1");
loadBalancer.AddServer("Server2");
Console.WriteLine(loadBalancer.GetNextServer());
```
Commonly Used Solutions:
Nginx, AWS Elastic Load Balancer, HAProxy.
17. Database Types (SQL vs NoSQL)
Definition:
SQL: Relational databases with structured schemas.
NoSQL: Non-relational, flexible schema for unstructured data.
Examples:
SQL: Banking systems.
NoSQL: Social media platforms.
Implementation:
SQL Example:
```sql

CREATE TABLE Users (ID INT PRIMARY KEY, Name VARCHAR(50));
NoSQL Example:
json
Copy code
{
  "userID": 1,
  "name": "John Doe"
}
```
Commonly Used Solutions:
SQL: PostgreSQL, MySQL.
NoSQL: MongoDB, DynamoDB.

25. Vertical vs Horizontal Scaling
Definition:
Vertical Scaling: Adding more resources (CPU, RAM) to a single machine to handle increased demand.
Horizontal Scaling: Adding more machines to distribute the load.
Examples:
Vertical Scaling: Upgrading a database server from 16GB RAM to 64GB.
Horizontal Scaling: Adding more web servers behind a load balancer.
Tradeoffs:
Vertical Scaling:
Pros: Simpler to implement.
Cons: Limited by hardware capacity.
Horizontal Scaling:
Pros: Higher fault tolerance, better scalability.
Cons: Requires additional infrastructure like load balancers.
Implementation:
```csharp

// Horizontal Scaling with Load Balancer Example
public class HorizontalScaler
{
    private readonly List<string> _servers = new();
    private int _currentIndex = 0;

    public void AddServer(string server) => _servers.Add(server);

    public string GetServer()
    {
        if (_servers.Count == 0) throw new Exception("No servers available");
        var server = _servers[_currentIndex];
        _currentIndex = (_currentIndex + 1) % _servers.Count;
        return server;
    }
}
```
Commonly Used Solutions:
Vertical: AWS EC2 instance resizing.
Horizontal: Kubernetes, Docker Swarm.
26. Stateful vs Stateless Design
Definition:
Stateful: Systems retain client session state (e.g., login sessions).
Stateless: Each request is independent, with no stored session state.
Examples:
Stateful: Traditional FTP sessions.
Stateless: REST APIs where each request includes all necessary information.
Tradeoffs:
Stateful:
Pros: Easier to manage long-term client sessions.
Cons: Harder to scale, requires session replication.
Stateless:
Pros: Easy to scale and maintain.
Cons: Repeated data transfer may increase overhead.
Implementation:
```csharp

// Stateless Example: Using JWT Tokens
public string GenerateJwtToken(string username)
{
    var securityKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("your_secret_key"));
    var credentials = new SigningCredentials(securityKey, SecurityAlgorithms.HmacSha256);

    var token = new JwtSecurityToken(
        issuer: "example.com",
        audience: "example.com",
        claims: new[] { new Claim("username", username) },
        expires: DateTime.Now.AddMinutes(30),
        signingCredentials: credentials);

    return new JwtSecurityTokenHandler().WriteToken(token);
}
```
Commonly Used Solutions:
Stateless: REST APIs with JWT.
Stateful: Databases with session tables or Redis for session caching.
27. Batch vs Stream Processing
Definition:
Batch Processing: Processes large chunks of data at a time.
Stream Processing: Processes data in real-time as it arrives.
Examples:
Batch: Generating monthly billing reports.
Stream: Fraud detection in credit card transactions.
Tradeoffs:
Batch:
Pros: Efficient for large datasets.
Cons: Higher latency.
Stream:
Pros: Low latency.
Cons: More complex infrastructure.
Implementation:
Batch: Using tools like Hadoop.
Stream: Using Apache Kafka or Spark Streaming.
Commonly Used Solutions:
Batch: Apache Hadoop, AWS Batch.
Stream: Apache Kafka, Apache Flink.
28. Push vs Pull Architecture
Definition:
Push: Server pushes updates to clients.
Pull: Clients request updates from the server.
Examples:
Push: Notifications sent to mobile devices.
Pull: Email clients fetching new messages.
Tradeoffs:
Push:
Pros: Low latency.
Cons: Higher resource usage on the server.
Pull:
Pros: Simplified server logic.
Cons: Increased client-side polling overhead.
Implementation:
Push Example (WebSockets): See WebSocket section.
Pull Example:
```csharp

public async Task<string> FetchUpdates(string url)
{
    using var client = new HttpClient();
    return await client.GetStringAsync(url);
}
```
Commonly Used Solutions:
Push: WebSockets, SignalR.
Pull: HTTP APIs.
29. Long-Polling vs WebSockets
Definition:
Long-Polling: Client keeps a request open until the server has data to send, then reopens the request.
WebSockets: Persistent full-duplex connection between client and server.
Examples:
Long-Polling: Chat applications without WebSocket support.
WebSockets: Real-time multiplayer games.
Tradeoffs:
Long-Polling:
Pros: Easier to implement with standard HTTP.
Cons: Higher latency, more overhead.
WebSockets:
Pros: Low latency, efficient communication.
Cons: Requires more resources on the server.
Implementation:
Long-Polling Example:
```csharp

// Polling client-side example
while (true)
{
    var response = await client.GetAsync("http://example.com/poll");
    Console.WriteLine(response.Content.ReadAsStringAsync());
    await Task.Delay(1000); // Poll every second
}
```
Commonly Used Solutions:
Long-Polling: HTTP APIs.
WebSockets: SignalR, Socket.IO.
30. REST vs RPC
Definition:
REST: Resource-based communication over HTTP.
RPC: Action-based communication using lightweight protocols.
Examples:
REST: CRUD operations on users (GET /users/1).
RPC: Calling a method like getUser(userId).
Tradeoffs:
REST:
Pros: Easy to understand, widely used.
Cons: Higher verbosity for some use cases.
RPC:
Pros: Lightweight and efficient.
Cons: Tightly coupled services.
Implementation:
REST Example:
```csharp

[HttpGet("users/{id}")]
public async Task<User> GetUser(int id)
{
    return await _userService.GetUserById(id);
}
RPC Example (gRPC):
protobuf
Copy code
// gRPC Definition
service UserService {
    rpc GetUser (UserRequest) returns (User);
}
```
Commonly Used Solutions:
REST: HTTP APIs.
RPC: gRPC, Thrift.
31. Synchronous vs Asynchronous Communication
Definition:
Synchronous: The sender waits for a response before continuing.
Asynchronous: The sender does not wait for a response and can continue processing.
Examples:
Synchronous: Traditional HTTP requests.
Asynchronous: Message queues like Kafka or RabbitMQ.
Tradeoffs:
Synchronous:
Pros: Simpler, easier debugging.
Cons: Higher latency.
Asynchronous:
Pros: Scalable, fault-tolerant.
Cons: Harder to debug.
Implementation:
Asynchronous Communication with RabbitMQ (see earlier section).
Commonly Used Solutions:
Synchronous: REST APIs.
Asynchronous: Kafka, RabbitMQ.

32. Latency vs Throughput
Definition:
Latency: The time it takes for a single request to be processed.
Throughput: The number of requests a system can handle in a given time.
Examples:
Low latency: Video conferencing systems.
High throughput: Batch processing in data warehouses.
Tradeoffs:
Systems optimized for low latency may reduce throughput by dedicating resources to faster response times.
Systems optimized for high throughput may increase latency by batching or queuing requests.
Implementation:
C# Example: Measuring Latency
csharp
Copy code
var stopwatch = Stopwatch.StartNew();
await Task.Delay(200); // Simulate request processing
stopwatch.Stop();
Console.WriteLine($"Latency: {stopwatch.ElapsedMilliseconds} ms");
C# Example: Throughput Simulation
```csharp

var tasks = Enumerable.Range(1, 100).Select(async _ =>
{
    await Task.Delay(200); // Simulate processing
});
await Task.WhenAll(tasks);
Console.WriteLine("Throughput: 100 requests processed concurrently");
```
Commonly Used Solutions:
Low-latency systems: Redis, WebSockets.
High-throughput systems: Kafka, Hadoop.
33. Read-Through vs Write-Through Cache
Definition:
Read-Through Cache: Data is fetched from the cache, and if not found, it's loaded from the database and cached for future requests.
Write-Through Cache: Data is written to the cache and database simultaneously.
Examples:
Read-Through: Loading product details from a cache in an e-commerce platform.
Write-Through: Updating user session data in real-time.
Tradeoffs:
Read-Through:
Pros: Ensures data is fresh when requested.
Cons: Higher latency on cache misses.
Write-Through:
Pros: Ensures cache and database consistency.
Cons: Higher write latency.
Implementation:
Read-Through Example:
```csharp

public async Task<string> GetData(string key)
{
    if (!_cache.TryGetValue(key, out var value))
    {
        value = await _database.GetAsync(key);
        _cache.Set(key, value);
    }
    return value;
}
```
Write-Through Example:
```csharp

public async Task SetData(string key, string value)
{
    _cache.Set(key, value);
    await _database.SetAsync(key, value);
}
```
Commonly Used Solutions:
Caching: Redis, Memcached.
Database: SQL Server, MongoDB.
34. Client-Server Architecture
Definition:
A distributed architecture where clients send requests to servers, which process them and return responses.

Examples:
Web applications: Browsers acting as clients and web servers handling requests.
Tradeoffs:
Pros: Centralized control, simplified client-side logic.
Cons: Dependency on the server; limited offline capabilities.
Implementation:
C# Example: Basic HTTP Client
```csharp

using var client = new HttpClient();
var response = await client.GetAsync("https://api.example.com");
Console.WriteLine(await response.Content.ReadAsStringAsync());
```
Commonly Used Solutions:
Web servers: Apache, Nginx.
Protocols: HTTP, WebSockets.
35. Microservices Architecture
Definition:
An architectural style where an application is built as a collection of small, independent services that communicate via APIs.

Examples:
An e-commerce platform with separate services for user management, payments, and inventory.
Tradeoffs:
Pros: Scalability, fault isolation, technology flexibility.
Cons: Increased complexity, higher latency due to inter-service communication.
Implementation:
Service Communication Example (REST):
```csharp

[HttpGet("products/{id}")]
public async Task<Product> GetProduct(int id)
{
    return await _productService.GetByIdAsync(id);
}
```
Commonly Used Solutions:
Orchestration: Kubernetes, Docker Swarm.
Service Communication: REST, gRPC.
36. Serverless Architecture
Definition:
A cloud-native model where developers focus on writing code, and the cloud provider handles the infrastructure, scaling, and management.

Examples:
AWS Lambda for executing code in response to events.
Azure Functions for building APIs or event-driven applications.
Tradeoffs:
Pros: Reduced operational overhead, auto-scaling.
Cons: Cold start latency, vendor lock-in.
Implementation:
AWS Lambda (Example in C#):
```csharp

public class Function
{
    public string Handler(string input, ILambdaContext context)
    {
        return $"Hello, {input}";
    }
}
```
Commonly Used Solutions:
AWS Lambda, Azure Functions, Google Cloud Functions.
37. Event-Driven Architecture
Definition:
An architecture where services communicate by producing and consuming events, enabling asynchronous communication.

Examples:
An e-commerce platform where an order service emits an event when an order is placed, and the inventory service updates stock accordingly.
Tradeoffs:
Pros: Decoupled systems, scalability.
Cons: Increased complexity in debugging and monitoring.
Implementation:
Event Publishing Example:
```csharp

var factory = new ConnectionFactory() { HostName = "localhost" };
using var connection = factory.CreateConnection();
using var channel = connection.CreateModel();

channel.ExchangeDeclare(exchange: "order_events", type: ExchangeType.Fanout);
var message = "Order Placed";
var body = Encoding.UTF8.GetBytes(message);

channel.BasicPublish(exchange: "order_events", routingKey: "", body: body);
Console.WriteLine("Event Published: {0}", message);
```
Commonly Used Solutions:
Kafka, RabbitMQ.
38. Peer-to-Peer (P2P) Architecture
Definition:
A decentralized architecture where nodes (peers) act as both clients and servers, sharing resources directly.

Examples:
File-sharing networks like BitTorrent.
Blockchain technology.
Tradeoffs:
Pros: Resilient to failures, cost-effective.
Cons: Harder to manage, inconsistent performance.
Implementation:
Custom P2P protocols typically involve socket programming.
Commonly Used Solutions:
Libraries: libp2p, WebRTC.
39. API Gateway Design
Definition:
An API Gateway acts as a single entry point for API requests, routing them to appropriate backend services.

Examples:
Aggregating responses from multiple microservices into a single client response.
Tradeoffs:
Pros: Centralized API management, rate limiting, and security.
Cons: Potential single point of failure, added latency.
Implementation:
Example with AWS API Gateway or custom routing logic in C#.
Commonly Used Solutions:
Kong, AWS API Gateway, Nginx.
40. Service Discovery Mechanisms
Definition:
Service discovery allows services to locate and communicate with each other in a dynamic environment, such as a microservices architecture.

Examples:
A payment service querying the service registry to find the inventory service's address.
Tradeoffs:
Pros: Simplifies service-to-service communication.
Cons: Adds complexity to system design.
Implementation:
Using tools like Consul or Eureka for service registration and discovery.
Commonly Used Solutions:
Consul, Eureka, Zookeeper.






