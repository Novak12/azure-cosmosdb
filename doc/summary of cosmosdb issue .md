### cosmosdb问题汇总

#### 1.如何创建cosmosdb？
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
            var res = p.client.CreateDocumentQuery<DocDto>(p.collection.SelfLink, sql).ToList();
```

#### 3. Query with PartitionKey


#### 4. performance and suggestions
