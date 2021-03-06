## Welcome to ActiveORM


ActiveORM is a Java ORM framework.

The philosophy of ActiveORM is to provide a familiar API to Java developers who have been using Rails ActiveRecord,while leveraging the power of all Hibernate.
ActiveORM is a really cool ORM in Java community corresponding to original Hibernate API.
Coding for fun Yay!

##Sample code

```sql
--tag
CREATE TABLE `tag` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `tag_synonym_id` int(11) DEFAULT NULL,
  `weight` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8;

--tag_synonym
CREATE TABLE `tag_synonym` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `tag_synonym_name` varchar(32) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

```

```java

public class Tag extends Model {
    @Validate
    private final static Map $name = map(
    presence, map("message", "name should not empty"),
    uniqueness, map("message", "name have already exists")
    );

    @ManyToOne
    private TagSynonym tag_synonym;

    public Association tag_synonym(){
      throw new AutoGeneration();
    }
}

public class TagSynonym extends Model {
    @OneToMany
    private List<Tag> tags = list();

    public Association tags(){
          throw new AutoGeneration();
        }
}


public class Usage{
   public void main(String[] args){

       TagSynonym tagSynonym = TagSynonym.where("tag_synonym_name=:name",map("name","wow")).limit(1).singleFetch();
       tagSynonym.tags().add(Tag.create(map("name","i am tag")));
       Tag tag = tagSynonym.tags().where("name=:name",map("name","i am tag"))).singleFetch();
       System.out.println(tag.attr("name",String.class));
       //or you can define name property in Tag so you can access name directly without using `attr` method to get name.
   }

}

```






## Installation

### Prerequisites
There are few things you need to have in your toolbox before tackling a web application using ActiveORM.

* A good to advanced knowledge of Java.
* Have good knowledge of your web framework if using one.
* A thorough understanding of ORM concepts.

Anyway ,you also  should notice that ActiveORM now is at version 1.0,and more important, for now it only support Mysql.

### Installation


I suppose you use it in a standard Servlet Container like Tomcat,Jetty. In order to make ActiveORM work properly,
you will write a filter like follows(the core code is in init.The name of filter as yourself)

For example:

```java

public class StartUpActiveActiveORMFilter implements Filter {

    public void doFilter(ServletRequest req, ServletResponse res,
            FilterChain chain) throws IOException, ServletException {
        chain.doFilter(req, res);
    }
    public void init(FilterConfig config) throws ServletException {
            // Actually this means you should put your ActiveORM configuration in a yaml file.And then load it.
            InputStream inputStream = FirstFilter.class.getResourceAsStream("application_for_test.yml");
            Settings settings = InternalSettingsPreparer.simplePrepareSettings(ImmutableSettings.Builder.EMPTY_SETTINGS,
                    inputStream);

            //when settings have been build , we can configure ActiveORM
            try//configure ORM
               JPA.CSDNORMConfiguration csdnormConfiguration = new JPA.CSDNORMConfiguration("development", settings, Main.class);
               JPA.configure(csdnormConfiguration);
            } catch (Exception e) {
                e.printStackTrace();
            }

    }
    public void destroy() {

    }
}

```

and then modify your web.xml file

```xml
<filter>
    <filter-name>StartUpActiveActiveORMFilter</filter-name>
    <filter-class>
        com.example.filters.StartUpActiveActiveORMFilter
    </filter-class>
</filter>
<filter-mapping>
    <filter-name>StartUpActiveActiveORMFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```



#### ActiveORM configuration file demo

```yaml
#mode
mode:
  development
#mode=production

###############datasource config##################
development:
    datasources:
        mysql:
           host: 127.0.0.1
           port: 3306
           database: wow
           username: me
           password: ****
           disable: false


production:
    datasources:
        mysql:
           host: 127.0.0.1
           port: 3306
           database: wow
           username: me
           password: ****


orm:
    show_sql: true
    pool_min_size: 5
    pool_max_size: 10
    timeout: 300
    max_statements: 50
    idle_test_period: 3000
###############application config##################
# tell ActiveORM where is your model classes


application:
    model:      com.example.model


###############validator config##################
validator:
   format:        net.csdn.validate.impl.Format
   numericality:  net.csdn.validate.impl.Numericality
   presence:      net.csdn.validate.impl.Presence
   uniqueness:    net.csdn.validate.impl.Uniqueness
   length:        net.csdn.validate.impl.Length
   associated:    net.csdn.validate.impl.Associated

################ 数据库类型映射 ####################
type_mapping:  net.csdn.jpa.type.impl.MysqlType

       
```

Oooops,forget something.you also need create META-INF/persistence.xml in your classpath(for example, put it in your src directory).
then copy&paste follow content to you file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://java.sun.com/xml/ns/persistence http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd"
             version="2.0">
    <persistence-unit name="{}">
        {}
        <properties>
        </properties>
    </persistence-unit>
