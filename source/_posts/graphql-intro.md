---
title: GraphQL 初探
date: 2017-09-06 21:14:05
categories:
  - Backend
tags:
  - GraphQL
---

## 什么是 GraphQL

> GraphQL is a query language for APIs and a runtime for fulfilling those queries with your existing data.

正如 GraphQL 官网所描述，GraphQL 是一套数据查询语言，或者理解为一个数据抽象层，使得在不同的客户端可以使用相同的查询语法来获取数据。GraphQL 由 Facebook 在 2015 年提出，主旨在于替代 RESTFul API 查询，建立更高效和灵活的数据获取方式。和 RESTFul API 不同，GraphQL 从产品的角度出发，希望 API 能够灵活的处理复杂多变的用户场景，尤其是处理对象和关系时，能够优于现有 RESTFul API 的解决方案，这也是 GraphQL 中 graph 的来源。

<!-- more -->

## RESTFul API 缺陷

RESTFul API 作为目前业界前后端沟通的主要方式，在面临业务迅速发展或快速迭代时，暴露出越来越多的问题，尤其是在获取一些具有复杂关联性的资源数据时，RESTFul API 这种面向资源（对象）的请求方式变显得捉襟见肘。

下面以开发同学最喜闻乐见的博客程序作为例子说明问题所在，以 RESTFul API 获取完整的博客数据需要以下几步：

1. GET /posts 获取帖子列表
2. GET /posts/:id 根据 id 获取帖子详情
3. GET /posts/:id/comments 获取帖子的评论列表

当然，有的同学肯定要来打脸，“我一个 GET /posts 请求一步到位，哪有这么麻烦！”。

