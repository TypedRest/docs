# Mocking endpoints

TypedRest endpoint classes implement interfaces, making them straightforward to mock with standard testing frameworks. This allows you to test service or business logic that depends on TypedRest endpoints without making any HTTP calls.

## Example service

Consider a service that filters contacts via a TypedRest endpoint:

=== "C#"

    ```csharp
    class ContactService(ICollectionEndpoint<Contact> contacts)
    {
        public async Task<List<Contact>> GetActiveContactsAsync(CancellationToken ct = default)
            => (await contacts.ReadAllAsync(ct))
                .Where(c => c.IsActive)
                .ToList();
    }
    ```

=== "Java"

    ```java
    class ContactService {
        private final CollectionEndpoint<Contact> contacts;

        ContactService(CollectionEndpoint<Contact> contacts) {
            this.contacts = contacts;
        }

        List<Contact> getActiveContacts() {
            return contacts.readAll().stream()
                .filter(Contact::isActive)
                .collect(Collectors.toList());
        }
    }
    ```

=== "Kotlin"

    ```kotlin
    class ContactService(private val contacts: CollectionEndpoint<Contact>) {
        fun getActiveContacts(): List<Contact> =
            contacts.readAll().filter { it.isActive }
    }
    ```

=== "TypeScript"

    ```typescript
    class ContactService {
        constructor(private contacts: CollectionEndpoint<Contact>) {}

        async getActiveContacts(): Promise<Contact[]> {
            return (await this.contacts.readAll()).filter(c => c.isActive);
        }
    }
    ```

## Unit tests with mocks

