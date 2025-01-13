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

// Usage
var loadBalancer = new LoadBalancer();
loadBalancer.AddServer("Server1");
loadBalancer.AddServer("Server2");

Console.WriteLine(loadBalancer.GetServer()); // Output: Server1
Console.WriteLine(loadBalancer.GetServer()); // Output: Server2

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

-- Configuring a secondary replica for failover in SQL Server
ALTER AVAILABILITY GROUP [MyAG]
ADD REPLICA ON 'SecondaryServer' 
WITH (
    ENDPOINT_URL = 'TCP://SecondaryServer:5022',
    FAILOVER_MODE = AUTOMATIC,
    AVAILABILITY_MODE = SYNCHRONOUS_COMMIT
);

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

Commonly Used Solutions:
	•	Nginx Rate Limiting.
	•	Cloudflare, AWS API Gateway.

This expanded guide includes more detailed explanations and implementations. Let me know which other topics you’d like to focus on next!
