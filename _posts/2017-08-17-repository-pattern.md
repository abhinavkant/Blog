---
layout: post
title: "Repository & Unit of Work Pattern"
---

> Mediates between the domain and data mapping layers using a collection-like interface for accessing domain objects.
> ref: https://martinfowler.com/eaaCatalog/repository.html

This pattern provides a abstraction of data, so that the application can work with a simple abstraction that has an interface approximating that of a collection.

Using this pattern can help achieve a loose coupling and can keep domain object *persistence ignorant*.

### How should a repository code look like:

```cs
IRepository
    Add(obj)
    Remove(obj)
    Get(id)
    GetAll()
    Find(predicate)
```
### Things to Observe:

* No methods for Insert/Update/Delete, this is achieve by "Unit of Work" pattern. Unit of Work is referred to as a single transaction that involves multiple operations of insert/update/delete and so on kinds.
For a specific user action (say registration on a website), all the transactions like insert/update/delete and so are done in one single transaction, rather then doing multiple database transactions.
So, if one of the operation is failing then entire db operations will be rollback.

* Pattern should return IEnumerable not IQueryable, as this will give a wrong impression to upper layer that they can use this to build queries which is completely against the Repository Pattern.

### Logical way to structure the code:

```cs
// Generic Repository : work for all the entity
Repository<T> : IRepository<T> where t : class

// Custom Repository : Extension on top of Generic Repository for any specific entity.

ICourseRepository : IRepository<Course>
CourseRepository : Repository<Course>, ICourseRepository

IAuthorRepository : IRepository<Author>
AuthorRepository : Repository<Author>, IAuthorRepository
```

### Benefit:
* Minimize duplicate query logic.
* Decouple your application from persistence framework.
* Promotes testability.

## Unit of Work
Reiterating, Unit of Work is referred to as a single transaction that involves multiple operations of insert/update/delete and so on kinds.

### How should a UnitOfWork code look like:

```cs
IUnitOfWork
    ICourseRepository Courses { get; }
    IAuthorRepository Authors { get; }
    Complete() or Save()

UnitOfWork : IUnitOfWork
```

### Benefit:
All the transactions like insert/update/delete and so are done in one single transaction


