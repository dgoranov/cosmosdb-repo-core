Simple repository pattern with DocumentDB

# Getting started

## Step 1: Get DocumentDB client and create Repository

Before you can play with your DocumentDB database you need to get the DocumentDB Client by passing your endopointUrl, authorizationKey (primary) and your database name (will be created if it doesn't exist).

```csharp
internal class Program
{
    public static DocumentClient Client { get; set; }

	private static void Main(string[] args)
	{
		IDocumentDbInitializer init = new DocumentDbInitializer();

		string endpointUrl = ConfigurationManager.AppSettings["azure.documentdb.endpointUrl"];
		string authorizationKey = ConfigurationManager.AppSettings["azure.documentdb.authorizationKey"];

		// get the Azure DocumentDB client
		Client = init.GetClient(endpointUrl, authorizationKey);

		// Run demo
		Task t = MainAsync(args);
		t.Wait();
	}
}    
```

## Step 2: Use the repo with your POCO objects

With the repository in place it's really simple to do the CRUD operations:

```csharp
private static async Task MainAsync(string[] args)
{
	string databaseId = ConfigurationManager.AppSettings["azure.documentdb.databaseId"];

	// create repository for persons
	DocumentDbRepository<Person> repo = new DocumentDbRepository<Person>(Client, databaseId);

	// create a new person
	Person matt = new Person
	{
		FirstName = "m4tt",
		LastName = "TBA",
		BirthDayDateTime = new DateTime(1990, 10, 10),
		PhoneNumbers =
			new Collection<PhoneNumber>
			{
				new PhoneNumber {Number = "555", Type = "Mobile"},
				new PhoneNumber {Number = "777", Type = "Landline"}
			}
	};

	// add person to database's collection (if collection doesn't exist it will be created and named as class name -it's a convenction, that can be configured during initialization of the repository)
	matt = await repo.AddOrUpdateAsync(matt);

	// create another person
	Person jack = new Person
	{
		FirstName = "Jack",
		LastName = "Smith",
		BirthDayDateTime = new DateTime(1990, 10, 10),
		PhoneNumbers = new Collection<PhoneNumber>()
	};

	// add jack to collection
	jack = await repo.AddOrUpdateAsync(jack);

	// update first name
	matt.FirstName = "Matt";

	// add last name
	matt.LastName = "Smith";

	// remove landline phone number
	matt.PhoneNumbers.RemoveAt(1);

	// should update person
	await repo.AddOrUpdateAsync(matt);

	// get Matt by his Id
	Person justMatt = await repo.GetByIdAsync(matt.Id);

	// ... or by his first name
	Person firstMatt = await repo.FirstOrDefaultAsync(p => p.FirstName.Equals("matt", StringComparison.OrdinalIgnoreCase));

	// query all the smiths
	var smiths = (await repo.WhereAsync(p => p.LastName.Equals("Smith", StringComparison.OrdinalIgnoreCase))).ToList();

	// remove matt from collection
	await repo.RemoveAsync(matt.Id);

	// remove jack from collection
	await repo.RemoveAsync(jack.Id);
}