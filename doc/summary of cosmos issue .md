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
在上面的代码示例中，CreateDatabaseIfNotExistsAsync会检测并创建Id为PennyDB的DataBase,然后使用CreateDocumentCollectionIfNotExistsAsync来创建collection,