![laofu](http://destec.qiniudn.com//blog/graphql-intro/laofu.png)

没错，当业务简单或者数据量不大时一步到位是没有问题，但是当一个帖子里几十个字段、帖子数量几百条你试试这个接口能不能通过 code review 上线。在极尽压榨性能服务端，保持接口数据的精简是必要的，但是代价是会建立众多只有返回结构或字段不一样的 API endpoints 去满足不同业务方的需求。

另一方面，随着业务扩展和变化，数据的结构发生了变化时，如何在保证接口的兼容性基础上满足新的需求，也是让 RESTFul API 为难之处，一般只能通过新建一个版本的 API 或者建立新的 API 完成，但是这样降低了产品的可维护性。所以有没有可能让客户端开发同学像使用 SQL 查询一样，自定义的建立查询语句得到满足需求的数据呢？**GraphQL** 因此应运而生。

## GraphQL 组成

GraphQL 主要由 Schema，Operation 两部分概念组成。Schema 用于在服务端建立对数据结构的描述，确保了对于数据和查询的强类型约束和自省性，而 Operation 使得客户段可以使用 DSL 对数据进行查询，而这些查询语句可以根据业务的需求自由灵活的进行编写。

### Schema

使用 RESTFul API 去请求数据时，若不了解服务端实现和对象数据结构，是无法预测响应的结构的，而 GraphQL 则为数据建立了强类型约束，使得客户端开发同学可以在请求前获知返回结构如何，包括可以请求哪些字段、字段的类型、字段的含义，更棒的是这些均是可以嵌套的，也就是说如果返回结构中包括子对象，那么子对象的字段、字段的类型等也是可以获知的。当然这些便捷的功能需要建立在一些代码约定的基础上，类似于 ORM/ODM 的概念，这就是 schema 的作用。

由于 GraphQL 只是一套标准，可以由多种语言实现，所以标准里并没有使用某种语言限定的语法去完成 schema 的定义，而是建立了一套叫做 `GraphQL schema language` 的描述性语言去定义 schema，其表达方式和 JSON 类似，而各种语言的实现则遵循了这套语言标准。下面使用 GraphQL 的 JavaScript 实现，以博客数据结构作为例子，创建 schemas：

```js
const PostType = new GraphQLObjectType({
  name: 'PostType',
  fields: () => ({
    id: {
      type: GraphQLID,
      description: 'the unique id for the post',
    },
    title: {
      type: GraphQLString,
      description: 'the title for the post',
    },
    content: {
      type: GraphQLString,
      description: 'the full content for the post',
    },
    author: {
      type: GraphQLString,
      description: 'the author name for the post',
    },
    comments: {
      type: new GraphQLList(CommentType),
      description: 'the comments list',
    },
  }),
});
```

```js
const CommentType = new GraphQLObjectType({
  name: 'CommentType',
  fields: () => ({
    id: {
      type: GraphQLID,
      description: 'the unique id for the comment item',
    },
    user: {
      type: GraphQLString,
      description: 'the username for the comment item',
    },
    content: {
      type: GraphQLString,
      description: 'the full comment content for the comment item',
    },
  }),
});
```

以上的两段代码分别创建了名为 `PostType` 和 `CommentType` 的 schema，其中 `PostType` 包括 `id` `title` `content` `author` `comments` 这五个字段， `CommentType` 包括`id` `user` `content` 这三个字段。其中出现了 GraphQL 的几种数据类型，`GraphQLID` 可用于表示类主键型唯一的字符串数据，`GraphQLString` 表示字符串型数据，而 `GraphQLList` 则表示数组型数据。除此之外，GraphQL 还有 `GraphQLInt`、`GraphQLBoolean`、`GraphQLUnionType` 等类型，能满足一般常见的数据类型表达。

当然以上建立的仅仅是描述性的 schema，想要获取数据还需要给每个 schema 加上 `resolve` 函数，定义 数据获取的实现。最简单的，若想要返回固定的值，只需要定义:

```js
resolve: () => 'test post';
```

实际的开发中，数据的获取一般需要与持久化的数据层进行交互，例如从数据库获取数据，那么就需要把数据库的查询代码定义在 `resolve` 函数中并返回，另外条件查询需要也可以将条件参数通过 `resolve` 函数进行传参：

```js
resolve: async (_, args) => {
  return await db.Posts.find({
    where: {
      id: args.id,
    },
  });
};
```

### Operation

在完成了 schema 后，就可以进行对数据进行查询和修改了，Operation 可分为两类：

- query：用于对数据的只读操作
- mutation：用于对数据的写操作

#### query

先看一个简单的例子：

```json
query {
  post {
    id
    title
  	content
  	author
  }
}
```

返回结果：

```json
{
  "data": {
    "post": {
      "id": 1,
      "title": "the first post",
      "content": "some text",
      "author": "admin"
    }
  }
}
```

上面的代码是一个简单的 GraphQL 查询语句(query)和其返回结果，可以看到查询语句和 schema 一样，查询语句的表现形式和 JSON 也很接近，而返回结果就是标准的 JSON 格式，而且查询的结果和查询语句结构完全一致，这也就是 GraphQL 所宣传的：

> you always get back what you expect, and the server knows exactly what fields the client is asking for.

再看一个复杂点的的查询：

```json
query {
  post(author: 'admin', order: 'id DESC', limit: 2) {
    title
    content
    author
    comments {
      user
      content
    }
  }
}
```

返回结果：

```json
{
  "data": {
    "post": [
      {
        "id": 2,
        "title": "another post",
        "content": "some other text",
        "author": "admin",
        "comments": [
          {
            "user": "user1",
            "contents": "nice!"
          },
          {
            "user": "user2",
            "contents": "exthausted..."
          }
        ]
      },
      {
        "id": 1,
        "title": "the first post",
        "content": "some text",
        "author": "admin",
        "comments": [
          {
            "user": "user1",
            "contents": "boring post"
          },
          {
            "user": "user2",
            "contents": "great post!"
          }
        ]
      }
    ]
  }
}
```

这个查询表示按照 id 倒序排列 author 名为 “admin” 的前两条记录，而返回结果中需要包括 `title`, `content`, `author` 和 `comments` 中的 `user`, `content` 字段。

从这两个查询可以看出，使用 GraphQL 进行查询时，允许客户端对查询的条件，返回的字段等等常用的参数进行自定义，从而避免了数据冗余或者数据不完整的问题，避免了像 RESTFul API 那样，为了满足不同的客户端业务需求建立多个 endpoints 或者在某个 API 上编写使用额外的参数去约束返回的字段（而且这种情况还并不能满足所有的业务需求），同时对于嵌套的字段，GraphQL 查询也可以进行自定义。总而言之，使用 GraphQL 建立查询可以让客户端根据其需求充分的自定义查询条件，后期业务需求变动后，大部分情况下也无需后端更新代码。

#### mutation

当然 GraphQL 除了查询功能外，也支持对于数据的修改操作。和 query 类似，mutation 也需要在编写 schema 时增加相应的 `resolve` 函数，并在初始化 GraphQL server 时，将 mutation 进行注册，所以本文就不赘述了。

看到这里，有的同学可能会问，纵然 GraphQL 提供了灵活的查询功能，但是在客户端如何感知到数据结构和 operation 的 name 呢？对于这点，GraphQL 具有强大的自省能力，对于 schema 和 mutation 可以分别在客户端建立查询：

```json
query {
  __schema {
    queryType {
      name
      fields {
        name
      }
    }
  }
}
```

```json
query {
  __schema {
    mutationType {
      name
      fields {
        name
      }
    }
  }
}
```

schema 的查询返回结果：

```json
{
  "data": {
    "__schema": {
      "queryType": {
        "name": "Root",
        "fields": [
          {
            "name": "id"
          },
          {
            "name": "title"
          },
          {
            "name": "content"
          },
          {
            "name": "author"
          },
          {
            "name": "comments"
          }
        ]
      }
    }
  }
}
```

所以，通过内置的 \_\_schema 查询方法，可以自省的查询到当前可用的 schema 和 operation。这样对于后端开发同学编写接口的文档，而客户端开发也可以方便的了解接口的细节，甚至将自省查询放在业务查询前，进行字段检验。

## GraphQL 发展

目前 GraphQL 已经有多种语言实现的版本，服务端除了官方支持的 JavaScript（[graphql-js](https://github.com/graphql/graphql-js)）(github starts > 5000) 版本实现外，Java/Scala，Python，Golang 等语言均已经提供了社区版本，而在客户端，目前也提供了 JavaScript(React，React Native，Angular2 等) 和 iOS(Swift) 的 SDK。另外，官方还提供了一些方便开发者进行学习和调试的工具，例如可交互式的 IDE [graphiql](https://github.com/graphql/graphiql)。
