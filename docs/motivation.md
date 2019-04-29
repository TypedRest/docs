There have been innumerable papers, posts and articles providing guidelines on how to design RESTful web services. Most of these guidelines are intended to make the behavior of web services more predictable for humans and machines alike. For example, the appropriate use of HTTP verbs allows developers to easily distinguish safe operations (such as `GET` and `HEAD`) from unsafe operations (such as `POST` and `DELETE`). Intermediate proxy servers can use this same information to determine candidates for caching. The plethora of HTTP headers provide many additional ways to express metadata and API structure in a standardized form.

While this slowly evolving consensus on "the right way" to design new APIs makes understanding such APIs easier, it does not necessarily help when actually consuming them. While HTTP libraries and tools expose all the underlying components (HTTP verbs/methods, headers, etc.) the burden of actually interpreting and combining all the pieces still lies with the developer. Let us take a look at an example:

Assume the URI `/contacts/` represents a collection of address book entries. `GET`ing the collection itself returns all entries. `GET`ing an URI like `/contacts/someid` gives you a single entry. `POST`ing to the collection adds a new element to it and returns the URI of the newly created resource using the `Location` header.

Pretty standard stuff, right? The problem is that all this knowledge currently only exists in your head. When you actually perform operations on the collection in your code you are usually manually building your `GET`s and your `POST`s while serializing and deserializing message bodies to and from JSON.

```csharp
var client = new HttpClient {BaseAddress = new Uri("http://example.com/")};

var contactsResponse = await client.GetAsync("contacts");
var contactList = await contactsResponse.Content.ReadAsAsync<List<ContactDto>>();

var janeResponse = await client.GetAsync("contacts/jane");
var jane = await janeResponse.Content.ReadAsAsync<ContactDto>();

await client.PostAsJsonAsync("contacts", new ContactDto("John"));
```

This is where TypedRest comes in. TypedRest is a .NET and Java library for consuming RESTful APIs that behave in a "predictable" way. Rather than applying your knowledge about how a REST collection usually behaves you simply tell the library that this particular endpoint *is* a collection and get a collection-like interface in return.

```csharp
var myService = new EntryEndpoint(new Uri("http://example.com/"));
var contacts = new CollectionEndpoint<ContactDto>(myService, relativeUri: "./contacts");

var contactList = await contacts.ReadAllAsync();
var jane = await contacts["jane"].ReadAsync();
await contacts.CreateAsync(new Contact("john"));
```

TypedRest uses a classic object-oriented approach to provide you with building blocks for modeling REST endpoints. Behavior of endpoints is described by inheritance while navigation between them is described by composition. For example, we could redesign our sample from above to make the service's functionality easy to discover and consume using code completion:

```csharp
class MyServiceClient : EntryEndpoint
{
  public MyServiceClient(Uri uri) : base(uri)
  {}

  public ICollectionEndpoint<ContactDto> Contacts => new CollectionEndpoint<ContactDto>(this, relativeUri: "./contacts");
}
```

The consuming code could look this:

```csharp
var myService = new MyServiceClient(new Uri("http://example.com/"));
var contactList = await myService.Contacts.ReadAllAsync();
var jane = await myService.Contacts["jane"].ReadAsync();
await myService.Contacts.CreateAsync(new ContactDto("john"));
```

TypedRest is all about nomenclature and patterns. An endpoint describes any resource addressable via an URI.

- An *entry endpoint* represents the top-level URI of an API. It takes care of shared concerns such as authentication.
- An *element endpoint* is a singular resource that can be read, modified and deleted.
- A *collection endpoint* can list and add elements as well as provide element endpoints for individual elements.

There are also a number of more specialized endpoint types such as pagination-aware collections. Each of these endpoint types has one or more corresponding classes in TypedRest. 

The only requirement on the server-side is that at least a part of the underlying pattern is implemented. For example, a collection endpoint class works as long as the server responds with an array of elements when `GET`ting the collection while `GET`ting a child endpoint provides a single element of the same type. If adding new elements via `POST` is not supported (yet) this will simply result in an exception at runtime when calling `.CreateAsync()`. For more graceful degradation TypedRest also exposes information about "allowed" methods as reported by `OPTIONS`.

We consider TypedRest's design to be opinionated yet pragmatic. The path of least resistance is to make your API match the patterns implemented in the built-in classes. These usually align with what is widely considered as "best practice". However:

- We explicitly support some unRESTful concepts such as RPC-like endpoints.
- HATEOAS-style, link-based navigation is possible but entirely optional.
- Link information is preferably encoded in HTTP headers instead of response bodies (although the latter is also supported in form of HAL).
- TypedRest does not use custom MIME types for API versioning, navigation, etc..

Of course, we don't expect our predefined patterns to cover all possible use cases. This where good old "extension through inheritance" comes into play. Let's say our sample API from above also allows us to store notes associated with a contact. We need to extend `ElementEndpoint` for individual `Contact` instances to expose this functionality. We also need to replace `CollectionEndpoint` with something that builds instances of our specialized element endpoint rather than using `ElementEndpoint`. Let's get coding!

```csharp
class MyServiceClient : EntryEndpoint
{
  public MyServiceClient(Uri uri) : base(uri)
  {}

  public ContactCollectionEndpoint Contacts => new ContactCollectionEndpoint(this);
}

// CollectionEndpointBase<TEntity, TElementEndpoint> is the superclass of CollectionEndpoint<TEntity>.
// The TElementEndpoint type argument allows you to customize the specific type of element endpoint it creates.
class ContactCollectionEndpoint : CollectionEndpointBase<Contact, ContactEndpoint>
{
  // Hard-coding the relative URI here makes the constructor signature nicer
  public ContactCollectionEndpoint(IEndpoint referrer) : base(referrer, relativeUri: "./contacts")
  {}
}

class ContactEndpoint : ElementEndpoint<ContactDto>
{
  public ContactEndpoint(IEndpoint referrer, Uri relativeUri) : base(referrer, relativeUri)
  {}

  public IElementEndpoint<NoteDto> Notes => new ElementEndpoint<NoteDto>(this, relativeUri: "./notes");
}
```

The consuming code could look this:

```csharp
var myService = new MyServiceClient(new Uri("http://example.com/"));
await myService.Contacts["jane"].Notes.SetAsync(new NoteDto("some note"));
```
