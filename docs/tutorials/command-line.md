# Command-line tool

!!! note
    This tutorial is .NET only. [TypedRest.CommandLine](https://www.nuget.org/packages/TypedRest.CommandLine/) has no Java or TypeScript equivalent.

A TypedRest client already describes the structure of an API: which resources exist, how they nest, and what can be done with each. TypedRest.CommandLine uses that to build a command-line interface with very little extra code. Each argument navigates one level deeper into the client, and the last one picks the operation:

```
addressbook contacts                           →  GET    /contacts
addressbook contacts create '{"name":"Smith"}' →  POST   /contacts
addressbook contacts 1337                      →  GET    /contacts/1337
addressbook contacts 1337 note                 →  GET    /contacts/1337/note
addressbook contacts 1337 poke                 →  POST   /contacts/1337/poke
addressbook contacts 1337 delete               →  DELETE /contacts/1337
```

This tutorial builds that tool for the `AddressBookClient` from [Build your first client](first-client.md).

## 1. Create the project

```bash
dotnet new console -n AddressBook.Cli
cd AddressBook.Cli
dotnet add package TypedRest.CommandLine
```

Then add a reference to the project or package that contains your client, or copy the client classes into the new project.

## 2. Mirror the client with commands

For each endpoint type there is a command type in the `TypedRest.CommandLine.Commands` namespace that maps arguments onto that endpoint's methods: `CollectionCommand` for collection endpoints, `ElementCommand` for element endpoints, `ActionCommand` for action endpoints, and so on.

Start with the root command. It derives from `EntryCommand` and names the top-level commands:

```csharp
using TypedRest.CommandLine.Commands;
using TypedRest.CommandLine.Commands.Generic;

class AddressBookCommand : EntryCommand<AddressBookClient>
{
    public AddressBookCommand(AddressBookClient client)
        : base(client)
    {
        Add("contacts", x => new CollectionCommand<Contact, ContactEndpoint, ContactCommand>(x.Contacts));
    }
}
```

`CollectionCommand` takes three type arguments here: the entity, the element endpoint type the collection hands out, and the command to use for those elements. A plain `ElementCommand<Contact>` would only offer reading, setting, merging and deleting a contact. To reach the note and the poke action, write a command for `ContactEndpoint` that adds them as sub-commands:

```csharp
using TypedRest.CommandLine.Commands.Rpc;

class ContactCommand : ElementCommand<Contact>
{
    private readonly ContactEndpoint _contact;

    public ContactCommand(ContactEndpoint endpoint)
        : base(endpoint)
    {
        _contact = endpoint;
    }

    protected override IEndpointCommand? GetSubCommand(string name)
        => name.ToLowerInvariant() switch
        {
            "note" => new ElementCommand<Note>(_contact.Note),
            "poke" => new ActionCommand(_contact.Poke),
            _ => base.GetSubCommand(name)
        };
}
```

The collection command creates `ContactCommand`s through their constructor, so it must be public and take exactly one `ContactEndpoint`.

## 3. Run the commands

An `Executor` connects everything. It builds the client, creates the root command, passes it the arguments and turns exceptions into error messages and exit codes. Replace the contents of `Program.cs`:

```csharp
using TypedRest.CommandLine;

return await new Executor<AddressBookClient, AddressBookCommand>().RunAsync(args);
```

By default, the `Executor` creates the client by calling its constructor that takes a `Uri`. The first time you run the tool, it asks for the API's URI and an OAuth token, and stores both in a folder named after your executable in the user's application data directory. Later runs reuse them. If the API answers with `401 Unauthorized`, the stored token is deleted and the tool asks for a new one.

If the API has a link with the relation type `token-provider`, the tool opens that page in the browser when it asks for a token, so users can log in and copy a token from there.

!!! tip "Providing the URI without prompting"
    A file named `uri` next to the executable takes precedence over the stored URI. Ship one with the tool to point it at a fixed server.

## 4. Use your own way of building the client

The default behavior suits APIs that use bearer tokens. For anything else, implement `IEndpointProvider` and pass it to the `Executor`. This one reads the URI from an environment variable and uses no authentication:

```csharp
class AddressBookProvider : IEndpointProvider<AddressBookClient>
{
    public AddressBookClient Build()
        => new(new Uri(Environment.GetEnvironmentVariable("ADDRESSBOOK_URI") ?? "http://localhost:5000/"));

    public void ResetAuthentication() {}
}
```

```csharp
return await new Executor<AddressBookClient, AddressBookCommand>(new AddressBookProvider()).RunAsync(args);
```

To keep the config files and prompts but change some details, derive from `EndpointProviderBase` or `CliEndpointProvider` instead. For example, override `ConfigDir` to store the settings somewhere else.

## 5. Try it

```bash
# GET /contacts
addressbook contacts

# GET /contacts with Range: elements=0-9
addressbook contacts 0-9

# POST /contacts, then GET the new contact and print it
addressbook contacts create '{"name":"Smith","email":"smith@example.com"}'

# The same, reading the JSON from stdin
echo '{"name":"Smith"}' | addressbook contacts create

# GET /contacts/1337
addressbook contacts 1337

# PUT /contacts/1337
addressbook contacts 1337 set '{"id":"1337","name":"Smith","email":"j.smith@example.com"}'

# GET and PUT /contacts/1337/note
addressbook contacts 1337 note
addressbook contacts 1337 note set '{"content":"Call back on Monday"}'

# POST /contacts/1337/poke
addressbook contacts 1337 poke

# DELETE /contacts/1337
addressbook contacts 1337 delete
```

Running `addressbook` on its own lists the top-level commands.

Entities are printed as indented JSON, unless their class overrides `ToString()`. A collection argument containing exactly one `-`, such as `0-9`, `10-` or `-5`, is read as a [range](paging.md) rather than an ID; IDs of that form cannot be addressed this way.

## 6. Use the exit codes in scripts

The `Executor` maps exceptions to exit codes, so scripts can react to specific errors:

| Exit code | Cause                                                                    |
| --------- | ------------------------------------------------------------------------ |
| 0         | Success                                                                  |
| 1         | Missing or invalid arguments                                             |
| 2         | `400 Bad Request`                                                        |
| 3         | `403 Forbidden`                                                          |
| 4         | `401 Unauthorized`, even after asking for a new token                    |
| 5         | `404 Not Found` or `410 Gone`, or no URI or token could be obtained      |
| 6         | `409 Conflict`, `412 Precondition Failed` or `416 Range Not Satisfiable` |
| 10        | Other HTTP error status                                                  |
| 11        | Invalid JSON                                                             |
| 12        | I/O error                                                                |
| 99        | Canceled                                                                 |

```bash
if ! addressbook contacts 1337 > /dev/null 2>&1; then
    echo "Contact 1337 is missing"
fi
```

## 7. Customize input and output

Override `OutputEntity` (or `OutputEntities` in collection commands) to print something other than JSON:

```csharp
class ContactCommand : ElementCommand<Contact>
{
    // ...

    protected override void OutputEntity(Contact entity)
        => Console.Write($"{entity.Name} <{entity.Email}>");
}
```

`Console` here is the command's `IConsole`, which writes to stdout, stderr and reads from stdin. Replace it with your own implementation to change how all output is formatted. Similarly, override `InputEntity` to build an entity from arguments such as `--name Smith` instead of JSON.

For endpoint types with no matching command, or to add operations of your own, derive from `EndpointCommand<TEndpoint>` and override `ExecuteInnerAsync`.
