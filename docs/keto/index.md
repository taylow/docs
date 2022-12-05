---
id: index
title: Introduction
---

Ory Permissions (based on the open-source Ory Keto Permission Server) is the first and only open source implementation of
[Zanzibar: Google's Consistent, Global Authorization System"](https://research.google/pubs/pub48190/).

With Ory Permissions, you can

- centralize your authorization logic in one service that is the authorative source across all your apps and services,
- define your permission model (RBAC, ABAC, anything in between) in a language-agnostic way, with [SDKs](./sdk/01_index.md)
  available for all major programming languages.
- issue fine-grained permissions based on relationships (e.g., `user x is owner of project y`), or manage all permissions through
  groups, roles, etc.
- query your permissons across the world against the globally-distributed Ory Network.

And point to the high-level concepts:

## Concepts or Ory Permissions

### Relationships

At its core, Ory Permissions is a graph database where the nodes are subjects (things that we want to give permissions to) and
objects (things that we want to check permissions for). The edges in this graph are `relationships`. Therefore, a relationship
always comes as a tuple:

| User:Bob | is  | editor   | of  | Document:Readme |
| -------- | --- | -------- | --- | --------------- |
| Subject  |     | Relation |     | Object          |

The part before the colon (`User` and `Document`) are the namespaces of the subject and object.

The concrete permissions often depend not only on the relationships stored in Ory Permission, but on the rules that define what
these relationships imply. In the example above, being an `editor` might imply permissions to `read` and `write` the document, but
not to `approve` or `delete` it. These rules are defined in the Ory Permission Language.

### Permission language

The Ory Permission Language defines the rules and implications of relationships. The rules are defined in a subset of TypeScript.
A rule that gives editors of a document the `read` and `write` permisson looks like this:

```ts
class User implements Namespace {}

class Document implements Namespace {
  related: {
    editors: User[]
  }

  permits = {
    read: (ctx: Context): boolean => this.related.editors.includes(ctx.subject),
    write: (ctx: Context): boolean => this.related.editors.includes(ctx.subject),
  }
}
```

You can learn more in the [guide on using the Ory Permission Language](./guides/userset-rewrites.mdx).

### Check API

Use the `check` API to check whether a subject has a permission on an object. This API is primarily used to
[check permissions to restrict actions](./guides/simple-access-check-guide.mdx).

A check-request can include the maximum depth of the search tree. If the value is less than 1 or greater than the global max-depth
then the global max-depth will be used instead. This is to ensure low latency and limit the resource usage per request. To find
out more about Ory Keto's performance, head over to the [performance considerations](./performance.mdx).

For more details, head over to the [gRPC API reference](./reference/proto-api.mdx#checkservice) or
[REST API reference](./reference/rest-api.mdx#check-a-relation-tuple).

### List API

Use the `list` API to query [relationships](./concepts/relation-tuples.mdx) by providing a partial relation tuple. It is used to:

- [list objects a user has access to](./guides/list-api-display-objects.mdx#listing-objects)
- [list users who have a specific role](./guides/list-api-display-objects.mdx#listing-subjects)
- list users who are members of a specific group
- audit permissions in the system

For more details, head over to the [gRPC API reference](./reference/proto-api.mdx#readservice) or
[REST API reference](./reference/rest-api.mdx#query-relation-tuples).

### Expand API

At times, it is important to know which subjects have access to a given object. Use the `expand` API to recursively expand a
[subject set](./concepts/subjects.mdx#subject-sets) into a tree of subjects. For each subject, the tree assembles the relation
tuples including the operands as defined in the [namespace configuration](./concepts/namespaces.mdx).

An expand request can include the maximum depth of the tree to be returned. If the value is less than 1 or greater than the global
max-depth then the global max-depth will be used instead. This is required to ensure low latency and limit the resource usage per
request. To find out more about Ory Keto's performance, head over to the [performance considerations](./performance.mdx).

For more details, head over to the [gRPC API reference](./reference/proto-api.mdx#expandservice) or
[REST API reference](./reference/rest-api.mdx#getexpand).
