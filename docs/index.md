![Logo](images/logo.svg)

TypedRest helps you build type-safe, fluent-style REST API clients. Common REST patterns such as collections are represented as classes, allowing you to write more idiomatic code.

Read the **[Introduction](introduction.md)** to TypedRest or jump right in with the **[Getting started](getting-started/index.md)** guide.

<table>
<tr>
<th>HTTP</th>
<th>C#</th>
<th>Java</th>
</tr>
<tr>

<td>

```plain


GET /contacts

POST /contacts
  Location: /contacts/1337

GET /contacts/1337

PUT /contacts/1337/note

GET /contacts/1337/note

DELETE /contacts/1337
```

</td>

<td>

```csharp
var client = new MyClient(new Uri("http://example.com/"));

var contactList = await client.Contacts.ReadAllAsync();

var smith = await client.Contacts.CreateAsync(new Contact("Smith"));
//var smith = client.Contacts["1337"];

var contact = await smith.ReadAsync();

await smith.Note.SetAsync(new Note("some note"));

var note = await smith.Note.ReadAsync();

await smith.DeleteAsync();
```

</td>
<td>

```java
MyClient client = new MyClient(URI.create("http://example.com/"));

List<Contact> contactList = client.getContacts().readAll();

ContactEndpoint smith = client.getContacts().create(new Contact("Smith"));
//ContactEndpoint smith = client.getContacts().get("1337");

Contact contact = smith.read();

smith.getNote().set(new Note("some note"));

Note note = smith.getNote().read();

smith.delete();
```

</td>
</tr>
</table>
