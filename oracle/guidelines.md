% Oracle SQL and PL/SQL Coding Guidelines
% Ian Hellström

# Introduction
To enable database developers on the same team to read one another's code more easily, and to have consistency in the code produced and to be maintained, a set of coding guidelines and standards is provided. The guidelines described in this document have been borrowed heavily from Steven Feuerstein's [Ideas for Oracle PL/SQL Naming Conventions and Coding Standards](http://www.toadworld.com/SF/standards), TopCoder's [Oracle PL/SQL Coding Guidelines](http://www.topcoder.com/i/development/uml/Oracle_PLSQL_Coding_Guidelines.pdf), Jonathan Ingram's [SQL and PL/SQL Coding Standards](http://www.dba-oracle.com/t_plsql_best_practices_standards.htm), and a 2008 white paper by Bryn Llewellyn entitled [Doing SQL from PL/SQL: Best and Worst Practices](http://www.oracle.com/technetwork/database/features/plsql/overview/doing-sql-from-plsql-129775.pdf).

# Source Control
All database objects have to be placed in a repository with revision control. Each entity has its own file.

By entity we mean a unit of code that belongs together. For instance, a package or object body and all its members form one entity. The package specification or object definition is another entity. A table, constraints on its columns, and related indexes are a single entity. Comments on the table and its columns are a separate entity though.

By grouping entities in separate files it is easy to navigate through the folder structure of the repository, run scripts against individual entities and objects, and create, compile, or alter individual components. 
 
Each file name is derived from a file prefix that based on the entity *type* it defines, the *name* of the entity itself, and a corresponding *file extension*: 

    PREFIX [WRAP] - entity_name.EXTENSION

Wrapped entities are indicated by WRAP appended to the file prefix. The prefix appendage WRAP is thus optional. For example, `PACKAGE WRAP - utilities.pkb` is the wrapped package body of the `utilities` package. The (unwrapped) source of that particular package body is of course also available in the repository and called `PACKAGE - utilities.pkb`.

The list of file prefixes and file extensions for the different entity types is shown below.

| Entity type                                                            | File prefix | File extension |
| :--------------------------------------------------------------------- | :---------- | :------------: |
| SQL code: `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `MERGE` (DML)        | SQL         | sql            |
| SQL code: comments (on a table or view, and its columns)               | COMMENT     | sql            |
| SQL code: directories                                                  | DIR         | sql            |
| SQL code: database links                                               | LINK        | sql            |
| SQL code: sequence                                                     | SEQUENCE    | sql            |
| SQL code: synonym(s)                                                   | SYNONYM     | sql            |
| SQL object (DDL): table (incl. indexes and constraints)                | TABLE       | sql            |
| SQL object (DDL): view                                                 | VIEW        | sql            |
| SQL object (DDL): materialized view (incl. indexes and constraints)    | MATVIEW     | sql            |
| SQL (object) type                                                      | TYPE        | sql            |
| PL/SQL function                                                        | FUNCTION    | pls            |
| PL/SQL procedure                                                       | PROCEDURE   | pls            |
| PL/SQL package specification                                           | PACKAGE     | pks            |
| PL/SQL package body                                                    | PACKAGE     | pkb            |
| PL/SQL queue                                                           | QUEUE       | pls            |
| PL/SQL scheduler definitions                                           | SCHEDULER   | pls            |
| PL/SQL script                                                          | SCRIPT      | pls            |
| PL/SQL trigger                                                         | TRIGGER     | pls            |

These prefixes may also be turned into a folder structure, where the folders are pluralized. For example, `packages` for the folder that contains all packages.

Out-of-code documentation is also included in the source code repository. A human-readable markup language/syntax, such as LaTeX, Markdown, or reStructuredText, is recommended as it too allows revision control on the source files. The file prefix for such documentation is DOC. The file extension depends on the markup language being used by all team members.

For large code bases it is recommended that these file prefixes be turned into folders and the hyphen (incl. surrounding spaces) be dropped from the files contained in these folders.

# Plural vs Singular
Objects that contain multiple pieces of the same type of information, such as tables and collections, are pluralized. A single row of such a pluralized object is singular. Likewise, a column or record is singular.

For instance, we may have the `employees` table or the `employees_ntt` nested table type. A local (variable) that is an instance of the type `employees_ntt` is called `l_employees`. A single row in a PL/SQL subprogram is named `l_employee`. More details on the naming conventions shown (esp. the `l_` prefix and `_ntt` suffix) can be found [below](#naming-conventions).

Whenever a variable (in PL/SQL) inherits the data type(s) with the `%TYPE` or `%ROWTYPE` attribute, the local identifier reflects the original name's grammatical number. For example, 

    l_salary     employees.salary%TYPE;
    l_employees  employees%ROWTYPE; 

# Style and Formatting Rules

## Capitalization
All *built-in* and *reserved* keywords and subprograms (e.g. packaged functions and procedures) are written in upper case. *Application-specific* identifiers are written in lower case to make it easier to identify the different components.

### Names and Word Separators
Object identifiers have to be self-explanatory; `display_full_call_stack` is to be preferred to `disp_fl_cl_stck` or something along these lines. If abbreviations have to be used because the identifiers are otherwise too long, stick to standard abbreviations (e.g. the ones already present in the data dictionary: "tab" for table, "privs" for privileges, etc.), and do not use the same abbreviation for different purposes. Please add any domain-specific abbreviations to the table below.

|  Abbreviation  | Explanation                               |
| :------------: | :---------------------------------------- |
| `dict`         | dictionary                                |
| `obj`          | object                                    |
| `proc`         | procedure                                 |
| `privs`        | privileges                                |
| `tab`          | table (or view)                           |

Words or abbreviations in any identifier are separated by an underscore, like so: `display_call_stack`. This is done to avoid 'API rate sheet' being read as 'A PIrate sheet'; thanks to [Baron Schwartz](http://www.xaprb.com/blog/2008/10/26/the-power-of-a-good-sql-naming-convention) for the fun, real-world example.

## Line Width and Line Breaks
The maximum line width is 120 characters. 

Line breaks are added after ``SELECT``, ``FROM``, ``WHERE``, and so on, before a comma in a ``SELECT``-list, before ``AND`` or ``OR`` in the ``JOIN`` or ``WHERE`` clause, and before the closing semicolon of a SQL statement. A line break before commas in the ``SELECT`` list makes commenting out all but the very first line so much easier in development. In case a long line spills over try to break the code up in natural pieces:

    SELECT
    , e.first_name
    , AVG(d.salary) 
        OVER ( PARTITION BY ...
               ORDER BY ... ) AS avg_salary  
    FROM
      employees e
    INNER JOIN
      departments d
    ON
      e.department = d.department
    ;

Line breaks are also added before and after `THEN` in conditional blocks and `LOOP` in loops (see [below](#indentation-and-alignment) for an example):

    IF l_salary > g_max_salary
    THEN
      l_salary_exceeds_maximum := TRUE;
    ELSE
      l_salary_exceeds_maximum := FALSE;
    END IF;

Conditional blocks that only assign Booleans, as the one shown, are best written as follows:

    l_salary_exceeds_maximum := ( l_salary > g_max_salary ); 

The parentheses are optional but may be added for additional clarity.

Column aliases are always specified with `AS`. The keyword is technically optional but it makes making mistakes (such as forgetting or overlooking a comma) so much easier to spot.

## Indentation and Alignment
All code is aligned at the left. Application-specific identifiers in SQL statements (as shown [before](#line-width-and-line-breaks)) and blocks in PL/SQL are indented by two spaces:

    DECLARE
    ..l_output  VARCHAR2(20) := 'Hello, Oracle!';
    BEGIN
    ..FOR iteration IN 1..10
      LOOP
      ..DBMS_OUTPUT.PUT_LINE(l_output || ' - ' || iteration);
    ..END LOOP;
    END;

## `IS` vs `AS`
SQL and PL/SQL are both programming languages. As such, it is important to write not only syntactically correct sentences but also legible sentences. 

When you *create an object*, use `AS`. When you *define a member* function (of an object or package), use `IS`. The difference becomes obvious when you read the statements as ordinary English sentences. In the former case, `IS` does not create a normal sentence. `AS` is likewise not entirely without its problems in the latter.

Example `AS`:

    CREATE OR REPLACE PACKAGE package_name
    AS
      ...
    BEGIN
      ...
    END package_name;

Example `IS`:

    CREATE OR REPLACE PACKAGE BODY package_name
    AS
      ...
    BEGIN
      ...
      FUNCTION member_function
      IS
        ...
      BEGIN
        ...
      END member_function;

    END package_name;

## Join Syntax
In all instances is the ANSI syntax (i.e. ``[ INNER ] JOIN``, ``{ LEFT | RIGHT | FULL } [ OUTER ] JOIN``) to be preferred. Here, square brackets indicate optional keywords, and curly braces indicate mandatory keywords of which one of the options presented has to be supplied. 

The deprecated `(+)` syntax and/or comma-separated lists in the ``FROM`` clause with corresponding join predicates in the ``WHERE`` clause are to be avoided completely.

### Constraints and Indexes
Named constraints are required for primary keys, foreign keys, and check constraints. The type of constraint is appended as a suffix to the identifier, which itself is either the table name or the table name and table that is referenced (for a foreign-key constraint) or the column(s) that are involved in the constraint definition (for a check constraint).

Examples: 

    ALTER TABLE employees 
      ADD CONSTRAINT employees_pk
      PRIMARY KEY (employee_id);

    ALTER TABLE employees
      ADD CONSTRAINT employees_departments_fk
      FOREIGN KEY employees(department_id)
      REFERENCES departments(department_id);

    ALTER TABLE employees
      ADD CONSTRAINT employees_salary_ck
      CHECK (salary > 0);

When multiple columns are involved in a check constraint, an appropriate abbreviation may be used.

Indexes are also to be named. Their suffix is `_ix`. For instance,

    CREATE INDEX employees_id_names_ix(employee_id, last_name, first_name);

    CREATE INDEX emp_id_lname_fname_ix(employee_id, last_name, first_name); 

## Headers
In-code documentation is automatically generated with [PLDoc](http://pldoc.sourceforge.net), which requires the headers of entities and objects to follow certain rules. 

The header for package specifications follows the `AS` in `CREATE [ OR REPLACE ] PACKAGE package_name AS` and

    /**
     * Project:       The project's name or identifier
     *
     * Compatibility: Oracle Database 10g Release 1 and above
     * Author(s):     John Doe
     *                Jane Roe
     *                
     * Notes:         Add appropriate notes about the package's intended use
     *                or any restrictions that developers and users ought to
     *                be aware of
     * @headcom               
     */

A template for a function, procedure, type, or subtype's header is as follows:

    /** A brief description of what the PL/SQL module does. This should not be longer 
     * than a paragraph of 3-5 lines. Only the first sentence will end up becoming the
     * summary for the generated documentation.
     * NB: Add important notes where appropriate. In case you need a line break use the
     * HTML <br/> tag.
     * @param  formal_parameter_name  a short description of the formal parameter
     * @throws exception_name         a short description of when the exception is thrown
     * @return                        a short description of the RETURN value (only for functions) 
     */

When these definitions are inside a package the header is to be added *before* the definition. For standalone modules (i.e. functions and procedures), the comment is placed after the `AS` that defines the module.

# Naming Conventions
The following table lists the various elements in PL/SQL code and how they ought to be named. Some of the elements can be defined at the schema level (SQL) too, for instance object types. 

| Element                                      | Model                       | Example(s)                                |
|:---------------------------------------------|:----------------------------|:------------------------------------------|
| Local variable                               | `l_<identifier>`            | `l_employee_salary`                       |
| Local constant                               | `c_<identifier>`            | `c_max_salary`                            |
| Global variable (defined in a package)       | `g_<identifier>`            | `g_manager_salary`                        |
| Global constant (defined in a package)       | `gc_<identifier>`           | `gc_max_manager_salary`                   |
| Types based on database columns with `%TYPE` | `<scope*>_<identifier>_t`   | `hire_date_t`</br>`l_hire_date_t`         |
| Subtypes of built-in types                   | `<scope*>_<identifier>_t`   | `string_t`</br>`l_positive_integer_t`     |
| Explicit cursor                              | `<scope*>_<identifier>_cur` | `employees_cur`</br>`l_employees_cur`     |
| Referenced cursor type                       | `<scope*>_<identifier>_ct`  | `employees_ct`</br>`l_employees_ct`       |
| Cursor variable                              | `<scope>_<identifier>_cv`   | `l_employees_cv`</br>`g_employees_cv`     |
| Record type                                  | `<scope*>_<identifier>_rt`  | `department_rt`</br>`l_department_rt`     |
| Record variable                              | `<scope>_<identifier>_rec`  | `l_department_rec`</br>`g_department_rec` |
| Associative-array (collection) type          | `<scope*>_<identifier>_aat` | `managers_aat`</br>`l_managers_aat`       | 
| Nested-table (collection) type               | `<scope*>_<identifier>_ntt` | `managers_ntt`</br>`l_managers_ntt`       |
| Variable array (collection) type             | `<scope*>_<identifier>_vat` | `managers_vat`</br>`l_managers_vat`       |
| Collection variable                          | `<scope>_<identifier>`      | `l_managers`</br>`g_managers`             |
| Object type                                  | `<identifier>_ot`           | `sales_stats_ot`                          |
| `IN` parameter                               | `<identifier>_in`           | `employee_id_in`                          | 
| `OUT` parameter                              | `<identifier>_out`          | `employee_id_out`                         |
| `IN OUT` parameter                           | `<identifier>_io`           | `employee_id_io`                          |
| Storage table (`CREATE TABLE ... STORE AS`)  | `<identifier>_st`           | `current_projects_st`                     |

The difference between `<scope>` and `<scope*>` is that the former is always required whereas the latter is only required when not defined globally in a package (specification). For instance, when we define an associative-array type in the package specification, we simply call it `employees_aat` although `g_employees_aat` would be acceptable too. If, however, we define it in the body and it is only accessible to the package (or even a subprogram), then we call it `l_employees_aat` to make it clear that it is local to our package or PL/SQL module. `<scope*>` applies to column-based types, subtypes, collection types, and cursor definitions yet not cursor or record variables.

In itself, we could call cursor variables, which are derived from referenced cursor types, the same as explicit cursors as they behave in similar ways. However, we want to emphasize that cursor variables do not refer to a single, static SQL statement, both explicit and implicit, but can and typically will be reused, repurposed, or passed as arguments to subprograms. Our naming conventions do not distinguish between strong referenced cursors (i.e. defined *with* a `RETURN` clause) and weak referenced cursors (i.e. defined *without* a `RETURN` clause) though.

For the record, associative arrays or index-by tables are defined by means of `TABLE OF ... INDEX BY`. They are may be sparse and they are *not* persistent, that is they are not stored in the database but they can be used in packages or locally in functions and procedures. Nested tables, declared with `TABLE OF`, and variable arrays, declared with `VARRAY OF`, are both persistent. In both persistent collection types, the order is preserved and they are dense. Variable arrays have a maximum size. Both have the `EXTEND` method. Object types are defined with `CREATE TYPE ... AS OBJECT`. 

# PL/SQL Modules 

## Initialization and Defaults
By default, all variables are initialized to `NULL` in PL/SQL. However, we recommend that all variables be initialized to `NULL` in the declaration section explicitly, just to make it clear that the 'uninitialized' case is covered. This is not necessary when variables are clearly initialized in the main block of a module.

Default parameters need to be specified where applicable. They can be specified with `:=` or `DEFAULT`. The former is to be preferred as it is shorter.

All PL/SQL functions, procedures, and packages are created by default as definer-rights modules (`AUTHID DEFINER`) instead of invoker-rights modules (`AUTHID CURRENT_USER`). All PL/SQL modules require the `AUTHID` clause, even when the default `AUTHID DEFINER` is implied. The reason for this rule is that it forces developers to carefully think about the implications, and it makes the original intentions clearer to all members of the team.

## Qualified Identifiers
All column and variable names in SQL statements inside PL/SQL code have to be fully qualified to avoid clashes when the data model evolves and includes (new) columns with the same identifiers as the variable names in PL/SQL.

## Functions
Functions should not contain `OUT` or `IN OUT` parameters: whatever needs to be returned must be returned from the `RETURN` statement.

## Function Calls
Functions and procedures called by other subprograms must use the named parameter notation (i.e. with `formal_parameter => actual_parameter`). This notation avoid problems when the order of formal parameters in interfaces changes. 

Custom parameterless functions or procedures that are called have empty parentheses added (e.g. `max_salary()`) to distinguish between a function or procedure call and a variable.

## Cursors
Results from a cursor are fetched into cursor records, not into multiple variables. This makes the code more robust in the face of future changes.

Single-row fetches are done in one step with an embedded (`SELECT ... INTO`) statement or `EXECUTE IMMEDIATE ... INTO` when dynamic SQL is called for.
 
When many rows are to be selected and the result set may have an arbitrary number of records, process the rows in batches using `FETCH ... BULK COLLECT INTO ... LIMIT`. The value for the `LIMIT` clause ought to be defined as a `CONSTANT PLS_INTEGER`. That way, a variable-size array can be used to fetch the records into. When, however, many rows are to be selected but the result set is of manageable maximum size, fetch all rows in a single step, that is, without the `LIMIT` option. In any case, use an explicit cursor when embedded SQL suffices; when it does not, use a cursor variable.
 
Always use `FORALL` instead of a cursor `FOR` loop when only one particular `INSERT`, `UPDATE`, or `DELETE` statement needs to be performed on a data set.

## Static vs Dynamic SQL
Embedded (static) SQL statements require aliases for all items in the `FROM`-list. Each column has to be dot-qualified with the appropriate alias.
Oracle-supplied objects (e.g. `DBMS_SQL`) must be dot-qualified with the owner (i.e. `SYS.DBMS_SQL`) to avoid potential invalidation due to name clashes.
 
Native dynamic SQL (NDS) is to be preferred to the `DBMS_SQL` API whenever dynamic SQL is required; NDS must not be used when the SQL statement is (defined as a) constant when the statement is a `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `MERGE`, or anonymous block. Furthermore, when using dynamic SQL, avoid concatenating literals. Instead, use bind variables to minimize both the number of hard parses and the risk of SQL injection. With regard to SQL injection, use `DBMS_ASSERT.ENQUOTE_LITERAL` and the like to safeguard against it.
 
Similarly, never concatenate elements of an `IN`-list whose size is only known at runtime. Instead, populate a collection with values and use the `TABLE` expression to generate the `IN`-list as follows: 

    WHERE col_name IN ( SELECT col_value FROM TABLE( collection_with_values ) )

This collection must be defined at the schema level.

If, on the one hand, the `SELECT`-list is known at compile time, use `DBMS_SQL.TO_REFCURSOR` to transform a numeric cursor to a cursor variable in combination with batched bulk fetch.

If, on the other hand, the `SELECT`-list is unknown at compile time, use `DBMS_SQL` to fetch the result set. In the special case where the bind variables are all known at compile time, use NDS to open a cursor variable and use `DBMS_SQL.TO_CURSOR_NUMBER` to transform it to a numeric cursor before fetching the results with `DBMS_SQL`.
 
## Identifier Recycling
Identifiers (i.e. variables) must not be used for more than one purpose in the same code fragment and/or in different scopes of the same entity. The same applies to custom exceptions too: an exception defines exactly one type of exceptional situation. 

Oracle's `NO_DATA_FOUND` exception is not one that should be emulated, as it is used for more than one of exception:

* No rows returned for a `SELECT INTO` statement.
* Attempted to reference a deleted element in a nested table or an uninitialized element of an associative array.
* Attempted to read past the end of file with `UTL_FILE`.

## The `NULL` Statement
The `NULL` statement may be added to improve the legibility of a particular code fragment:

    IF l_employee_salary <= g_max_salary
    THEN
      l_employee_salary := l_employee_salary + l_department_bonus;
    ELSE
      NULL; -- do nothing
    END IF;

## Spaghetti Code
Loops should have exactly one entry and one exit point. `FOR` and `WHILE` loops may not contain `EXIT [ WHEN ]` statements. 

More importantly, the `GOTO` statement will not be used in PL/SQL. Similarly, `RETURN` may not be specified inside a loop; the `RETURN` statement should be a function's last statement before the exception section.

## The End
Always add `END [ module_name ];` statements to all packages, procedures, functions, and the like. Do not leave it to a fellow programmer to figure out where a module ends.

# Exceptions
All exception handling is managed by a single utility, the `errors` package. Custom exceptions and custom names for built-in exceptions are defined in the package specification. No entity may contain definitions for exceptions or call `DBMS_OUTPUT` (or similar packages) to display error codes and messages.
