---
title: "Identifiers (Entity SQL) | Microsoft Docs"
ms.custom: ""
ms.date: "03/30/2017"
ms.prod: ".net-framework"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "dotnet-ado"
ms.tgt_pltfrm: ""
ms.topic: "article"
dev_langs: 
  - "VB"
  - "CSharp"
  - "C++"
ms.assetid: d58a5edd-7b5c-48e1-b5d7-a326ff426aa4
caps.latest.revision: 2
author: "JennieHubbard"
ms.author: "jhubbard"
manager: "jhubbard"
---
# Identifiers (Entity SQL)
Identifiers are used in [!INCLUDE[esql](../../../../../../includes/esql-md.md)] to represent query expression aliases, variable references, properties of objects, functions, and so on. [!INCLUDE[esql](../../../../../../includes/esql-md.md)] provides two kinds of identifiers: simple identifiers and quoted identifiers.  
  
## Simple Identifiers  
 A simple identifier in [!INCLUDE[esql](../../../../../../includes/esql-md.md)] is a sequence of alphanumeric and underscore characters. The first character of the identifier must be an alphabetical character (a-z or A-Z).  
  
## Quoted Identifiers  
 A quoted identifier is any sequence of characters enclosed in square brackets ([]). Quoted identifiers let you specify identifiers with characters that are not valid in identifiers. All characters between the square brackets become part of the identifier, including all whitespace.  
  
 A quoted identifier cannot include the following characters:  
  
-   Newline.  
  
-   Carriage returns.  
  
-   Tabs.  
  
-   Backspace.  
  
-   Additional square brackets (that is, square brackets within the square brackets that delineate the identifier).  
  
 A quoted-identifier can include Unicode characters.  
  
 Quoted identifiers enable you to create property name characters that are not valid in identifiers, as illustrated in the following example:  
  
 `SELECT c.ContactName AS [Contact Name] FROM customers AS c`  
  
 You can also use quoted identifiers to specify an identifier that is a reserved keyword of [!INCLUDE[esql](../../../../../../includes/esql-md.md)]. For example, if the type `Email` has a property named "From", you can disambiguate it from the reserved keyword FROM by using square brackets, as follows:  
  
 `SELECT e.[From] FROM emails AS e`  
  
 You can use a quoted identifier on the right side of a dot (.) operator.  
  
 `SELECT t FROM ts as t WHERE t.[property] == 2`  
  
 To use the square bracket in an identifier, add an extra square bracket. In the following example "`abc]`" is the identifier:  
  
 `SELECT t from ts as t WHERE t.[abc]]] == 2`  
  
 For quoted identifier comparison semantics, see [Input Character Set](../../../../../../docs/framework/data/adonet/ef/language-reference/input-character-set-entity-sql.md).  
  
## Aliasing Rules  
 We recommend specifying aliases in [!INCLUDE[esql](../../../../../../includes/esql-md.md)] queries whenever needed, including the following [!INCLUDE[esql](../../../../../../includes/esql-md.md)] constructs:  
  
-   Fields of a row constructor.  
  
-   Items in the FROM clause of a query expression.  
  
-   Items in the SELECT clause of a query expression.  
  
-   Items in the GROUP BY clause of a query expression.  
  
### Valid Aliases  
 Valid aliases in [!INCLUDE[esql](../../../../../../includes/esql-md.md)] are any simple identifier or quoted identifier.  
  
### Alias Generation  
 If no alias is specified in an [!INCLUDE[esql](../../../../../../includes/esql-md.md)] query expression, [!INCLUDE[esql](../../../../../../includes/esql-md.md)] tries to generate an alias based on the following simple rules:  
  
-   If the query expression (for which the alias is unspecified) is a simple or quoted identifier, that identifier is used as the alias. For example, `ROW(a, [b])` becomes `ROW(a AS a, [b] AS [b])`.  
  
-   If the query expression is a more complex expression, but the last component of that query expression is a simple identifier, then that identifier is used as the alias. For example, `ROW(a.a1, b.[b1])` becomes `ROW(a.a1 AS a1, b.[b1] AS [b1])`.  
  
 We recommend that you do not use implicit aliasing if you want to use the alias name later. Anytime aliases (implicit or explicit) conflict or are repeated in the same scope, there will be a compile error. An implicit alias will pass compilation even if there is an explicit or implicit alias of the same name.  
  
 Implicit aliases are autogenerated based on user input. For example, the following line of code will generate NAME as an alias for both columns and therefore will conflict.  
  
