Since you know about Spring Boot framework, you may have used JPA and Hibernate when you work with relational databases like MySQL / Postgres SQL…So, Hibernate ORM is an object–relational mapping tool for the Java programming language. It provides a framework for mapping an object-oriented domain model to a relational database. In more simple words, we can map the Java object to a table! Hibernate is the default implementation of JPA specification in Spring Boot.

This framework magically enhance the codebase with more reusability and maintainability. We can link just java objects using few annotations straight forward. According to the rules we specify, then data will be saved in the database tables.

I will be using a Spring Boot application with H2 in memory database to demonstrate the relationships practically. You can find the source code from here: https://github.com/SalithaUCSC/spring-boot-orm

All the dependencies used are available here: https://github.com/SalithaUCSC/spring-boot-orm/blob/main/pom.xml

Let’s start with some basics… 😎

Fetch Types in Hibernate
EAGER
Load the associated data of the other entity, beforehand which is bit costly.

LAZY
Load the associated data of the other entity, only when requested. This is done on demand.

There are specified fetching types for each relationship type which is applied by Hibernate by default.

OneToMany: LAZY
ManyToOne: EAGER
ManyToMany: LAZY
OneToOne: EAGER
Example:
If we have a relationship between university and student, when university data is fetched, we don’t want to fetch students right? Because, one university will have thousands of students in the students array in the mapping. It will be a very costly operation. So, we can tell hibernate to keep it with LAZY initialization.

We can decide how to that later…

Cascade Types in Hibernate
In Hibernate, Cascade Types mean how the data should be kept between the two related entities. There are set of pre defined types.

CascadeType.PERSIST : Both save() or persist() operations cascade to related entities.
CascadeType.MERGE : Related entities are merged when the ownership entity is merged.
CascadeType.REFRESH : Does same thing for the refresh() operation.
CascadeType.REMOVE : Removes all related entities association with this setting when the ownership entity is deleted.
CascadeType.DETACH : Detaches all related entities if a “manual detach” occurs.
CascadeType.ALL : All of the above cascade operations.
🔴 There is no default cascade type in JPA. By default, no operation is cascaded. If we want, we can use several cascade types at once also.

cascade = { CascadeType.PERSIST, CascadeType.MERGE, CascadeType.DETACH, CascadeType.REFRESH }
Let’s start to learn about object relational mapping with examples. 💪

One To One Mapping 💥
I’m going to build up a relationship between User and Address. Any User has an address here. Two users cannot have the same Address!

I don’t want the Address record, without the relevant User. I think it can be the most common scenario. In a one to one mapping, both entities are tightly coupled. After the User is removed, we cannot use his/her Address. So I will define CascadeType as ALL(If you want to keep the Address, change it to PERSIST). Then address won’t be deleted even we delete the user. Since Hibernate decides FetchType for one to one mapping is EAGER by default, I don’t want to mention it as a rule.

How relationship is built???

Normally we record child entity primary key as the foreign key of the owner entity. So User should have a column in the table to record the address ID. I have given its name as “address_id” and its referenced by “id” column in Address entity.

User

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    Long id;
    String name;
    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "address_id", referencedColumnName = "id")
    Address address;
}
Address

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "addresses")
public class Address {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    Long id;
    String street;
    String city;
    @JsonIgnore
    @OneToOne(mappedBy = "address")
    User user;
}
In the child entity(Adress), we just need to link the name of the property mapped in User entity.

“@JsonIgnore” annotation was placed there for user property since I do not need to have the user object to be seen in Address data. Just to ignore that field from JSON object.


Postman API response for a User
This way we can have a Bi directional one-to-one mapping! 💪

One To Many Mapping 💥
I’m creating another relationship between Post and Comment entities. Any kind of post can have one or more comments. But every comment is having only one post! So, this is a one-to-many relationship exactly…

I need to see the comments of each post, while post data is fetched. That means, comments should be eagerly fetched right? That’s why I have put fetch type as EAGER or comments set. Hibernate will set fetch type as LAZY by default for one to many mappings. To, override that, I have specifically mentioned the fetch type.

How relationship is built???

Tables should be connected in a way when the Comment table is having a column to place the primary key of the Post. That’s how we design the logical schema right…I have named the column “post_id” in this case. You may remember ER diagrams for this as I feel now.

Post

@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "posts")
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    Long id;
    String title;
    String description;
    @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER)
    @JoinColumn(name = "post_id", referencedColumnName = "id")
    Set<Comment> comments = new HashSet<>();
}
When a post is deleted, comments should not be there...It’s obvious. So, we can put CascadeType as ALL.

Comment

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "comments")
public class Comment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    Long id;
    String author;
    String content;
    @Column(name = "post_id")
    Long postId;
}
Comment is having the owning entity primary key as described before.

This way we can have a Uni directional one-to-one mapping! Actually in one to many scenarios, it is enough to have uni directional relationship. No need to be bi directional.💪 We just need to make sure that the primary key of the Post should be inserted into Comment object while a comment is saved.


Postman API response for a Post
Many To Many Mapping 💥
Do you have any idea how did we represent a many-to-many relationship in our logical design(ER diagram)?? This is a very special scenario, where we need an additional table more than the two entities.

Let me take an example and explain. In a Employee management system, every employee has one or more Role. Any employee can have one or more roles and Any role can have one or more employees! Then that is many to many right?

How relationship is built???

Here, we have to record, which employee has which role…That’s the need for 3rd table! Simply we can save “employee_id ”and “role_id ” to keep the employee and role associations. If there any other related things specific to this association, we can reserve more columns in the same table.

Employee

@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "employees")
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    Long id;
    String name;
    String email;
    @ManyToMany(fetch = FetchType.EAGER, cascade = CascadeType.ALL)
    @JoinTable(
        name = "employee_roles",
        joinColumns = @JoinColumn(
            name = "employee_id", referencedColumnName = "id"
        ),
        inverseJoinColumns = @JoinColumn(
            name = "role_id", referencedColumnName = "id"
        )
    )
    Set<Role> roles = new HashSet<>();

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Employee employee = (Employee) o;
        return id.equals(employee.id) && name.equals(employee.name) && email.equals(employee.email);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name, email);
    }
}
Employee is the owning entity in the relationship. So, it’s annotated with “@ManyToMany”. We have to place join table config here. So, “@JoinTable” annotation represents this new table with name as “employee_roles”. There we have to give the two columns we use for the associations. They are primary keys in employees and roles tables.

I need to show the user roles when user data is retrieved. Since hibernate considers LAZY as the default fetch type for ManyToMany mappings, I had to setup EAGER fetch there.

Role

@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "roles")
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    Long id;
    String name;
    @ManyToMany(mappedBy = "roles")
    @JsonIgnore
    Set<Employee> employees = new HashSet<>();

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Role role = (Role) o;
        return id.equals(role.id) && name.equals(role.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }
}
In the child entity, we just need to link the name of the property mapped in User entity.

“@JsonIgnore” annotation was placed there for employees property since I do not need to have the employee object to be seen in Role data.

This way we can establish a Bi directional one-to-one mapping! 💪

How data is saved in the 3rd table in our in-memory database is shown below, for this scenario.


Database table results
When we fetch employees, it will give the below result.


Postman API response for a Employee
That’s all guys! All the common relationship scenarios have been explained! 😍 According to the setup, uni or bi directional mappings can be there. I have taken some real world examples and tried to explain them all with code. We should always think from the aspect of real world entities and how they are going practically exist. Then our lives be be easier while coding!!!
