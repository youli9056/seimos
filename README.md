Seimos
======

Hibernate Criteria Encapsulation

Seimos is not a Hibernate substitute but an extension. It encapsulates repetitives issues when programming through 
Criteria. Here you are some examples:

1. Full list
------------
  1.1. Criteria

    Criteria criteria = session.createCriteria(Cat.class);    
    List cats = criteria.list();

  1.2. Seimos
  
    List cats = dao.list();
    
2. Adding restrictions
----------------------
  2.1. Criteria

    Criteria criteria = session.createCriteria(Cat.class);
    criteria.add(Restrictions.eq(“id”, 10000));
    List cats = criteria.list();
    
  2.2. Seimos
  
    Filters filters = new Filters();
    filters.add(new Filter(“id”, 10000));
    List cats = dao.find(filters);
    
  Of course both Criteria and Seimos have a lot of restrictions type. For Seimos each new Filter added to filters could use a specific condition for filtering. For example, new Filter's could be used as follows:
    filters.add(new Filter("description", "Pap"));
    filters.add(new Filter("birth", new Date()));
    filters.add(new Filter("serial", 10000, Condition.GREATER);  // Serial for cats? It's just an example :)
    
  And Condition could be any of 
    
  * EQUALS
  * NOT_EQUALS
  * STARTS_WITH
  * NOT_STARTS_WITH
  * ENDS_WITH
  * NOT_ENDS_WITH
  * EQUALS_CASE_INSENSITIVE
  * STARTS_WITH_CASE_SENSITIVE
  * NOT_STARTS_WITH_CASE_SENSITIVE
  * ENDS_WITH_CASE_SENSITIVE
  * NOT_ENDS_WITH_CASE_SENSITIVE
  * GREATER
  * GREATER_OR_EQUALS
  * LESS
  * LESS_OR_EQUALS
  * NULL
  * NOT_NULL
  * BETWEEN
  * OR
  * AND
  * IN
  * NOT_IN
  * EQ_PROPERTY
  * LESS_THAN_PROPERTY
  * LESS_OR_EQUALS_THAN_PROPERTY
  * GREATER_THAN_PROPERTY
  * GREATER_OR_EQUALS_THAN_PROPERTY
  
  For default, any search for an attribute of String type use MatchMode.START and 'like' clausule when Condition is supressed, i.e., new Filter("description", "Pap") it's the same of new Filter("description", "Pap", Condition.STARTS_WITH). For any other type, default condition is EQUALS, that means, new Filter("id", 10000) it's the same of new Filter("id", 10000, Condition.EQUALS).
    
  Filter's can be nested when Condition allows. Condition.AND and Condition.OR use constructor

    new Filter(Condition.AND, filter01, filter02)
    new Filter(Condition.OR, filter01, filter02)
  
  filter01 and filter02 are new Filter(...) of any type, including with Condition.AND or Condition.OR
    
3. Sorting
----------
  3.1. Criteria

    Criteria criteria = session.createCriteria(Cat.class);
    criteria.add(Restrictions.like(“description”, “Pap”)
      .addOrder(Order.asc(“description”);
    List cats = criteria.list();
    
  3.2. Seimos
  
    Filters filters = new Filters();
    filters.add(new Filters(“description”, “Pap”)
      .add(new Filter(“description”, Order.ASC));
    List cats = dao.find(filters);
    
  More Filter's with Order.ASC or Order.DESC can be added to filters.
  
4. Pagination
-------------
  4.1. Criteria

    Criteria criteria = session.createCriteria(Cat.class);
    criteria.add(Restrictions.like(“description”, “Pap”)
      .addOrder(Order.asc(“description”)
      .setFirstResult(101)
      .setMaxResult(10);
    List cats = criteria.list();

  4.2. Seimos
  
    Filters filters = new Filters();
    filters.add(new Filters(“description”, “Pap”)
      .add(new Filter(“description”, Order.ASC));
    List cats = dao.find(filters, 101, 10);
    
5. Associations
---------------
  Till now there's not great difference between Criteria and Seimos. But when using associations Seimos can help avoid a lot of code lines.

  5.1. Criteria
  
    Criteria criteria = session.createCriteria(Cat.class);
    criteria.add(Restrictions.like(“description”, “Pap”)
      .addOrder(Order.asc(“description”)
      .setFirstResult(101)
      .setMaxResult(10);
      
    Criteria subCriteria = criteria.createCriteria("kind", "kind");
    subCriteria.add(Restrictions.eq("description", "description"));
    
    List cats = criteria.list();  

  For deep association in more than one level another subCriteria is needed.
  
    Criteria criteria = session.createCriteria(Cat.class);
    criteria.add(Restrictions.like(“description”, “Pap”)
      .addOrder(Order.asc(“description”)
      .setFirstResult(101)
      .setMaxResult(10);
      
    Criteria subCriteria = criteria.createCriteria("kind", "kind");
    subCriteria.add(Restrictions.eq("description", "persa"));
    
    Criteria anotherSubCriteria = subCriteria.createCriteria("anAssociation", "anAssociation");
    anotherSubCriteria.add(Restrictions.eq("attribute", "anything"));
    
    /* ... and so on */
    
    List cats = criteria.list();  

  5.2. Seimos
  
    Filters filters = new Filters();
    filters.add(new Filters(“description”, “Pap”)
      .add(new Filter(“description”, Order.ASC))
      .add(new Filter("kind.description", "persa"))
      .add(new Filter("kind.anAssociation.attribute", "anything"));
    List cats = dao.find(filters, 101, 10);
    
  Seimos can either define JoinType when adding a Filter with association.
  
      .add(new Filter("kind.description", "persa", JoinType.LEFT_OUTER_JOIN))

6. Greedy results
-----------------
  Criteria results naturally fetchs a list of Objects but can also being transformed. Of course, in most cases, is necessary use a particular domain, DTO, VO, table mapping, bean or whatever. Criteria has an appropriate transformer for this cases.

  6.1. Criteria
    
    Criteria criteria = session.createCriteria(Cat.class);
    criteria.add(Restrictions.like(“description”, “Pap”)
      .addOrder(Order.asc(“description”)
      .setFirstResult(101)
      .setMaxResult(10);
      
    Criteria subCriteria = criteria.createCriteria("kind", "kind");
    subCriteria.add(Restrictions.eq("description", "persa"));
    
    Criteria anotherSubCriteria = subCriteria.createCriteria("anAssociation", "anAssociation");
    anotherSubCriteria.add(Restrictions.eq("attribute", "anything"));
    
    /* ... and so on */
    
    criteria.setResultTransformer(new AliasToBeanResultTransformer(Cat.class));
    
    List cats = criteria.list();  

  Hibernate has some transformers in its package. AliasToBeanResultTransformer above helps Criteria to fetch results within a list of Cat's. But when Cat have an association with another bean, AliasToBeanResultTransformer is not able to transform.
  
  6.2. Seimos
  
  The same code in 5.2 already embed a transformer for fetching a list of Cat's. Thus List can be strongly typed.
  
    Filters filters = new Filters();
    filters.add(new Filters(“description”, “Pap”)
      .add(new Filter(“description”, Order.ASC))
      .add(new Filter("kind.description", "persa"))
      .add(new Filter("kind.anAssociation.attribute", "anything"));
    List<Cat> cats = dao.find(filters, 101, 10);

