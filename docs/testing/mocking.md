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
