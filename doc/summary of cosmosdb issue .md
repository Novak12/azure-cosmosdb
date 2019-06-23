### cosmosdb问题汇总

#### 1.How to create cosmosdb with .net core？
```javascript
public class Program
    {
        private const string EndpointUrl = "https://pennytemplatetest.documents.azure.cn:443/";
        private const string PrimaryKey = "*****";
        private DocumentClient client;
        
        private async Task GetStartedDemo()
        {
            client = new DocumentClient(new Uri(EndpointUrl), PrimaryKey);
            await client.CreateDatabaseIfNotExistsAsync(new Database { Id = "PennyDB" });
            await client.CreateDocumentCollectionIfNotExistsAsync(UriFactory.CreateDatabaseUri("PennyDB"), new DocumentCollection { Id = "MyCollection4", PartitionKey = new PartitionKeyDefinition { Paths = new Collection<string> { "/country" } } }, new RequestOptions { OfferThroughput = 1000 });
            //await client.CreateDocumentCollectionIfNotExistsAsync(UriFactory.CreateDatabaseUri("PennyDB"), new DocumentCollection { Id = "PennyCollection4" },  new RequestOptions { PartitionKey = new PartitionKeyDefinition { Paths = new Collection<string> { "/country" } }, OfferThroughput = 1000 });
        }      
    }
```
引入依赖包Microsoft.Azure.DocumentDB.Core.<br/>
在上面的代码示例中，CreateDatabaseIfNotExistsAsync会检测并创建Id为PennyDB的DataBase,然后使用CreateDocumentCollectionIfNotExistsAsync来创建collection,在创建collection，可配置PartitionKey和RU的相关设置。<br/>

#### 2. Query
cosmosdb提供了都中API查询方式，如sql API,Mongo API,Table API等，其中sql API与关系型数据库的查询极为相似，指出直接使用SQl或Linq来查询cosmosdb.本文提供的查询方式也是以sql API为主：
```javascript
 //********************************************************************************************************
            // 1 -  Query for a Database
            //
            // Note: we are using query here instead of ReadDatabaseAsync because we're checking if something exists
            //       the ReadDatabaseAsync method expects the resource to be there, if its not we will get an error
            //       instead of an empty 
            //********************************************************************************************************
            Database database = client.CreateDatabaseQuery().Where(db => db.Id == databaseId).AsEnumerable().FirstOrDefault();
            Console.WriteLine("1. Query for a database returned: {0}", database==null?"no results":database.Id);
 //***************************************
            // 2 - List all databases for an account
            //***************************************
            var databases = await client.ReadDatabaseFeedAsync();
            Console.WriteLine("\n4. Reading all databases resources for an account");
            foreach (var db in databases)
            {
                Console.WriteLine(db);    
            }
   //***************************************
            // 3 - Query database by sql
            //***************************************
            var sql = "SELECT * FROM c where c.StudentNo>=250000 and c.StudentNo <= 250020";
            var res = client.CreateDocumentQuery<DocDto>(p.collection.SelfLink, sql).ToList();
```

#### 3. Query with PartitionKey
在cosmosdb中，数据库会根据partition key进行分区。如果在查询数据时，已经知道数据所在的分区，最佳的查询方式就是带上partition key进行查询：
```javascript
var option = new FeedOptions { PartitionKey = new PartitionKey("/school")};
var stu = p.client.CreateDocumentQuery<DocDto>(
                    UriFactory.CreateDocumentCollectionUri("PennyDB", "MyCollection4"), option)
                    .Where(x => x.StudentNo == 250000).ToList();
```

#### 4. performance and suggestions
##### 策略1：使用网络直连的方式<br/>
客户端如何连接到Azure Cosmos DB对性能有重要的影响，尤其是在客户端观察到的延迟方面。配置客户端连接策略有两个关键的配置设置—连接模式和连接协议.<br/>
1) 网关模式 --- 默认模式，支持所有的SDK平台。适用于具有严格防火墙限制的网络中，因为它是使用标准的https协议.<br/>
2) 直连模式 --- 支持TCP和HTTPS协议,.net standard 2.0支持该模式.<br/>

最佳的方式 --- 直连模式
```javascript
client = new DocumentClient(new Uri(EndpointUrl), PrimaryKey, new ConnectionPolicy
            {
                ConnectionMode = ConnectionMode.Direct,
                ConnectionProtocol = Protocol.Tcp,
                RequestTimeout = new TimeSpan(1, 0, 0),
            });
```
在database创建的时候指定网络连接方式。<br/>
##### 策略2：Use 64-bit host processing
对于跨partition查询，使用64位宿主处理会提升性能。
