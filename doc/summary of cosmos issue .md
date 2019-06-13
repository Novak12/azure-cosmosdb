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
        private void WriteToConsoleAndPromptToContinue(string format, params object[] args)
        {
            Console.WriteLine(format, args);
            Console.WriteLine("Press any key to continue...");
           Console.ReadKey();
        }
        private static void Main(string[] args)
        {
            try
            {
                Program p = new Program();
                p.GetStartedDemo().Wait();
            }
            catch (DocumentClientException de)
            {
                System.Exception baseException = de.GetBaseException();
                Console.WriteLine($"{de.StatusCode} error occurred: {de.Message}, Message: {baseException.Message}");
            }
            catch (Exception e)
            {
                Exception baseException = e.GetBaseException();
                Console.WriteLine($"Error: {e.Message}, Message: {baseException.Message}");
            }
            finally
            {
                Console.WriteLine("End of demo, press any key to exit.");
                Console.ReadKey();
            }
        }
    }
```
