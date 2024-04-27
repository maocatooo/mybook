---
title: 为什么我们放弃了基于 GraphQL 的CQRS架构
date: 2023-07-13
tags: [思考]
#hide: true
#hidden: true
---
当接手项目后，一开始 Hasura GraphQL 确实为我们搭建数据表和进行连表查询提供了便利。在项目初期，它似乎非常有吸引力，能够迅速获得所需的基础数据，查询接口的编写速度快，前端可以无缝使用跨库跨表查询。

然而，随着项目的进一步开发，一些细节问题逐渐浮出水面，我们发现 GraphQL 并不适合我们的业务场景。有时，一个查询语句可能需要花费大量时间，而系统复杂度的增加经常导致以前的接口需要重构，这增加了开发成本。这让我们不禁思考，GraphQL 是否真的能提高开发效率。在选择上变得更加困难，因为维护它通常需要更多的成本。因此，我向领导提出了放弃 GraphQL 的建议。


1. **人员成本和学习成本**：在 Hasura GraphQL 初始阶段，后端配置了基础查询语句，但前端仍需添加细节。这导致后端开发人员不断跟在前端人员后面，前端人员也需要学习 GraphQL 知识, 这使得前后端在任务开发时是紧密耦合的，本来只用定义好开发文档，就能进行并行开发了。

2. **业务复杂度**：在业务开发中，通常会出现一些特殊的需求，这些需求不仅预料之外，而且超出了常规范围，需要特殊条件查询和字段过滤。例如，基于树状数据表进行特殊处理查询，这在 GraphQL 中很难实现，因为 GraphQL 是基于数据表而不是业务的。这可能需要在数据库层面编写大量函数来解决，进一步增加了系统的复杂性。并且业务不是一成不变的, GraphQL 配置中的 Schema 的维护又得单独维护

3. **数据权限控制**：Hasura GraphQL 的权限配置繁琐，开发人员需要大量时间来设置权限。随着系统角色的增加，配置变得更加复杂。如果没有正确配置数据的权限，当前用户对这些数据的查询将无法看到。虽然可以通过视图来解决权限问题，但引入视图会带来对其的维护成本。

4. **项目发布问题**：在正常系统中，只需在发布前修改或添加数据库数据表，项目就可以轻松上线。然而，在 Hasura GraphQL 中，需要在发布前修改数据库数据表，并相应地更新 Hasura GraphQL 配置，这使得发布过程变得复杂且容易出错，尤其在多个系统环境下。

5. **缓存**：使用 Hasura GraphQL 意味着失去了对缓存的控制权，这可能会对性能产生负面影响。

6. **日志**：在 GraphQL 中，与后端日志的精细连接变得困难，这会导致排查数据问题变得复杂。

7. **聚合数据**：在现代系统中，通常需要处理一些与业务智能 (BI) 相关的内容。在 GraphQL 中，聚合数据变得非常复杂，需要在数据库层面编写大量函数来解决。业务可能会不断变化，因此不希望每次都修改数据库层面的代码，但维护多一套的成本可能不可接受。

综上所述，项目中All In GraphQL，有时候并不能提高开发效率，反而会增加开发成本，维护成本。这也并不意味着 GraphQL 不适合所有项目，它仍然是一个很好的工具，可以在某些场景下提高开发效率。在选择 GraphQL 时，需要考虑项目的复杂度，以及是否有必要使用 GraphQL。