```  
SELECT product.NAME, person.NAME  
```  
  
 The following line of code, which uses explicit aliases, will also fail. However, the failure will be more apparent by reading the code.  
  
```  
SELECT 1 AS X, 2 AS X …  
```  
  
## Scoping Rules  
 [!INCLUDE[esql](../../../../../../includes/esql-md.md)] defines scoping rules that determine when particular variables are visible in the query language. Some expressions or statements introduce new names. The scoping rules determine where those names can be used, and when or where a new declaration with the same name as another can hide its predecessor.  
  
 When names are defined in an [!INCLUDE[esql](../../../../../../includes/esql-md.md)] query, they are said to be defined within a scope. A scope covers an entire region of the query. All expressions or name references within a certain scope can see names that are defined within that scope. Before a scope begins and after it ends, names that are defined within the scope cannot be referenced.  
  
 Scopes can be nested. Parts of [!INCLUDE[esql](../../../../../../includes/esql-md.md)] introduce new scopes that cover entire regions, and these regions can contain other [!INCLUDE[esql](../../../../../../includes/esql-md.md)] expressions that also introduce scopes. When scopes are nested, references can be made to names that are defined in the innermost scope, which contains the reference. References can also be made to any names that are defined in any outer scopes. Any two scopes defined within the same scope are considered sibling scopes. References cannot be made to names that are defined within sibling scopes.  
  
 If a name declared in an inner scope matches a name declared in an outer scope, references within the inner scope or within scopes declared within that scope refer only to the newly declared name. The name in the outer scope is hidden.  
  
 Even within the same scope, names cannot be referenced before they are defined.  
  
 Global names can exist as part of the execution environment. This can include names of persistent collections or environment variables. For a name to be global, it must be declared in the outermost scope.  
  
 Parameters are not in a scope. Because references to parameters include special syntax, names of parameters will never collide with other names in the query.  
  
### Query Expressions  
 An [!INCLUDE[esql](../../../../../../includes/esql-md.md)] query expression introduces a new scope. Names that are defined in the FROM clause are introduced into the from scope in order of appearance, left to right. In the join list, expressions can refer to names that were defined earlier in the list. Public properties (fields and so on) of elements identified in the FROM clause are not added to the from-scope. They must be always referenced by the alias-qualified name. Typically, all parts of the SELECT expression are considered within the from-scope.  
  
 The GROUP BY clause also introduces a new sibling scope. Each group can have a group name that refers to the collection of elements in the group. Each grouping expression will also introduce a new name into the group-scope. Additionally, the nest aggregate (or the named group) is also added to the scope. The grouping expressions themselves are within the from-scope. However, when a GROUP BY clause is used, the select-list (projection), HAVING clause, and ORDER BY clause are considered to be within the group-scope, and not the from-scope. Aggregates receive special treatment, as described in the following bulleted list.  
  
 The following are additional notes about scopes:  
  
-   The select-list can introduce new names into the scope, in order. Projection expressions to the right might refer to names projected on the left.  
  
-   The ORDER BY clause can refer to names (aliases) specified in the select list.  
  
-   The order of evaluation of clauses within the SELECT expression determines the order that names are introduced into the scope. The FROM clause is evaluated first, followed by the WHERE clause, GROUP BY clause, HAVING clause, SELECT clause, and finally the ORDER BY clause.  
  
### Aggregate Handling  
 [!INCLUDE[esql](../../../../../../includes/esql-md.md)] supports two forms of aggregates: collection-based aggregates and group-based aggregates. Collection-based aggregates are the preferred construct in [!INCLUDE[esql](../../../../../../includes/esql-md.md)], and group-based aggregates are supported for SQL compatibility.  
  
 When resolving an aggregate, [!INCLUDE[esql](../../../../../../includes/esql-md.md)] first tries to treat it as a collection-based aggregate. If that fails, [!INCLUDE[esql](../../../../../../includes/esql-md.md)] transforms the aggregate input into a reference to the nest aggregate and tries to resolve this new expression, as illustrated in the following example.  
  
 `AVG(t.c) becomes AVG(group..(t.c))`  
  
## See Also  
 [Entity SQL Reference](../../../../../../docs/framework/data/adonet/ef/language-reference/entity-sql-reference.md)   
 [Entity SQL Overview](../../../../../../docs/framework/data/adonet/ef/language-reference/entity-sql-overview.md)   
 [Input Character Set](../../../../../../docs/framework/data/adonet/ef/language-reference/input-character-set-entity-sql.md)