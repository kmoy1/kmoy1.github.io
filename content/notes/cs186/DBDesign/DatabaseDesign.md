# Database Design

The **entity-relationship (ER) model** is a core part of initial database design: it simplifies complex database data to a (special) directed graph of entities and relationships. It gives us an DBMS-implementational blueprint for what users want from their database. In this note set, we'll analyze the ER model and show just how useful it is in modeling various kinds of data. 

But before we discuss that, let's look at what database design really means first. 

### Database Design: Big Picture

The database design concept consists of six steps, the first three being most relevant to the ER model:

1. **Requirements Analysis**
2. **Conceptual Database Design**
3. **Logical Database Design**
4. Schema Refinement
5. Physical Database Design
6. Application and Security Design

#### Requirements Analysis

This first step is essential in *any* design task. We want to find out what our database's purpose is-what users want and get from the database. It incurs us to ask questions such as the type of data  to be stored in our DB, what applications we build on top (utilize our DB), the most frequent operations performed, etc. This is essentially the "brainstorm" step in which we collect important data on requirements. 

#### Conceptual DB Design

We use this brainstorm data to provide a basic description of our database data along with its **constraints**. That sounds a lot like our ER model, and indeed it is the step that utilizes it the most. Our description should represent how (ALL) people think of our database's data. 

#### Logical DB Design

We must choose a (relational) DBMS to implement our design. We must then convert our ER schema into a relational database schema. In the end, we produce a **logical schema** in the relational database model.





