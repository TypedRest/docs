# TypeScript

## Getting started

### 1. Install the dependency

Add the `typedrest` NPM package to your project:

```bash
npm install typedrest --save
```

### 2. Create an entry endpoint

Create an `EntryEndpoint` pointing at your API:

```typescript
import {EntryEndpoint} from 'typedrest';

const client = new EntryEndpoint(new URL("https://example.com/api/"));
```

### 3. Define your client class

Extend `EntryEndpoint` and expose your API's resources as properties:

```typescript
import {EntryEndpoint, CollectionEndpoint} from 'typedrest';

class MyClient extends EntryEndpoint {
    get contacts() {
        return new CollectionEndpoint<Contact>(this, "./contacts");
    }
}
```

### 4. Use the client

Use your client to interact with the API:

```typescript
const client = new MyClient(new URL("https://example.com/api/"));

// Read all contacts
const contacts: Contact[] = await client.contacts.readAll();

// Create a new contact
const newContact = await client.contacts.create({name: "Smith"});

// Read a specific contact
const contact: Contact = await newContact.read();
```

### 5. Next steps

- Read the [Introduction](../introduction.md) for the full conceptual walkthrough
- Explore all available [Endpoints](../endpoints/index.md) types
- Check out the [API documentation](https://typescript.typedrest.net/) for detailed reference

## See also

- [API documentation](https://typescript.typedrest.net/)
- [GitHub repository](https://github.com/TypedRest/TypedRest-TypeScript)