</persistence>
```



#### Retrieving Objects from the Database

To retrieve objects from the database, ActiveORM provides several finder methods.
Each finder method allows you to pass arguments into it to perform certain queries on your database without
writing raw SQL.

The methods are:

* where
* select
* group
* order
* limit
* offset
* joins
* from

All of the above methods return an instance of net.csdn.jpa.model.JPQL or net.csdn.jpa.association.Association


1.1 Retrieving a Single Object

1.1.1 Using a Primary Key
Using Model.find(primary_key), you can retrieve the object corresponding to the specified primary key that matches any supplied options.
For example:

```java
//Find the tag with primary key (id) 10.
Tag tag = Tag.find(10)
```

1.1.2 Normal way
when you make sure that only one object will be found,then use method like this

```
 Tag tag = Tag.where("name=:name",map("name","java")).singleFetch();
```

1.2  Retrieving Multi Objects

1.2.1 Using multi primary keys

```java
//Find the tag with primary key (id) 10,11,12
List<Tag> tag = Tag.find(list(10,11,12))
```

1.2.2 Normal Way

```
List<Tag> tags = Tag.where("name=:name",map("name","java")).fetch();
```


1.3 Conditions

The where method allows you to specify conditions to limit the records returned, representing the WHERE-part of the SQL statement.
Conditions can only specified as a string,and you should pass parameters to them by named parameter.

for example:

```java
List<Tag> tags = Tag.where("name=:name and id > :id",map("name","java","id",10)).fetch();
```

1.4 Ordering

To retrieve records from the database in a specific order, you can use the order method.
For example, if you’re getting a set of records and want to order them in ascending order by the id field in your table:

```java
List<Tag> tags = Tag.order("id asc").fetch();
//or
List<Tag> tags = Tag.order("id asc,created_at desc").fetch();
```


1.5 Selecting Specific Fields

By default, Model.find selects all the fields from the result set using select *.
To select only a subset of fields from the result set, you can specify the subset via the select method.

```java
    List<Map> tags = Tag.select("id,name").fetch();
```

Notice that the return result is list of map.

1.6 Limit and Offset

```java
   List<Tag> tags = Tag.offset(0).limit(10).fetch();
```


1.7 Group

To apply a GROUP BY clause to the SQL fired by the finder, you can specify the group method on the find.

```java
List<Tag> tags = Tag.where("id>10").group("name").fetch();
```

1.8 Joining Tables

ActiveORM provides a finder method called joins for specifying JOIN clauses on the resulting SQL.

```java
  List<Tag> tags = Tag.joins("tag_synonym").fetch();
```
you also can join many tables at one time:

```java
List<Tag> tags = Tag.joins("tag_synonym left join  tag_groups left join blog_tags").fetch();
```

#### Name_Scope

Name_Scope is a powerful way to keep you code more objective and clean.
For example ,suppose you have a status filed in table like "status". Assume
status=1 is active tag. Normally,you will retrieve them using query like this:

```java
 List<Tag> activeTags = Tag.where("status=1").limit(10).fetch();
```

With Name_Scope,you can define a method in Model
For example:

```java
public class Tag extends Model {
    public static JPQL active(){
      return where("status=1");
    }
    // more content.......
}
```

now you can do it like this:

```java
List<Tag> tags = Tag.active().where("id>10").join("tag_groups").offset(0).limit(15).fetch();
```

Cool? HaHa


#### Associations

Suppose we wanted to add a new tag for an existing tag_synonym. We could  do something like this:

```java
tag_synonym.associate("tags").add(Tag.create(map("name","i am a new tag")));
```

At first glance you must be curious about `associate("tags")`.
In very model,there are many methods generated by ActiveORM,you cannot seen them but they definitely exits.
You can invoke them using `associate()` or 'm()' method.
Well,i guess you wanna IDE can give you some hint,so ,you can define a empty method and tell ActiveORM to implements it.
for example:

```java
class TagSynonym extends Model{

  public Association tags(){throw new AutoGeneration();}
  //other stuffs
  ....
}
```

Now ,you can invoke add a tag like this：

```java
  List<Tag> tags = tagSynonym.tags().where("id>10").fetch();
```
For ManyToMany relation,it is hard in hibernate ,you should write a lot code.

```java
tagGroup.getTags().add(tag);
tag.getTagGroups.add(tagGroup);
tag.save();
```

But with ActiveORM, you just do like this:

```
 tagGroup.tags().add(tag);
 //if you wanna delete tag
 tagGroup.tags().remove(tag);
```

So association in ActiveORM is easy to use.

For now ,ActiveORM support three kinds of relation.

*OneToOne
*OneToMany
*ManyToMany

#### Model methods and Instance Methods available when you extends  Model

Model static methods

```java
    Model.create(map)
    Model.deleteAll()
    Model.count()
    Model.count(String query, Object... params)
    Model.findAll()
    Model.findById(Object id)
    Model.all()
    Model.delete(String query, Object... params)

    Model.where(String whereCondition, Object... params)
    Model.join(String join)
    Model.order(String order)
    Model.offset(int offset)
    Model.limit(int limit)
    Model.select(String select)
```

Model instance methods

```java
    model.save()
    model.valid()
    model.update()
    model.refresh()
    model.delete()

```

#### Raw SQL Support

Sometimes you need to write complex sql,so ActiveORM support Raw sql.
It's easy to use.

```java
  List<Map> tags = Model.nativeSqlClient().execute("select * from tag where name=?","java");
```

#### Validations and Callbacks (coming soon...)









