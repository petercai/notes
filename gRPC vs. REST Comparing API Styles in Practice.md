# gRPC vs. REST: Comparing API Styles in Practice 

In this article, we’ll explore APIs created using both the REST architecture and a fairly new player called gRPC. Our goal is to understand how to use these architectural styles to build APIs.

In 2000, Roy Fielding wrote his dissertation and introduced the world to an architecture for distributed systems called “Representational State Transfer,” or REST for short. He outlined several constraints for system architecture rooted in the principles that drive the web itself. The goal was to leverage decades of wisdom around building distributed systems that helped to ensure good separation of concerns and scalability.

Central to the REST philosophy is the concept of hypermedia and hypertext (like the “H” in HTTP). A RESTful API allows users to interact with resources identified by URLs and encoded in a variety of hypermedia representations (such as JSON, XML, or HTML). Modifying these resources changes the application state. Typically, clients use HTTP as the protocol of choice for building and interacting with RESTful services.

An often neglected but critical concept to REST is the idea of hypermedia as the engine of application state (HATEOAS). Simply put, instead of clients choosing what methods to invoke to change an application’s state, it’s the hypermedia itself that determines what resources a client can interact with and at what time. This is similar to how users interact with web pages today. Users only need to visit a single URL and links provided on the resulting page inform the user where they can browse.

REST is designed to leverage all of the existing technologies that make the web work, especially HTTP, TCP, URLs, and hypermedia types like JSON, XML, and HTML. To that end, no special software is required in order to build or use RESTful APIs.

The distributed client-server communication style known as “Remote Procedure Call” (RPC) is nothing new, dating back to at least the early 80s. However, Google’s implementation known as gRPC was invented in 2015 as a performance-centric framework for enabling web communication between systems. Built to leverage the newer HTTP/2 protocol under the hood, gRPC supports bidirectional communication between clients and servers in addition to streaming capabilities that enable multiple messages per request or response. This makes it very efficient for building high-performance, high-throughput applications. _Performance_ is the killer feature of gRPC.

Unlike RESTful APIs, RPC-based APIs define procedures (or methods) using an Interface Definition Language (IDL). Tools are required to generate server and client stub classes from the interface definition that can be used in a variety of programming languages. For gRPC, Google chose to develop its own format called “Protocol Buffers” (or “protobufs” for short) which benefit from a very efficient wire serialization format designed for speed.

