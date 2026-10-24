# Generate a client from OpenAPI

If your API has an [OpenAPI](https://swagger.io/resources/open-api/) document, TypedRest can write the client for you. This tutorial takes a spec for the [example address book API](index.md#the-example-api), generates a client from it, fixes an endpoint the generator could not have known about, and adds hand-written code on top.

The [Code generation](../code-generation/index.md) section describes each tool in detail. This page shows how they fit together.

## 1. The spec

Save this as `addressbook.yml`. Besides the paths from the example API it has one more: `/contacts/{id}/events`, which pushes a contact's changes as [Server-Sent Events](../endpoints/reactive/sse.md).

```yaml
openapi: 3.0.1
info:
  title: Address Book
  version: '1.0'
paths:
  /contacts:
    get:
      responses:
        '200':
          description: All contacts
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Contact'
    post:
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Contact'
      responses:
        '201':
          description: Created
  '/contacts/{id}':
    get:
      responses:
        '200':
          description: A contact
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Contact'
    put:
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Contact'
      responses:
        '204':
          description: Updated
    delete:
      responses:
        '204':
          description: Deleted
  '/contacts/{id}/note':
    get:
      responses:
        '200':
          description: The note attached to a contact
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Note'
    put:
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Note'
      responses:
        '204':
          description: Updated
  '/contacts/{id}/poke':
    post:
      responses:
        '204':
          description: Poked
  '/contacts/{id}/events':
    get:
      responses:
        '200':
          description: Changes to a contact as Server-Sent Events
          content:
            text/event-stream:
              schema:
                type: string
components:
  schemas:
    Contact:
      type: object
      required: [name]
      properties:
        id:
          type: string
        name:
          type: string
        email:
          type: string
    Note:
      type: object
      properties:
        content:
          type: string
    ContactEvent:
      type: object
      properties:
        type:
          type: string
        contact:
          $ref: '#/components/schemas/Contact'
```

## 2. Install the tool

```bash
dotnet tool install -g typedrest-codegen
```

The command-line tool needs the [.NET SDK](https://dotnet.microsoft.com/download) to run, but it can generate clients in every language. C# projects can use the [source generator](../code-generation/source-generator.md) instead, which is introduced in step 5. Install the CLI anyway, since step 3 uses it to inspect the inference.

## 3. Check what the generator infers

The generator does not translate paths one by one. It matches each path against [patterns](../code-generation/index.md#how-inference-works) such as "a `GET` returning an array, plus a child path returning one element of it is a collection". Before generating code, look at the result of that step:

```bash
typedrest-codegen pattern -f addressbook.yml
```

This writes the inferred structure back into `addressbook.yml` as an `x-typedrest` extension:

```yaml
x-typedrest:
  contacts:
    kind: collection
    uri: ./contacts
    description: All contacts
    schema:
      $ref: '#/components/schemas/Contact'
    element:
      kind: element
      description: A contact
      children:
        note:
          kind: element
          uri: ./note
          description: The note attached to a contact
          schema:
            $ref: '#/components/schemas/Note'
        poke:
          kind: action
          uri: ./poke
          description: Poked
        events:
          kind: blob
          uri: ./events
          description: Changes to a contact as Server-Sent Events
```

The descriptions are taken from the spec and end up as documentation comments in the generated code.

!!! warning "Run `pattern` only once"
    `pattern` always runs the inference again and replaces any `x-typedrest` already in the document. Running it after the edit in the next step would undo that edit. It also rewrites the whole file in a normalized form, so commit the spec before running it if you want to review the changes.

`/contacts` became a collection whose elements have a `note` element and a `poke` action, as intended. But `events` became a [blob endpoint](../endpoints/raw/blob.md): its response is not JSON, and an OpenAPI document cannot say that a response is a stream of entities rather than a file to download.

## 4. Fix the inference

Edit the `events` entry by hand to make it an SSE endpoint that delivers `ContactEvent` entities:

```yaml
        events:
          kind: sse
          uri: ./events
          description: Changes to a contact as Server-Sent Events
          schema:
            $ref: '#/components/schemas/ContactEvent'
```

When a document contains `x-typedrest`, the code generator (the `generate` command and the source generator) uses it as-is instead of running the inference again, so your edit survives every regeneration of the client. It also means that paths you add to the spec later have to be added to the extension too. [Custom code](../code-generation/custom-code.md#1-annotate-the-document) explains this and lists every `kind` and its properties.

## 5. Generate the client

=== "C#"

    Add the source generator and the libraries the generated code depends on. The generated `events` endpoint needs `TypedRest.Reactive`:

    ```bash
    dotnet add package TypedRest.SourceGenerator
    dotnet add package TypedRest
    dotnet add package TypedRest.Reactive
    ```

    Reference the spec in your project file:

    ```xml
    <ItemGroup>
      <TypedRestOpenApi Include="addressbook.yml" ServiceName="AddressBook" Namespace="MyCompany.AddressBook" />
    </ItemGroup>
    ```

    The client is generated on every build, together with interfaces and DTO classes. Set `<EmitCompilerGeneratedFiles>true</EmitCompilerGeneratedFiles>` if you want to read the generated files in `obj/`.

    If you would rather check the generated code in, use the CLI instead:

    ```bash
    typedrest-codegen generate -f addressbook.yml -o AddressBook/ -s AddressBook -n MyCompany.AddressBook --generate-interfaces --generate-dtos
    ```

=== "Java"

    ```bash
    typedrest-codegen generate -l java -f addressbook.yml -o src/main/java/ -s AddressBook -n com.mycompany.addressbook --generate-dtos
    ```

    Add the `net.typedrest:typedrest`, `net.typedrest:typedrest-serializers-jackson` and `net.typedrest:typedrest-reactive` dependencies, plus `org.jspecify:jspecify` for the nullability annotations on the DTOs.

=== "Kotlin"

    ```bash
    typedrest-codegen generate -l kotlin -f addressbook.yml -o src/main/kotlin/ -s AddressBook -n com.mycompany.addressbook --generate-dtos
    ```

    Add the `net.typedrest:typedrest` and `net.typedrest:typedrest-reactive` dependencies. The DTOs are annotated for kotlinx.serialization, so you also need the `kotlin("plugin.serialization")` Gradle plugin and an explicit `org.jetbrains.kotlinx:kotlinx-serialization-json` dependency.

=== "TypeScript"

    ```bash
    typedrest-codegen generate -l typescript -f addressbook.yml -o src/addressbook/ -s AddressBook --generate-dtos
    ```

    Run `npm install typedrest`. Reactive endpoints are part of the main package.

The [command-line tool](../code-generation/cli.md) page lists all options, including how to pick a different [serializer](../code-generation/index.md#serializers) for the DTOs.

## 6. Use the generated client

The service name `AddressBook` produces an entry endpoint called `AddressBookClient`. The collection becomes a `Contacts` property, its elements are `ContactElementEndpoint`s, and the children are named after their path segments:

=== "C#"

    ```csharp
    using MyCompany.AddressBook;

    var client = new AddressBookClient(new Uri("https://example.com/api/"));

    var smith = (await client.Contacts.CreateAsync(new Contact { Name = "Smith" }))!;
    await smith.Note.SetAsync(new Note { Content = "Call back on Monday" });
    await smith.Poke.InvokeAsync();

    // Print every change to the contact as it happens
    smith.Events.GetObservable().Subscribe(x => Console.WriteLine($"{x.Type}: {x.Contact?.Name}"));
    ```

=== "Java"

    ```java
    AddressBookClient client = new AddressBookClient(URI.create("https://example.com/api/"));

    Contact contact = new Contact();
    contact.name = "Smith";
    ContactElementEndpoint smith = client.getContacts().create(contact);
    smith.getNote().set(new Note("Call back on Monday"));
    smith.getPoke().invoke();

    // Print every change to the contact as it happens
    smith.getEvents().getObservable().subscribe(
        x -> System.out.println(x.type + ": " + (x.contact == null ? null : x.contact.name)),
        Throwable::printStackTrace);
    ```

    Generated Java DTOs have public fields; see [Command-line tool](../code-generation/cli.md#generating-a-client) for what else they look like.

=== "Kotlin"

    ```kotlin
    val client = AddressBookClient(URI.create("https://example.com/api/"))

    val smith = client.contacts.create(Contact(name = "Smith"))!!
    smith.note.set(Note(content = "Call back on Monday"))
    smith.poke.invoke()

    // Print every change to the contact as it happens
    smith.events.getObservable().subscribe { println("${it.type}: ${it.contact?.name}") }
    ```

=== "TypeScript"

    ```typescript
    import { AddressBookClient } from "./addressbook";

    const client = new AddressBookClient("https://example.com/api/");

    const smith = (await client.contacts.create({ name: "Smith" }))!;
    await smith.note.set({ content: "Call back on Monday" });
    await smith.poke.invoke();

    // Print every change to the contact as it happens
    for await (const x of smith.events.stream()) {
        console.log(`${x.type}: ${x.contact?.name}`);
    }
    ```

Compare this with the [hand-written client](first-client.md): the structure is the same, only the class names differ.

## 7. Add your own code

Generated code is regenerated, so do not edit it. Add to it instead, as described in [Extend the generated code](../code-generation/custom-code.md#2-extend-the-generated-code):

=== "C#"

    Add to the `partial` class and its `partial` interface in a file of your own. The collection hands out elements as `IContactElementEndpoint`, so a method that is only on the class would not be reachable from `client.Contacts["1337"]`:

    ```csharp
    namespace MyCompany.AddressBook;

    public partial interface IContactElementEndpoint
    {
        /// <summary>
        /// Changes the name of the contact, retrying if someone else modified it in the meantime.
        /// </summary>
        Task<Contact?> RenameAsync(string name, CancellationToken cancellationToken = default);
    }

    public partial class ContactElementEndpoint
    {
        public Task<Contact?> RenameAsync(string name, CancellationToken cancellationToken = default)
            => UpdateAsync(contact => contact.Name = name, cancellationToken: cancellationToken);
    }
    ```

    ```csharp
    await client.Contacts["1337"].RenameAsync("Smith");
    ```

=== "Java"

    Derive from the generated class:

    ```java
    public class MyAddressBookClient extends AddressBookClient {
        public MyAddressBookClient(URI uri) {
            super(uri);
        }

        public List<Contact> findByName(String name) {
            return getContacts().readAll().stream()
                .filter(x -> x.name.equals(name))
                .toList();
        }
    }
    ```

=== "Kotlin"

    Derive from the generated class, or use an extension function:

    ```kotlin
    fun AddressBookClient.findByName(name: String) = contacts.readAll().filter { it.name == name }
    ```

=== "TypeScript"

    Derive from the generated class:

    ```typescript
    export class MyAddressBookClient extends AddressBookClient {
        async findByName(name: string) {
            return (await this.contacts.readAll()).filter(x => x.name === name);
        }
    }
    ```

`RenameAsync` uses a read-modify-write cycle with ETags; see [Optimistic concurrency](concurrency.md).

## Keeping client and spec in sync

The source generator picks up a changed spec on the next build, while the CLI has to be rerun ([details](../code-generation/index.md#two-ways-to-use-it)). Either way, paths added to the spec must also be added to `x-typedrest`, as described in step 4.

## Next steps

- [Custom code](../code-generation/custom-code.md): every `x-typedrest` kind, and how to plug your own patterns into the generator.
- [Live updates](live-updates.md): more about the SSE endpoint used for `events`.