=== "C#"

    Use [Moq](https://github.com/moq/moq) (or any other mocking library) to substitute the endpoint interface:

    ```csharp
    var mockContacts = new Mock<ICollectionEndpoint<Contact>>();
    mockContacts
        .Setup(c => c.ReadAllAsync(It.IsAny<CancellationToken>()))
        .ReturnsAsync([
            new Contact { Name = "Alice", IsActive = true },
            new Contact { Name = "Bob",   IsActive = false }
        ]);

    var service = new ContactService(mockContacts.Object);
    var result = await service.GetActiveContactsAsync();

    result.Should().ContainSingle(c => c.Name == "Alice");
    ```

    Key interfaces available for mocking:

    | Interface | Description |
    |---|---|
    | `IElementEndpoint<T>` | Single resource (`ReadAsync`, `SetAsync`, `DeleteAsync`, …) |
    | `ICollectionEndpoint<T>` | Resource collection (`ReadAllAsync`, `CreateAsync`, …) |
    | `ICollectionEndpoint<T, TElementEndpoint>` | Resource collection with a custom element endpoint type |
    | `IActionEndpoint` | Parameterless RPC trigger |
    | `IConsumerEndpoint<T>` | RPC call that accepts input |
    | `IFunctionEndpoint<TIn, TOut>` | RPC call with input and output |
    | `IProducerEndpoint<T>` | RPC call that returns a result |

=== "Java"

    Use [Mockito](https://site.mockito.org/) (or any other mocking library) to substitute the endpoint interface:

    ```java
    CollectionEndpoint<Contact> mockContacts = mock(CollectionEndpoint.class);
    when(mockContacts.readAll()).thenReturn(List.of(
        new Contact("Alice", true),
        new Contact("Bob",   false)
    ));

    ContactService service = new ContactService(mockContacts);
    List<Contact> result = service.getActiveContacts();

    assertEquals(1, result.size());
    assertEquals("Alice", result.get(0).getName());
    ```

    Key interfaces available for mocking:

    | Interface | Description |
    |---|---|
    | `ElementEndpoint<T>` | Single resource (`read`, `set`, `delete`, …) |
    | `CollectionEndpoint<T>` | Resource collection (`readAll`, `create`, …) |
    | `GenericCollectionEndpoint<T, TElementEndpoint>` | Resource collection with a custom element endpoint type |
    | `ActionEndpoint` | Parameterless RPC trigger |
    | `ConsumerEndpoint<T>` | RPC call that accepts input |
    | `FunctionEndpoint<TIn, TOut>` | RPC call with input and output |
    | `ProducerEndpoint<T>` | RPC call that returns a result |

=== "Kotlin"

    Use [mockito-kotlin](https://github.com/mockito/mockito-kotlin) (or any other mocking library) to substitute the endpoint interface:

    ```kotlin
    val mockContacts = mock<CollectionEndpoint<Contact>>()
    whenever(mockContacts.readAll()).thenReturn(listOf(
        Contact("Alice", isActive = true),
        Contact("Bob",   isActive = false)
    ))

    val service = ContactService(mockContacts)
    val result = service.getActiveContacts()

    assertEquals(listOf(Contact("Alice", isActive = true)), result)
    ```

    Key interfaces available for mocking:

    | Interface | Description |
    |---|---|
    | `ElementEndpoint<T>` | Single resource (`read`, `set`, `delete`, …) |
    | `CollectionEndpoint<T>` | Resource collection (`readAll`, `create`, …) |
    | `GenericCollectionEndpoint<T, TElementEndpoint>` | Resource collection with a custom element endpoint type |
    | `ActionEndpoint` | Parameterless RPC trigger |
    | `ConsumerEndpoint<T>` | RPC call that accepts input |
    | `FunctionEndpoint<TIn, TOut>` | RPC call with input and output |
    | `ProducerEndpoint<T>` | RPC call that returns a result |

=== "TypeScript"

    Use Jest's built-in mock utilities (or any other mocking library) to create a partial stand-in for the endpoint:

    ```typescript
    const mockContacts = {
        readAll: jest.fn().mockResolvedValue([
            { name: 'Alice', isActive: true },
            { name: 'Bob',   isActive: false }
        ])
    } as unknown as CollectionEndpoint<Contact>;

    const service = new ContactService(mockContacts);
    const result = await service.getActiveContacts();

    expect(result).toEqual([{ name: 'Alice', isActive: true }]);
    ```

    Because TypeScript uses structural typing, any object that provides the methods your service calls is a valid substitute - no separate interface declaration is required.

## Mocking a custom client

Real clients are trees of endpoints: an entry endpoint with child endpoints, some of them [custom endpoint types](../introduction.md#patterns) with children of their own. To mock code that navigates such a tree, for example by calling `Contacts["1337"].Poke`, each level of the chain needs to be mockable. The examples below test a `WelcomeService` that pokes the contact with a given ID.

=== "C#"

    Mocking libraries such as Moq can only intercept interface members and virtual methods. TypedRest's own endpoints all have interfaces; give your custom endpoints interfaces too, and use the interfaces in your client's properties:

    ```csharp
    public interface IContactEndpoint : IElementEndpoint<Contact>
    {
        IElementEndpoint<Note> Note { get; }
        IActionEndpoint Poke { get; }
    }

    public class ContactEndpoint(IEndpoint referrer, Uri relativeUri)
        : ElementEndpoint<Contact>(referrer, relativeUri), IContactEndpoint
    {
        public IElementEndpoint<Note> Note => new ElementEndpoint<Note>(this, relativeUri: "./note");
        public IActionEndpoint Poke => new ActionEndpoint(this, relativeUri: "./poke");
    }

    public interface IMyClient : IEndpoint
    {
        ICollectionEndpoint<Contact, IContactEndpoint> Contacts { get; }
    }

    public class MyClient(Uri uri) : EntryEndpoint(uri), IMyClient
    {
        public ICollectionEndpoint<Contact, IContactEndpoint> Contacts
            => new CollectionEndpoint<Contact, ContactEndpoint>(this, relativeUri: "./contacts");
    }
    ```

    `ICollectionEndpoint` is covariant in the element endpoint type, so a collection of `ContactEndpoint`s can be exposed as a collection of `IContactEndpoint`s. The [code generator](../code-generation/index.md) can create interfaces like these for you.

    Moq can then set up and verify the whole chain in one expression:

    ```csharp
    var client = new Mock<IMyClient>();
    client.Setup(x => x.Contacts["1337"].Poke.InvokeAsync(It.IsAny<CancellationToken>()))
          .Returns(Task.CompletedTask);

    await new WelcomeService(client.Object).WelcomeAsync("1337");

    client.Verify(x => x.Contacts["1337"].Poke.InvokeAsync(It.IsAny<CancellationToken>()));
    ```

=== "Java"

    Mockito can mock classes as well as interfaces, so custom endpoints don't need interfaces of their own. Stub each level of the chain, here for a client whose `getContacts()` returns a `GenericCollectionEndpoint<Contact, ContactEndpoint>`:

    ```java
    ActionEndpoint poke = mock(ActionEndpoint.class);
    ContactEndpoint contact = mock(ContactEndpoint.class);
    when(contact.getPoke()).thenReturn(poke);
    GenericCollectionEndpoint<Contact, ContactEndpoint> contacts = mock(GenericCollectionEndpoint.class);
    when(contacts.get("1337")).thenReturn(contact);
    MyClient client = mock(MyClient.class);
    when(client.getContacts()).thenReturn(contacts);

    new WelcomeService(client).welcome("1337");

    verify(poke).invoke();
    ```

=== "Kotlin"

    mockito-kotlin can mock classes as well as interfaces, so custom endpoints don't need interfaces of their own. Kotlin classes are final by default, which Mockito 5 handles out of the box. Stub each level of the chain, here for a client whose `contacts` is a `GenericCollectionEndpoint<Contact, ContactEndpoint>`:

    ```kotlin
    val poke = mock<ActionEndpoint>()
    val contact = mock<ContactEndpoint> { on { this.poke } doReturn poke }
    val contacts = mock<GenericCollectionEndpoint<Contact, ContactEndpoint>> { on { get("1337") } doReturn contact }
    val client = mock<MyClient> { on { this.contacts } doReturn contacts }

    WelcomeService(client).welcome("1337")

    verify(poke).invoke()
    ```

=== "TypeScript"

    Build a plain object for each level of the chain, with only the members your code uses:

    ```typescript
    const poke = { invoke: jest.fn().mockResolvedValue(undefined) };
    const contacts = { get: jest.fn().mockReturnValue({ poke }) };
    const client = { contacts } as unknown as MyClient;

    await new WelcomeService(client).welcome('1337');

    expect(contacts.get).toHaveBeenCalledWith('1337');
    expect(poke.invoke).toHaveBeenCalled();
    ```