Let’s take a look at some sample projects that demonstrate the distinctions between the REST and gRPC architectural styles. All of the projects we’ll examine are implemented as a full-stack address book apps with a simple front-end UI built on [Svelte](https://svelte.dev/) and a [NodeJS](https://nodejs.org/) back-end API. You can find the sample project monorepos on GitHub at [anthonydmays/grpc-vs-rest](https://github.com/anthonydmays/grpc-vs-rest).

To keep things simple, I only have a few features I’ve implemented along with several key constraints.

*   **Implement basic CRUD operations**. Nothing special here in the way of functional requirements. We just want to be able to list all of our contacts and create, update, or delete a single contact.
*   **Full-stack type safety**. We should be able to ensure that our entire codebase is type-safe and supports modern IDE niceties like autocomplete.
*   **Browser-consumable API**. We want to allow any clients to use our API from the browser environment.
*   **Unit-testable code**. It should be fairly trivial to write unit tests for our API to ensure that we have the behavior we expect.
*   **Minimal diffs**. In order to highlight the differences when moving between styles, I’ve done my best to minimize differences between the sample projects at the expense of some best practices. These demos are not designed to show you how to write production-ready systems, but rather to help you understand each style sufficiently.

First, let’s get the API and client up and running. Clone the `[anthonydmays/grpc-vs-rest](https://github.com/anthonydmays/grpc-vs-rest)` repo to your machine, open a terminal, and change it to the [rest-api-app](https://github.com/anthonydmays/grpc-vs-rest/tree/main/rest-api-app) directory. Follow the README instructions to install dependencies and build the `apiTypes` package. The commands are shown below for your convenience:

```
$ git clone https://github.com/anthonydmays/grpc-vs-rest  
$ cd rest-api-app  
$ npm install  
$ npm run build:apiTypes
```

In separate terminal windows, run the server by executing the `npm run dev:api` command and the client using `npm run dev:client`. You should be able to now browse to `http://localhost:5173` to see the basic UI.

Let’s turn our attention to the API for a moment. I’ll use `curl` to retrieve a single contact in order to demonstrate what makes our API conformant with REST principles. We'll see from the response how our API satisfies REST architectural constraints.

```
$ curl -iX GET http://localhost:9090/v1/contacts/1  
HTTP/1.1 200 OK  
X-Powered-By: Express  
Access-Control-Allow-Origin: *  
Content-Type: application/json; charset=utf-8  
Content-Length: 349  
ETag: W/"15d-bXqO1UHKOU8wml7G/sca1xVRseU"  
Date: Mon, 20 Feb 2023 22:54:24 GMT  
Connection: keep-alive  
Keep-Alive: timeout=5

{"resource":{"uri":"contacts/1","firstName":"Hedda","lastName":"Ready","email":"hready0@ftc.gov","phoneNumber":"919-521-1661","_links":{"self":{"href":"http://localhost:9090/v1/contacts/1","type":"GET"},"allContacts":{"href":"http://localhost:9090/v1/contacts","type":"GET"},"delete":{"href":"http://localhost:9090/v1/contacts/1","type":"DELETE"}}}}


```

*   **Consistent identification of resources**: Our API identifies resources by URL. For any resource available to us via the API, there is a unique URL that we can use to access or act upon it.
*   **Manipulation of resources through representation**: We can see a list of contacts by issuing an HTTP GET request to `http://localhost:9090/v1/contacts`. If we issue an OPTIONS request, we'll see that we can also `POST` to this resource to create a new contact if we wish. By sending the JSON representation of a contact, we'll be able to update the properties of that contact.
*   **Self-descriptive messages**: The `contacts` resource is represented using the `application/json` hypermedia format. We can see exactly the fields available to us and their basic types (string, numeric, array, and anything else JSON supports).
*   **Hypermedia as the engine of application state (HATEOAS)**: Importantly, the resource also provides us with links that tell us what other operations we can perform on this resource or other related resources.
*   **Cacheability**: Since we’re just using HTTP, we already have all the mechanisms we need to indicate whether a resource can be cached or not. We can also use ETags to convey this same information.
*   **Stateless**: Interactions between the client and our API server are stateless. Everything needed for the server to understand a request is provided in the request itself. The server does not store any state about previous requests or responses in order to fulfill new requests.

Now if we look at our client app, we’ll see something a bit unusual about the front-end implementation. Since our aim is to abide by RESTful constraints, we are required to use the links provided by our hypermedia resource in order to do anything with the API. It’s important to note that there is no URL construction happening in our client. We are simply using whatever we’re given from the API.

![](https://miro.medium.com/v2/resize:fit:700/0*IBjbDFdSap65_xmF.png)

```


  
<nav>  
  {#if data._links?.firstPage}  
    <a href="?url={encodeURIComponent(data._links.firstPage.href)}">{'<<'}</a> |  
  {/if}  
  {#if data._links?.previousPage}  
    <a href="?url={encodeURIComponent(data._links.previousPage.href)}">{'<'}</a>  
    |  
  {/if}  
  {#if data._links?.nextPage}  
    <a href="?url={encodeURIComponent(data._links.nextPage.href)}">{'>'}</a> |  
  {/if}  
  {#if data._links?.lastPage}  
    <a href="?url={encodeURIComponent(data._links.lastPage.href)}">{'>>'}</a>  
  {/if}  
</nav>


```

```


import { env } from '$env/dynamic/public';  
import type { ListContactsResponse } from '@grpc-vs-rest/api-types';  
import type { PageLoad } from './$types';

  
export const load = (async ({ url }) => {  
    
    
  const apiEndpoint =  
    url.searchParams.get('url') ||  
    env.PUBLIC\_API\_ENDPOINT ||  
    'http://localhost:9090/v1/contacts';

  const res = (await (  
    await fetch(`${apiEndpoint}`)  
  ).json()) as ListContactsResponse;

  return res;  
}) satisfies PageLoad;


```

Switching it up a bit, let’s now examine the gRPC API implementation which you’ll find in the [grpc-api-app](https://github.com/anthonydmays/grpc-vs-rest/tree/main/grpc-api-app) directory. The structure of this project is identical to our rest app, so I’ll just focus on the essential bits that have changed. Let’s start with the `apiTypes` project.

In order to define the contract by which the server and client will communicate with one another, we define a proto with the services, functions, parameters, and model definitions we need. We have a single proto defined in our `apiTypes` project under the namespace `proto.contacts.v1`.

```


  
service ContactsService {  
    
  rpc ListContacts(ListContactsRequest) returns (ListContactsResponse);

    
  rpc GetContact(GetContactRequest) returns (GetContactResponse);

    
  rpc UpdateContact(UpdateContactRequest) returns (UpdateContactResponse);

    
  rpc DeleteContact(DeleteContactRequest) returns (google.protobuf.Empty);

    
  rpc CreateContact(CreateContactRequest) returns (CreateContactResponse);  
}


```

The second big difference is that we now have auto-generated client and server stubs. For this task, I chose to use [buf](https://github.com/bufbuild/buf) and the [protobuf-ts](https://github.com/timostamm/protobuf-ts) plugin in order to generate idiomatic Typescript classes and objects. Not only do these classes describe the types we’ll use in the server and client, but also includes the actual gRPC implementations used to serialize and send messages back and forth across the wire.

```
  
version: v1  
plugins:  
  - name: ts  
    out: src/  
    opt: generate\_dependencies,long\_type\_string,server\_generic,client_generic
```

Next comes the server. The implementation is fairly trivial, thanks to the straightforward service interface produced by `protobuf-ts`. Note that, for this demo, we've ditched `Express` as the server and will instead host our service via the built-in `node:http2` module.

```


import api = require('@grpc-vs-rest/api-types');  
import { Server, ServerCredentials } from '@grpc/grpc-js';  
import { adaptService } from '@protobuf-ts/grpc-backend';  
import { ServerCallContext } from '@protobuf-ts/runtime-rpc';  
import {  
  createContact,  
  deleteContact,  
  getContact,  
  getContacts,  
  getContactsCount,  
  updateContact,  
} from './contacts.js';

export class ContactsService implements api.IContactsService {  
  async listContacts(  
    request: api.ListContactsRequest,  
    context: ServerCallContext,  
  ): Promise<api.ListContactsResponse\> {  
    let { pageNumber, pageSize, orderBy } = request;  
    pageSize = pageSize || 25;  
    pageNumber = pageNumber ?? 0;  
    const contacts = getContacts({ pageNumber, pageSize, orderBy });  
    return {  
      contacts,  
      pageNumber,  
      pageSize,  
      orderBy,  
      totalCount: getContactsCount(),  
    };  
  }  
    
}

const port = process.env.PORT || 9090;  
const server = new Server();  
server.bindAsync(  
  `0.0.0.0:${port}`,  
  ServerCredentials.createInsecure(),  
  () => {  
    server.start();  
    server.addService(  
      ...adaptService(api.ContactsService, new ContactsService()),  
    );  
    console.log(`server is running on 0.0.0.0:${port}`);  
  },  
);


```

If we now run our client and inspect requests to the API using a packet sniffer such as Wireshark, you’ll see that the response for the same contact retrieval method we implemented in REST is substantially smaller in size at 132 bytes compared to 349 bytes of JSON that was delivered previously.

![](https://miro.medium.com/v2/resize:fit:700/0*IMi_AEHyWIHNr1BD.png)

There is one more significant change to note. What might not be obvious is that the gRPC client code is required to run server-side (note that the load function for the main page is named `+page.server.ts` to denote a server-side rendered page). That’s because the base gRPC client library for JavaScript ([grpc/grpc-node](https://github.com/grpc/grpc-node)) used by `protobuf-ts` needs to run on NodeJS.

Browser support for HTTP/2 is still limited at the moment. That means that, unfortunately, our API is not browser-accessible as we set out to design it initially.

But what if we could make some changes to enable clients to interact with our gRPC API as though it were a REST service? Our service won’t be RESTful since it will still be RPC under the covers, but we will get to access it via HTTP/1.1, URLs, and JSON. Thankfully, we can use an [Envoy](https://envoyproxy.io/) proxy server to easily accomplish this feat without having to write our own code manually!

All the required changes can be viewed in our last demo, the [grpc-rest-app](https://github.com/anthonydmays/grpc-vs-rest/tree/main/grpc-rest-app) implementation. First, we need to update our proto service interface to help the proxy service make our gRPC service methods available at the right URLs and for the correct HTTP operations. To do this, the [Google API HTTP](https://github.com/googleapis/googleapis/blob/master/google/api/http.proto) library provides annotations we can add to our proto to describe the correct mappings. The `buf` tool allows us to include the `googleapis` dependency as a plugin in our `[buf.yaml](https://github.com/anthonydmays/grpc-vs-rest/blob/main/grpc-rest-app/packages/apiTypes/buf.yaml)` file).

```


  
service ContactsService {  
    
  rpc ListContacts(ListContactsRequest) returns (ListContactsResponse) {  
    option (google.api.http) = {  
      get: "/v1/contacts"  
    };  
  }

    
  rpc UpdateContact(UpdateContactRequest) returns (UpdateContactResponse) {  
    option (google.api.http) = {  
      put: "/v1/contacts/{id}"  
      body: "contact"  
    };  
  }

    
  rpc DeleteContact(DeleteContactRequest) returns (google.protobuf.Empty) {  
    option (google.api.http) = {  
      delete: "/v1/contacts/{id}"  
    };  
  }

    
}


```

As you can see in the example, we use the `google.api.http` annotation to indicate for each method what HTTP verb and URL path we want to make it accessible under.

We can now stand up an `Envoy` reverse proxy server instance that will be able to read these bindings and generate a service endpoint that will accept requests using the specified URIs. You'll find the implementation in the [packages/proxy](https://github.com/anthonydmays/grpc-vs-rest/tree/main/grpc-rest-app/packages/proxy) subfolder complete with a fully configured Docker Compose image. We'll require three things for this to work:

*   [**Envoy**](https://www.envoyproxy.io/). I chose to use the Docker image for simplicity, but you can also install via your favorite package manager and run the provided yaml on your own with a one-line change to the config (see [file comments](https://github.com/anthonydmays/grpc-vs-rest/blob/c4729ca7a7db2e0f5bdba2c81713175ad9bc3cf9/grpc-rest-app/packages/proxy/envoy.yaml#L63) for details).
*   **gRPC-JSON transcoder plugin**. We need to configure this plugin as a filter in Envoy in order to intercept HTTP requests with JSON payloads and transform them into gRPC-encoded messages in binary proto format.
*   **The proto descriptor file for our service**. Using the `buf build` command line option in our `apiTypes` package script, we're able to generate a proto descriptor file that will be used by the transcoder plugin to understand what services and operations are available.

In addition to starting our API and client servers as before, we’ll now need to run our proxy server using the `docker compose up` command. The client code is also updated to send fetch requests to our proxy service instead of to our backend API directly.

```


import { env } from '$env/dynamic/public';  
import type { ListContactsResponse } from '@grpc-vs-rest/api-types';  
import type { PageLoad } from './$types';

  
export const load = (async ({ url }) => {  
  const baseUrl = env.PUBLIC\_API\_ENDPOINT || 'http://localhost:8080';  
  const pageNumber = Number(url.searchParams.get('pageNumber')) || 0;  
  const orderBy = url.searchParams.get('orderBy');  
  const apiUrl = `${baseUrl}/v1/contacts?pageSize=25&pageNumber=${pageNumber}&orderBy=${orderBy}`;

  const res = (await (await fetch(apiUrl)).json()) as ListContactsResponse;

  return res;  
}) satisfies PageLoad;


```

The flexibility of this new approach is that services that want to speak to our API using native gRPC are free to do so, while clients that need to communicate via JSON and HTTP/1.1 won’t be left out in the cold.

Since we’re using Envoy, there’s one more neat trick that we can employ. It turns out that Envoy also support [gRPC-Web](https://github.com/grpc/grpc-web) out of the box, a JavaScript client designed to support gRPC communication from the browser! That means that we can send gRPC messages over HTTP/1.1 as base64 encoded strings or as binary protobufs. Messages will be sent through our proxy and on to our backend service. The advantage of this is smaller and more efficient wire communication which should lead to better performance.

We’ve covered a lot in this post. I encourage you to take a look at the GitHub repo a bit more and play around to see how everything fits together.

Because things are still evolving in the gRPC ecosystem, new tools and capabilities are being added all the time to make things easier for devs. I recently discovered that `Buf` has [introduced a new library](https://buf.build/blog/connect-web-protobuf-grpc-in-the-browser) for supporting gRPC apps that includes browser support for gRPC, gRPC-Web, and their own Connect Protocol for RPC communication. Consider checking it out if you are looking to explore gRPC for your next API.

Designing great APIs is hard. Hopefully, this article has presented you with some options for building APIs that scale well and support your use cases.

*   [gRPC vs REST: Understanding gRPC, OpenAPI and REST and when to use them in API design](https://cloud.google.com/blog/products/api-management/understanding-grpc-openapi-and-rest-and-when-to-use-them) by Martin Nally
*   [Architectural Styles and the Design of Network-based Software Architectures](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm) by Roy Fielding
*   [Use gRPC with Node.js and Typescript](https://dev.to/devaddict/use-grpc-with-node-js-and-typescript-3c58) by Aria Azadi Pour

