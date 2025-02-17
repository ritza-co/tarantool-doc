
.. _sql_user_guide:

--------------------------------------------------------------------------------
SQL user guide
--------------------------------------------------------------------------------

The User Guide describes how users can start up with SQL with Tarantool, and necessary concepts.

.. container:: table

    .. rst-class:: left-align-column-1
    .. rst-class:: left-align-column-2

    +----------------------------------------------+---------------------------------+
    | Heading                                      | Summary                         |
    +==============================================+=================================+
    | :ref:`Getting Started                        | Typing SQL statements on a      |
    | <sql_getting_started>`                       | console                         |
    +----------------------------------------------+---------------------------------+
    | :ref:`Supported Syntax                       | For example what characters     |
    | <sql_supported_syntax>`                      | are allowed                     |
    +----------------------------------------------+---------------------------------+
    | :ref:`Concepts                               | tokens, literals, identifiers,  |
    | <sql_concepts>`                              | operands, operators,            |
    |                                              | expressions, statements         |
    +----------------------------------------------+---------------------------------+
    | :ref:`Data type conversion                   | Casting, implicit or explicit   |
    | <sql_data_type_conversion>`                  |                                 |
    +----------------------------------------------+---------------------------------+

.. _sql_getting_started:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Getting Started
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The explanations for installing and starting the Tarantool server are in earlier chapters of the Tarantool manual..

To get started specifically with the SQL features, using Tarantool as a client, execute these requests:

.. code-block:: lua

    box.cfg{}
    box.execute([[VALUES ('hello');]])

The bottom of the screen should now look like this:

.. code-block:: tarantoolsession

    tarantool> box.execute([[VALUES ('hello');]])
    ---
    - metadata:
      - name: COLUMN_1
        type: string
      rows:
      - ['hello']
    ...

That's an SQL statement done with Tarantool.

Now you are ready to execute any SQL statements via the connection. For example

.. code-block:: lua

    box.execute([[CREATE TABLE things (id INTEGER PRIMARY key,
                                       remark STRING);]])
    box.execute([[INSERT INTO things VALUES (55, 'Hello SQL world!');]])
    box.execute([[SELECT * FROM things WHERE id > 0;]])

And you will see the results of the SQL query.

For the rest of this chapter, the
:ref:`box.execute([[...]]) <box-sql>` enclosure will not be shown.
Examples will simply say what a piece of syntax looks like, such as
``SELECT 'hello';`` |br|
and users should know that must be entered as |br|
``box.execute([[SELECT 'hello';]])`` |br|
It is also legal to enclose SQL statements inside single or double quote marks instead of [[ ... ]].

.. _sql_supported_syntax:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Supported Syntax
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Keywords, for example CREATE or INSERT or VALUES, may be entered in either upper case or lower case.

Literal values, for example ``55`` or ``'Hello SQL world!'``, should be entered without single quote marks
if they are numeric, and should be entered with single quote marks if they are strings.

Object names, for example table1 or column1, should usually be entered without double quote marks
and are subject to some restrictions. They may be enclosed in double quote marks and in that case
they are subject to fewer restrictions.

Almost all keywords are :ref:`reserved <sql_reserved_words>`,
which means that they cannot be used as object names
unless they are enclosed in double quote marks.

Comments may be between ``/*`` and ``*/`` (bracketed)
or between ``--`` and the end of a line (simple).

.. code-block:: sql

    INSERT /* This is a bracketed comment */ INTO t VALUES (5);
    INSERT INTO t VALUES (5); -- this is a simple comment

Expressions, for example ``a + b`` OR ``a > b AND NOT a <= b``, may have arithmetic operators
``+ - / *``, may have comparison operators ``= > < <= >= LIKE``, and may be combined with
``AND OR NOT``, with optional parentheses.

.. _sql_concepts:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Concepts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the :ref:`SQL beginners' guide <sql_beginners_guide>` we discussed: |br|
What are: relational databases, tables, views, rows, and columns? |br|
What are: transactions, write-ahead logs, commits and rollbacks? |br|
What are: security considerations? |br|
How do we: add, delete, or update rows in tables? |br|
How do we: work inside transactions with commits and/or rollbacks? |br|
How do we: select, join, filter, group, and sort rows?

Tarantool has a "schema". A schema is a container for all database objects.
A schema may be called a "database" in other DBMS implementations

Tarantool allows four types of "database objects" to be created within
the schema: tables, triggers, indexes, and constraints.
Within tables, there are "columns".

Almost all Tarantool SQL statements begin with a reserved-word "verb"
such as INSERT, and end optionally with a semicolon.
For example: ``INSERT INTO t VALUES (1);``

A Tarantool SQL database and a Tarantool NoSQL database are the same thing.
However, some operations are only possible with SQL, and others are only
possible with NoSQL. Mixing SQL statements with NoSQL requests is allowed.

.. _sql_tokens:

********************************************************************************
Tokens
********************************************************************************

The token is the minimum SQL-syntax unit that Tarantool understands.
These are the types of tokens:

Keywords -- official words in the language, for example ``SELECT`` |br|
Literals -- constants for numbers or strings, for example ``15.7`` or ``'Taranto'`` |br|
Identifiers -- for example column55 or table_of_accounts |br|
Operators (strictly speaking "non-alphabetic operators") -- for example ``* / + - ( ) , ; < = >=``

Tokens can be separated from each other by one or more separators: |br|
* White space characters: tab (U+0009), line feed (U+000A), vertical tab (U+000B), form feed (U+000C), carriage return (U+000D), space (U+0020), next line (U+0085), and all the rare characters in Unicode classes Zl and Zp and Zs. For a full list see https://github.com/tarantool/tarantool/issues/2371. |br|
* Bracketed comments (beginning with ``/*`` and ending with ``*/``) |br|
* Simple comments (beginning with ``--`` and ending with line feed) |br|
Separators are not necessary before or after operators. |br|
Separators are necessary after keywords or numbers or ordinary identifiers, unless the following token is an operator. |br|
Thus Tarantool can understand this series of six tokens: |br|
``SELECT'a'FROM/**/t;`` |br|
but for readability one would usually use spaces to separate tokens: |br|
``SELECT 'a' FROM /**/ t;``

.. _sql_literals:

********************************************************************************
Literals
********************************************************************************

There are five kinds of literals: BOOLEAN INTEGER DOUBLE STRING VARBINARY.

BOOLEAN literals:  |br|
TRUE | FALSE | UNKNOWN |br|
A literal has :ref:`data type = BOOLEAN <sql_data_type_boolean>` if it is the keyword TRUE or FALSE.
UNKNOWN is a synonym for NULL.
A literal may have type = BOOLEAN if it is the keyword NULL and there is no context to indicate a different data type.

INTEGER literals: |br|
[plus-sign | minus-sign] digit [digit ...] |br|
or, for a hexadecimal integer literal, |br|
[plus-sign | minus-sign] 0X | 0x hexadecimal-digit [hexadecimal-digit ...] |br|
Examples: 5, -5, +5, 55555, 0X55, 0x55 |br|
Hexadecimal 0X55 is equal to decimal 85.
A literal has :ref:`data type = INTEGER <sql_data_type_integer>` if it contains only digits and is in
the range  -9223372036854775808 to +18446744073709551615, integers outside that range are illegal.

DOUBLE literals: |br|
[plus-sign | minus-sign] [digit [digit ...]] period [digit [digit ...]] |br|
[E|e [plus-sign | minus-sign] digit ...] |br|
Examples: .0, 1.0, 1E5, 1.1E5. |br|
A literal has :ref:`data type = DOUBLE <sql_data_type_double>` if it contains a period, or contains "E".
DOUBLE literals are also known as floating-point literals or approximate-numeric literals.
To represent "Inf" (infinity), write a real number outside the double-precision number range, for example 1E309.
To represent "nan" (not a number), write an expression that does not result in a real number,
for example 0/0, using Tarantool/NoSQL. This will appear as NULL in Tarantool/SQL.
In an earlier version literals containing periods were considered to be :ref:`NUMBER <sql_data_type_number>` literals.
In a future version "nan" may not appear as NULL.

STRING literals: |br|
[quote] [character ...] [quote] |br|
Examples: ``'ABC'``, ``'AB''C'`` |br|
A literal has :ref:`data type type = STRING <sql_data_type_string>`
if it is a sequence of zero or more characters enclosed in single quotes.
The sequence ``''``  (two single quotes in a row) is treated as ``'`` (a single quote) when enclosed in quotes,
that is, ``'A''B'`` is interpreted as ``A'B``.

VARBINARY literals: |br|
X|x [quote] [hexadecimal-digit-pair ...] [quote] |br|
Example: ``X'414243'``, which will be displayed as ``'ABC'``. |br|
A literal has :ref:`data type = VARBINARY <sql_data_type_varbinary>`
("variable-length binary") if it is the letter X followed by quotes containing pairs of hexadecimal digits, representing byte values.

Here are four ways to put non-ASCII characters,such as the Greek letter α alpha,  in string literals: |br|
First make sure that your shell program is set to accept characters as UTF-8. A simple way to check is |br|
``SELECT hex('α');``
If the result is CEB1 -- which is the hexadecimal value for the UTF-8 representation of α -- it is good. |br|
(1) Simply enclose the character inside ``'...'``, |br|
``'α'`` |br|
or |br|
(2) Find out what is the hexadecimal code for the UTF-8 representation of α,
and enclose that inside ``X'...'``, then cast to STRING because ``X'...'`` literals are data type VARBINARY not STRING, |br|
``CAST(X'CEB1' AS STRING)`` |br|
or |br|
(3) Find out what is the Unicode code point for α, and pass that to the :ref:`CHAR function <sql_function_char>`. |br|
``CHAR(945)  /* remember that this is α as data type STRING not VARBINARY */`` |br|
(4) Enclose statements inside double quotes and include Lua escapes, for example
``box.execute("SELECT '\206\177';")`` |br|
One can use the concatenation operator ``||`` to combine characters made with any of these methods.

Limitations: (`Issue#2344 <https://github.com/tarantool/tarantool/issues/2344>`_) |br|
* Numeric literals may be quoted, one cannot depend on the presence or
absence of quote marks to determine whether a literal is numeric. |br|
* ``LENGTH('A''B') = 3`` which is correct, but the display from
``SELECT A''B;`` is ``A''B``, which is misleading. |br|
* It is unfortunate that ``X'41'`` is a byte sequence which looks the same as ``'A'``,
but it is not the same. ``box.execute("select 'A' < X'41';")`` is not legal at the moment.
This happens because ``TYPEOF(X'41')`` yields ``'varbinary'``.
Also it is illegal to say ``UPDATE ... SET string_column = X'41'``,
one must say ``UPDATE ... SET string_column = CAST(X'41' AS STRING);``. |br|
* It is non-standard to say that any number which contains a period has data type = DOUBLE.

.. _sql_identifiers:

********************************************************************************
Identifiers
********************************************************************************

All database objects -- tables, triggers, indexes, columns, constraints, functions, collations -- have identifiers.
An identifier should begin with a letter or underscore (``'_'``) and should contain
only letters, digits, dollar signs (``'$'``), or underscores.
The maximum number of bytes in an identifier is between 64982 and 65000.
For compatibility reasons, Tarantool recommends that an identifier should not have more than 30 characters.

Letters in identifiers do not have to come from the Latin alphabet,
for example the Japanese syllabic ひ and the Cyrillic letter д are legal.
But be aware that a Latin letter needs only one byte but a Cyrillic letter needs two bytes,
so Cyrillic identifiers consume a tiny amount more space.

.. _sql_reserved_words:

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Reserved words
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Certain words are reserved and should not be used for identifiers.
The simple rule is: if a word means something in Tarantool SQL syntax,
do not try to use it for an identifier. The current list of reserved words is:

ALL ALTER ANALYZE AND ANY AS ASC ASENSITIVE AUTOINCREMENT
BEGIN BETWEEN BINARY BLOB BOOL BOOLEAN BOTH BY CALL CASE
CAST CHAR CHARACTER CHECK COLLATE COLUMN COMMIT CONDITION
CONNECT CONSTRAINT CREATE CROSS CURRENT CURRENT_DATE
CURRENT_TIME CURRENT_TIMESTAMP CURRENT_USER CURSOR DATE
DATETIME dec DECIMAL DECLARE DEFAULT DEFERRABLE DELETE DENSE_RANK
DESC DESCRIBE DETERMINISTIC DISTINCT DOUBLE DROP EACH ELSE
ELSEIF END ESCAPE EXCEPT EXISTS EXPLAIN FALSE FETCH FLOAT
FOR FOREIGN FROM FULL FUNCTION GET GRANT GROUP HAVING IF
IMMEDIATE IN INDEX INNER INOUT INSENSITIVE INSERT INT
INTEGER INTERSECT INTO IS ITERATE JOIN LEADING LEAVE LEFT
LIKE LIMIT LOCALTIME LOCALTIMESTAMP LOOP MATCH NATURAL NOT
NULL NUM NUMBER NUMERIC OF ON OR ORDER OUT OUTER OVER PARTIAL
PARTITION PRAGMA PRECISION PRIMARY PROCEDURE RANGE RANK
READS REAL RECURSIVE REFERENCES REGEXP RELEASE RENAME
REPEAT REPLACE RESIGNAL RETURN REVOKE RIGHT ROLLBACK ROW
ROWS ROW_NUMBER SAVEPOINT SCALAR SELECT SENSITIVE SESSION SET
SIGNAL SIMPLE SMALLINT SPECIFIC SQL START STRING SYSTEM TABLE
TEXT THEN TO TRAILING TRANSACTION TRIGGER TRIM TRUE
TRUNCATE UNION UNIQUE UNKNOWN UNSIGNED UPDATE USER USING VALUES
VARBINARY VARCHAR VIEW WHEN WHENEVER WHERE WHILE WITH

.. COMMENT:
   This is the Lua code that I (Peter Gulutzan) use for making the
   list of SQL reserved words.
   I assume the Tarantool 2.3 source is on /home/pgulutzan/tarantool-2.3
   I check whether I can create tables with names in the
   source file mkkeywordhash.c.
   This is only reliable if the database is new and empty.
   This is only reliable if mkkeywordhash.c keywords,
   and only keywords, are listed exactly this way:
   { "ROW_NUMBER",             "TK_STANDARD", RESERVED,         true  },
   I do not check whether mask = RESERVED or ALWAYS,
   because I would get false positives.
   statement = ''
   keyword = ''
   fh_string = ''
   fio = require('fio')
   fh = fio.open('/home/pgulutzan/tarantool-master/extra/mkkeywordhash.c', {'O_RDONLY'})
   fh_string = fh:read(100000)
   reserved_word_list = {}
   word_start = 1
   function f () local status local err status, err = box.execute(statement) if err == nil then return 0 else print(err) return 1 end end
   while true do
     i, word_start = string.find(fh_string, "\n  { \"", word_start)
     if i == nil then break end
     word_end = string.find(fh_string, "\"", word_start + 1)
     keyword = string.sub(fh_string, word_start+1, word_end-1)
     statement = "CREATE TABLE " .. keyword .. " (" .. keyword .. " INT PRIMARY KEY);"
     if f() == 1 then table.insert(reserved_word_list, keyword) end
     statement = "DROP TABLE IF EXISTS " .. keyword .. ";"
     if keyword ~= "END" and keyword ~= "IF" and keyword ~= "MATCH"
       and keyword ~= "RELEASE" and keyword ~= "RENAME" and keyword ~= "REPLACE"
       and keyword ~= "BINARY" and keyword ~= "CHARACTER" and keyword ~= "SMALLINT"
       then f() end
   end
   table.sort(reserved_word_list)
   fh:close()
   reserved_word_list

Identifiers may be enclosed in double quotes.
These are called quoted identifiers or "delimited identifiers"
(unquoted identifiers may be called "regular identifiers").
The double quotes are not part of the identifier.
A delimited identifier may be a reserved word and may contain
any printable character. Tarantool converts letters in regular
identifiers to upper case before it accesses the database,
so for statements like
``CREATE TABLE a (a INTEGER PRIMARY KEY);``
or
``SELECT a FROM a;``
the table name is A and the column name is A.
However, Tarantool does not convert delimited identifiers
to upper case, so for statements like
``CREATE TABLE "a" ("a" INTEGER PRIMARY KEY);``
or
``SELECT "a" FROM "a";``
the table name is a and the column name is a.
The sequence ``""`` is treated as ``"`` when enclosed in double quotes,
that is, ``"A""B"`` is interpreted as ``"A"B"``.

Examples: things, t45, journal_entries_for_2017, ддд, ``"into"``

Inside certain statements, identifiers may have "qualifiers" to prevent ambiguity.
A qualifier is an identifier of a higher-level object, followed by a period.
For example column1 within table1 may be referred to as table1.column1.
The "name" of an object is the same as its identifier, or its qualified identifier.
For example, inside ``SELECT t1.column1, t2.column1 FROM t1, t2;`` the qualifiers
make it clear that the first column is column1 from table1 and the second column
is column2 from table2.

The rules are sometimes relaxed for compatibility reasons, for example
some non-letter characters such as $ and « are legal in regular identifiers.
However, it is better to assume that rules are never relaxed.

The following are examples of legal and illegal identifiers.

.. code-block:: none

    _A1   -- legal, begins with underscore and contains underscore | letter | digit
    1_A   -- illegal, begins with digit
    A$« -- legal, but not recommended, try to stick with digits and letters and underscores
    + -- illegal, operator token
    grant -- illegal, GRANT is a reserved word
    "grant" -- legal, delimited identifiers may be reserved words
    "_space" -- legal, but Tarantool already uses this name for a system space
    "A"."X" -- legal, for columns only, inside statements where qualifiers may be necessary
    'a' -- illegal, single quotes are for literals not identifiers
    A123456789012345678901234567890 -- legal, identifiers can be long
    ддд -- legal, and will be converted to upper case in identifiers

The following example shows that conversion to upper case affects regular identifiers but not delimited identifiers.

.. code-block:: sql

    CREATE TABLE "q" ("q" INTEGER PRIMARY KEY);
    SELECT * FROM q;
    -- Result = "error: 'no such table: Q'.

.. _sql_operands:

********************************************************************************
Operands
********************************************************************************

An operand is something that can be operated on. Literals and column identifiers are operands. So are NULL and DEFAULT.

NULL and DEFAULT are keywords which represent values whose data types are not known until they are assigned or compared,
so they are known by the technical term "contextually typed value specifications".
(Exception: for the non-standard statement "SELECT NULL FROM table-name;"  NULL has data type BOOLEAN.)

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Operand data types
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Every operand has a data type.

For literals, :ref:`as we saw earlier <sql_literals>`, the data type is usually determined by the format.

For identifiers, the data type is usually determined by the definition.

The usual determination may change because of context or because of
:ref:`explicit casting <sql_function_cast>`.

For some SQL data type names there are *aliases*.
An alias may be used for data definition.
For example VARCHAR(5) and TEXT are aliases of STRING and may appear in
:samp:`CREATE TABLE {table_name} ({column_name} VARCHAR(5) PRIMARY KEY);` but Tarantool,
if asked, will report that the data type of :samp:`{column_name}` is STRING.

For every SQL data type there is a corresponding NoSQL type, for example
an SQL STRING is stored in a NoSQL space as :ref:`type = 'string' <index-box_string>`.

To avoid confusion in this manual, all references to SQL data type names are
in upper case and all similar words which refer to NoSQL types or to other kinds
of object are in lower case, for example:

* STRING is a data type name, but string is a general term;
* NUMBER is a data type name, but number is a general term.

Although it is common to say that a VARBINARY value is a "binary string",
this manual will not use that term and will instead say "byte sequence".

Here are all the SQL data types, their corresponding NoSQL types, their aliases,
and minimum / maximum literal examples.

.. container:: table

    .. rst-class:: left-align-column-1
    .. rst-class:: left-align-column-2
    .. rst-class:: left-align-column-3
    .. rst-class:: left-align-column-4

    +-----------+------------+------------+----------------------+-------------------------+
    | SQL type  | NoSQL type | Aliases    | Minimum              | Maximum                 |
    +===========+============+============+======================+=========================+
    | BOOLEAN   | boolean    | BOOL       | FALSE                | TRUE                    |
    +-----------+------------+------------+----------------------+-------------------------+
    | INTEGER   | integer    | INT        | -9223372036854775808 | 18446744073709551615    |
    +-----------+------------+------------+----------------------+-------------------------+
    | UNSIGNED  | unsigned   | (none)     | 0                    | 18446744073709551615    |
    +-----------+------------+------------+----------------------+-------------------------+
    | DOUBLE    | double     | (none)     | -1.79769e308         | 1.79769e308             |
    +-----------+------------+------------+----------------------+-------------------------+
    | NUMBER    | number     | (none)     | -1.79769e308         | 1.79769e308             |
    +-----------+------------+------------+----------------------+-------------------------+
    | STRING    | string     | TEXT,      | ``''``               | ``'many-characters'``   |
    |           |            | VARCHAR(n) |                      |                         |
    +-----------+------------+------------+----------------------+-------------------------+
    | VARBINARY | varbinary  | (none)     | ``X''``              | ``X'many-hex-digits'``  |
    +-----------+------------+------------+----------------------+-------------------------+
    | SCALAR    | scalar     | (none)     | FALSE                |  ``X'many-hex-digits'`` |
    +-----------+------------+------------+----------------------+-------------------------+

.. _sql_data_type_boolean:

BOOLEAN values are FALSE, TRUE, and UNKNOWN (which is the same as NULL).
FALSE is less than TRUE.

.. _sql_data_type_integer:

INTEGER values are numbers that do not contain decimal points and are
not expressed with exponential notation. The range of possible values is
between -2^63 and +2^64, or NULL.

.. _sql_data_type_unsigned:

UNSIGNED values are numbers that do not contain decimal points and are not
expressed with exponential notation. The range of possible values is
between 0 and +2^64, or NULL.

.. _sql_data_type_double:

DOUBLE values are numbers that do contain decimal points (for example 0.5) or
are expressed with exponential notation (for example 5E-1).
The range of possible values is the same as for the IEEE 754 floating-point
standard, or NULL. Numbers outside the range of DOUBLE literals may be displayed
as -inf or inf.

.. _sql_data_type_number:

NUMBER values have the same range as DOUBLE values.
But NUMBER values may also also be integers, and, if so,
arithmetic operation results will be exact rather than approximate.
For example, if a NUMBER column ``X`` contains 5, then ``X / 2`` is 2, an integer.
There is no literal format for NUMBER (literals like ``1.5`` or ``1E555``
are considered to be DOUBLEs), so use :ref:`CAST <sql_function_cast>`
to insist that a number has data type NUMBER, but that is rarely necessary.
See the description of NoSQL type :ref:`'number' <index-box_number>`.

.. _sql_data_type_string:

STRING values are any sequence of zero or more characters encoded with UTF-8,
or NULL. The possible character values are the same as for the Unicode standard.
Byte sequences which are not valid UTF-8 characters are allowed but not recommended.
STRING literal values are enclosed within single quotes, for example ``'literal'``.
If the VARCHAR alias is used for column definition, it must include a maximum
length, for example column_1 VARCHAR(40). However, the maximum length is ignored.
The data-type may be followed by :ref:`[COLLATE collation-name] <sql_collate_clause>`.

.. _sql_data_type_varbinary:

VARBINARY values are any sequence of zero or more octets (bytes), or NULL.
VARBINARY literal values are expressed as X followed by pairs of hexadecimal
digits enclosed within single quotes, for example ``X'0044'``.
VARBINARY's NoSQL equivalent is ``'varbinary'`` but not character string -- the
MessagePack storage is MP_BIN (MsgPack binary).

.. _sql_data_type_scalar:

SCALAR can be used for
:ref:`column definitions <sql_column_def_data_type>` but the individual column values have
one of the preceding types -- BOOLEAN, INTEGER, DOUBLE, STRING, or VARBINARY.
See more about SCALAR in the section
:ref:`Column definition -- the rules for the SCALAR data type <sql_column_def_scalar>`.
The data-type may be followed by :ref:`[COLLATE collation-name] <sql_collate_clause>`.

Any value of any data type may be NULL. Ordinarily NULL will be cast to the
data type of any operand it is being compared to or to the data type of the
column it is in. If the data type of NULL cannot be determined from context,
it is BOOLEAN.

All the SQL data types correspond to
:ref:`Tarantool/NoSQL types <details_about_index_field_types>` with the same name.
There are also some Tarantool/NoSQL data types which have no corresponding SQL data types.
If Tarantool/SQL reads a Tarantool/NoSQL value which has a type which has no SQL equivalent,
Tarantool/SQL may treat it as NULL or INTEGER or VARBINARY.
For example, ``SELECT "flags" FROM "_space";`` will return a column whose data type is ``'map'``.
Such columns can only be manipulated in SQL by
:ref:`invoking Lua functions <sql_calling_lua>`.

********************************************************************************
Operators
********************************************************************************

An operator signifies what operation can be performed on operands.

Almost all operators are easy to recognize because they consist of one-character
or two-character non-alphabetic tokens, except for six keyword operators (AND IN IS LIKE NOT OR).

Almost all operators are "dyadic", that is, they are performed on a pair of operands
-- the only operators that are performed on a single operand are NOT and ~ and (sometimes) -.

The result of an operation is a new operand. If the operator is a comparison operator
then the result has data type BOOLEAN (TRUE or FALSE or UNKNOWN).
Otherwise the result has the same data type as the original operands, except that:
promotion to a broader type may occur to avoid overflow.
Arithmetic with NULL operands will result in a NULL operand.

In the following list of operators, the tag "(arithmetic)" indicates
that all operands are expected to be numbers and should result in a number;
the tag "(comparison)" indicates that operands are expected to have similar
data types and should result in a BOOLEAN; the tag "(logic)"
indicates that operands are expected to be BOOLEAN and should result in a BOOLEAN.
Exceptions may occur where operations are not possible, but see the "special situations"
which are described after this list.
Although all examples show literals, they could just as easily show column identifiers.

.. _sql_operator_arithmetic:

.. _sql_operator_addition:

``+`` addition (arithmetic)
Add two numbers according to standard arithmetic rules.
Example: ``1 + 5``, result = 6.

.. _sql_operator_subtraction:

``-`` subtraction (arithmetic)
Subtract second number from first number according to standard arithmetic rules.
Example: ``1 - 5``, result = -4.

``*`` multiplication (arithmetic)
Multiply two numbers according to standard arithmetic rules.
Example: ``2 * 5``, result = 10.

``/`` division (arithmetic)
Divide second number into first number according to standard arithmetic rules.
Division by zero is not legal.
Division of integers always results in rounding down, use :ref:`CAST <sql_function_cast>` to DOUBLE to get
non-integer results.
Example: ``5 / 2``, result = 2.

``%`` modulus (arithmetic)
Divide second number into first number according to standard arithmetic rules.
The result is the remainder.
Example: ``17 % 5``, result = 2.

``<<`` shift left (arithmetic)
Shift the first number to the left N times, where N = the second number.
For positive numbers, each 1-bit shift to the left is equivalent to multiplying times 2.
Example: ``5 << 1``, result = 10.

``>>`` shift right (arithmetic)
Shift the first number to the right N times, where N = the second number.
For positive numbers, each 1-bit shift to the right is equivalent to dividing by 2.
Example: ``5 >> 1``, result = 2.

``&`` and (arithmetic)
Combine the two numbers, with 1 bits in the result if and only if both original numbers have 1 bits.
Example: ``5 & 4``, result = 4.

``|`` or (arithmetic)
Combine the two numbers, with 1 bits in the result if either original number has a 1 bit.
Example: ``5 | 2``, result = 7.

``~`` negate (arithmetic), sometimes called bit inversion
Change 0 bits to 1 bits, change 1 bits to 0 bits.
Example: ``~5``, result = -6.

.. _sql_operator_comparison:

``<`` less than (comparison)
Return TRUE if the first operand is less than the second by arithmetic or collation rules.
Example for numbers: ``5 < 2``, result = FALSE. Example for strings: ``'C' < ' '``, result = FALSE.

``<=`` less than or equal (comparison)
Return TRUE if the first operand is less than or equal to the second by arithmetic or collation rules.
Example for numbers: ``5 <= 5``, result = TRUE. Example for strings: ``'C' <= 'B'``, result = FALSE.

``>`` greater than (comparison)
Return TRUE if the first operand is greater than the second by arithmetic or collation rules.
Example for numbers: ``5 > -5``, result = TRUE. Example for strings: ``'C' > '!'``, result = TRUE.

``>=`` greater than or equal (comparison)
Return TRUE if the first operand is greater than or equal to the second by arithmetic or collation rules.
Example for numbers: ``0 >= 0``, result = TRUE. Example for strings: ``'Z' >= 'Γ'``, result = FALSE.

.. _sql_equal:

``=`` equal (assignment or comparison)
After the word SET, "=" means the first operand gets the value from the second operand.
In other contexts, "=" returns TRUE if operands are equal.
Example for assignment: ``... SET column1 = 'a';``
Example for numbers: ``0 = 0``, result = TRUE. Example for strings:  ``'1' = '2 '``, result = FALSE.

``==`` equal (assignment), or equal (comparison)
This is a non-standard equivalent of
:ref:`"= equal (assignment or comparison)" <sql_equal>`.

.. _sql_not_equal:

``<>`` not equal (comparison)
Return TRUE if the first operand is not equal to the second by arithmetic or collation rules.
Example for strings: ``'A' <> 'A     '`` is TRUE.

``!=`` not equal (comparison)
This is a non-standard equivalent of
:ref:`"\<\> not equal (comparison)" <sql_not_equal>`.

.. _sql_is_null:

``IS NULL`` and ``IS NOT NULL`` (comparison)
For IS NULL: Return TRUE if the first operand is NULL, otherwise return FALSE.
Example: column1 IS NULL, result = TRUE if column1 contains NULL.
For IS NOT NULL: Return FALSE if the first operand is NULL, otherwise return TRUE.
Example: ``column1 IS NOT NULL``, result = FALSE if column1 contains NULL.

.. _sql_operator_like:

``LIKE`` (comparison)
Perform a comparison of two string operands.
If the second operand contains ``'_'``, the ``'_'`` matches any single character in the first operand.
If the second operand contains ``'%'``, the ``'%'`` matches 0 or more characters in the first operand.
If it is necessary to search for either ``'_'`` or ``'%'`` within a string without treating it specially,
an optional clause can be added, ESCAPE single-character-operand, for example
``'abc_' LIKE 'abcX_' ESCAPE 'X'`` is TRUE because ``X'`` means "following character is not
special". Matching is also affected by the string's collation.

.. _sql_operator_between:

``BETWEEN`` (comparison)
:samp:`{x} BETWEEN {y} AND {z}` is shorthand for :samp:`{x} >= {y} AND {x} <= {z}`.

``NOT`` negation (logic)
Return TRUE if operand is FALSE return FALSE if operand is TRUE, else return UNKNOWN.
Example: ``NOT (1 > 1)``, result = TRUE.

``IN`` is equal to one of a list of operands (comparison)
Return TRUE if first operand equals any of the operands in a parenthesized list.
Example: ``1 IN (2,3,4,1,7)``, result = TRUE.

``AND`` and (logic)
Return TRUE if both operands are TRUE.
Return UNKNOWN if both operands are UNKNOWN.
Return UNKNOWN if one operand is TRUE and the other operand is UNKNOWN.
Return FALSE if one operand is FALSE and the other operand is (UNKNOWN or TRUE or FALSE).

``OR`` or (logic)
Return TRUE if either operand is TRUE.
Return FALSE if both operands are FALSE.
Return UNKNOWN if one operand is UNKNOWN and the other operand is (UNKNOWN or FALSE).

.. _sql_operator_concatenate:

``||`` concatenate (string manipulation)
Return the value of the first operand concatenated with the value of the second operand.
Example: ``'A' || 'B'``, result = ``'AB'``.

The precedence of dyadic operators is:

.. code-block:: none

    ||
    * / %
    + -
    << >> & |
    <  <= > >=
    =  == != <> IS IS NOT IN LIKE
    AND
    OR

To ensure a desired precedence, use () parentheses.

********************************************************************************
Special Situations
********************************************************************************

If one of the operands has data type DOUBLE, Tarantool uses floating-point arithmetic.
This means that exact results are not guaranteed and rounding may occur without warning.
For example, 4.7777777777777778 = 4.7777777777777777 is TRUE.

The floating-point values inf and -inf are possible.
For example, ``SELECT 1e318, -1e318;`` will return "inf, -inf".
Arithmetic on infinite values may cause NULL results,
for example ``SELECT 1e318 - 1e318;`` is NULL and ``SELECT 1e318 * 0;`` is NULL.

SQL operations never return the floating-point value -nan,
although it may exist in data created by Tarantool's NoSQL. In SQL, -nan is treated as NULL.

A string will be converted to a number if it is used with an arithmetic operator and conversion is possible,
for example ``'7' + '7'`` = 14.
And for comparison, ``'7'`` = 7.
This is called implicit casting. It is applicable for STRINGs and all numeric data types.

Limitations: (`Issue#2346 <https://github.com/tarantool/tarantool/issues/2346>`_) |br|
* Some words, for example MATCH and REGEXP, are reserved but are not necessary for current or planned Tarantool versions |br|
* 999999999999999 << 210 yields 0. (1 << 63) >> 63 yields -1.

.. _sql_expressions:

********************************************************************************
Expressions
********************************************************************************

An expression is a chunk of syntax that causes return of a value.
Expressions may contain literals, column-names, operators, and parentheses.

Therefore these are examples of expressions:
``1``, ``1 + 1 << 1``, ``(1 = 2) OR 4 > 3``, ``'x' || 'y' || 'z'``.

Also there are two expressions that involve keywords:

value IS [NOT] NULL |br|
  ... for determining whether value is (not) NULL

CASE ... WHEN ... THEN ... ELSE ... END |br|
  ... for setting a series of conditions.

See also: :ref:`subquery <sql_subquery>`.

Limitations: IS TRUE and IS FALSE return an error.

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Comparing and Ordering
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

There are rules for determining whether value-1 is "less than", "equal to", or "greater than" value-2.
These rules are applied for searches, for sorting results in order by column values,
and for determining whether a column is unique.
The result of a comparison of two values can be TRUE, FALSE, or UNKNOWN (the three BOOLEAN values).
Sometimes for retrieval TRUE is converted to 1, FALSE is converted to 0, UNKNOWN is converted to NULL.
For any comparisons where neither operand is NULL, the operands are "distinct" if the comparison
result is FALSE.
For any set of operands where all operands are distinct from each other, the set is considered to be "unique".

When comparing a number to a number: |br|
* infinity = infinity is true |br|
* regular numbers are compared according to usual arithmetic rules

When comparing any value to NULL: |br|
(for examples in this paragraph assume that column1 in table T contains {NULL, NULL, 1, 2}) |br|
* value comparison-operator NULL is UNKNOWN (not TRUE and not FALSE), which affects "WHERE condition" because the condition must be TRUE, and does not affect  "CHECK (condition)" because the condition must be either TRUE or UNKNOWN. Therefore SELECT * FROM T WHERE column1 > 0 OR column1 < 0 OR column1 = 0; returns only  {1,2}, and the table can have been created with CREATE TABLE T (... column1 INTEGER, CHECK (column1 >= 0)); |br|
* for any operations that contain the keyword DISTINCT, NULLs are not distinct. Therefore SELECT DISTINCT column1 FROM T; will return {NULL,1,2}. |br|
* for grouping, NULL values sort together. Therefore SELECT column1, COUNT(*) FROM T GROUP BY column1; will include a row {NULL, 2}. |br|
* for ordering, NULL values sort together and are less than non-NULL values. Therefore SELECT column1 FROM T ORDER BY column1; returns {NULL, NULL, 1,2}. |br|
* for evaluating a UNIQUE constraint or UNIQUE index, any number of NULLs is okay. Therefore CREATE UNIQUE INDEX i ON T (column1); will succeed.

When comparing a number to a STRING: |br|
* If implicit casting is possible, the STRING operand is converted to a number before comparison.
If implicit casting is not possible, and one of the operands is the name of a column which was
defined as SCALAR, and the column is being compared with a number, then number is less than STRING. Otherwise, the comparison is not legal.

When comparing a BOOLEAN to a BOOLEAN: |br|
TRUE is greater than FALSE.

When comparing a VARBINARY to a VARBINARY: |br|
* The numeric value of each pair of bytes is compared until the end of the byte sequences or until inequality. If two byte sequences are otherwise equal but one is longer, then the longer one is greater.

When comparing for the sake of eliminating duplicates: |br|
* This is usually signalled by the word DISTINCT, so it applies to SELECT DISTINCT, to set operators such as UNION (where DISTINCT is implied), and to aggregate functions such as  AVG(DISTINCT). |br|
* Two operators are "not distinct" if they are equal to each other, or are both NULL |br|
* If two values are equal but not identical, for example 1.0 and 1.00, they are non-distinct and there is no way to specify which one will be eliminated |br|
* Values in primary-key or unique columns are distinct due to definition.

When comparing a STRING to a STRING: |br|
* Ordinarily collation is "binary", that is, comparison is done according to the numeric values of the bytes. This can be cancelled by adding a :ref:`COLLATE clause <sql_collate_clause>` at the end of either expression. So ``'A' < 'a'`` and ``'a' < 'Ä'``, but ``'A' COLLATE "unicode_ci" = 'a'`` and ``'a' COLLATE "unicode_ci" = 'Ä'``. |br|
* When comparing a column with a string literal, the column's defined collation is used. |br|
* Ordinarily trailing spaces matter. So ``'a' = 'a  '`` is not TRUE. This can be cancelled by using the :ref:`TRIM(TRAILING ...) <sql_function_trim>` function. |br|

Limitations: |br|
* LIKE comparisons return integer results according to meta-information. |br|
* LIKE is not expected to work with VARBINARY.

********************************************************************************
Statements
********************************************************************************

A statement consists of SQL-language keywords and expressions that direct Tarantool to do something with a database.
Statements begin with one of the words
ALTER ANALYZE COMMIT CREATE DELETE DROP EXPLAIN INSERT PRAGMA RELEASE REPLACE ROLLBACK SAVEPOINT
SELECT SET START TRUNCATE UPDATE VALUES WITH.
Statements should end with ";" semicolon although this is not mandatory.

A client sends a statement to the Tarantool server.
The Tarantool server parses the statement and executes it.
If there is an error, Tarantool returns an error message.

SQL statements should end with ; (semicolon); this is not mandatory but it is recommended.

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
List of legal statements
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

In alphabetical order, the following statements are legal.

|nbsp| :ref:`ALTER TABLE table-name [RENAME or ADD CONSTRAINT or DROP CONSTRAINT clauses]; <sql_alter_table>` |br|
|nbsp| ANALYZE [table-name]; -- temporarily disabled in current version |br|
|nbsp| :ref:`COMMIT; <sql_commit>` |br|
|nbsp| :ref:`CREATE [UNIQUE] INDEX [IF NOT EXISTS] index-name <sql_create_index>` |br|
|nbsp| |nbsp| |nbsp| |nbsp| :ref:`ON table-name (column-name [, column-name ...]); <sql_create_index>` |br|
|nbsp| :ref:`CREATE TABLE [IF NOT EXISTS] table-name <sql_create_table>` |br|
|nbsp| |nbsp| |nbsp| |nbsp| :ref:`(column-or-constraint-definition <sql_create_table>` |br|
|nbsp| |nbsp| |nbsp| |nbsp| :ref:`[, column-or-constraint-definition ...]) <sql_create_table>` |br|
|nbsp| |nbsp| |nbsp| |nbsp| :ref:`[WITH ENGINE = engine-name]; <sql_create_table>` |br|
|nbsp| :ref:`CREATE TRIGGER [IF NOT EXISTS] trigger-name <sql_create_trigger>` |br|
|nbsp| |nbsp| |nbsp| |nbsp| :ref:`BEFORE|AFTER INSERT|UPDATE|DELETE ON table-name <sql_create_trigger>` |br|
|nbsp| |nbsp| |nbsp| |nbsp| :ref:`FOR EACH ROW <sql_create_trigger>` |br|
|nbsp| |nbsp| |nbsp| |nbsp| :ref:`BEGIN dml-statement [, dml-statement ...] END; <sql_create_trigger>` |br|
|nbsp| :ref:`CREATE VIEW [IF NOT EXISTS] view-name <sql_create_view>`  |br|
|nbsp| |nbsp| |nbsp| |nbsp| :ref:`[(column-name [, column-name ...])] <sql_create_view>`  |br|
|nbsp| |nbsp| |nbsp| |nbsp| :ref:`AS select-statement | values-statement; <sql_create_view>`  |br|
|nbsp| :ref:`DROP INDEX [IF EXISTS] index-name ON table-name; <sql_drop_index>`  |br|
|nbsp| :ref:`DROP TABLE [IF EXISTS] table-name; <sql_drop_table>`  |br|
|nbsp| :ref:`DROP TRIGGER [IF EXISTS] trigger-name; <sql_drop_trigger>` |br|
|nbsp| :ref:`DROP VIEW [IF EXISTS] view-name; <sql_drop_view>` |br|
|nbsp| :ref:`EXPLAIN explainable-statement; <sql_explain>` |br|
|nbsp| :ref:`INSERT INTO table-name <sql_insert>` |br|
|nbsp| |nbsp| |nbsp| |nbsp| :ref:`[(column-name [, column-name ...])] <sql_insert>` |br|
|nbsp| |nbsp| |nbsp| |nbsp| :ref:`values-statement | select-statement; <sql_insert>` |br|
|nbsp| :ref:`PRAGMA pragma-name[(value)]; <sql_pragma>` |br|
|nbsp| :ref:`RELEASE SAVEPOINT savepoint-name; <sql_release_savepoint>` |br|
|nbsp| :ref:`REPLACE INTO table-name VALUES (expression [, expression ...]); <sql_replace>` |br|
|nbsp| :ref:`ROLLBACK [TO [SAVEPOINT] savepoint-name]; <sql_rollback>` |br|
|nbsp| :ref:`SAVEPOINT savepoint-name; <sql_savepoint>` |br|
|nbsp| :ref:`SELECT [DISTINCT|ALL] expression [, expression ...] <sql_select>` |br|
|nbsp| |nbsp| |nbsp| |nbsp| :ref:`FROM table-name | joined-table-names [AS alias]  <sql_select>` |br|
|nbsp| |nbsp| |nbsp| |nbsp| :ref:`[WHERE expression] <sql_select>` |br|
|nbsp| |nbsp| |nbsp| |nbsp| :ref:`[GROUP BY expression [, expression ...]] <sql_group_by>` |br|
|nbsp| |nbsp| |nbsp| |nbsp| :ref:`[HAVING expression] <sql_having>` |br|
|nbsp| |nbsp| |nbsp| |nbsp| :ref:`[ORDER BY expression] <sql_order_by>` |br|
|nbsp| |nbsp| |nbsp| |nbsp| :ref:`LIMIT expression [OFFSET expression]]; <sql_limit>` |br|
|nbsp| :ref:`SET SESSION session-name = session-value; <sql_set>` |br|
|nbsp| :ref:`START TRANSACTION; <sql_start_transaction>` |br|
|nbsp| :ref:`TRUNCATE TABLE table-name; <sql_truncate>` |br|
|nbsp| :ref:`UPDATE table-name <sql_update>` |br|
|nbsp| |nbsp| |nbsp| |nbsp| :ref:`SET column-name=expression [,column-name=expression...] <sql_update>` |br|
|nbsp| |nbsp| |nbsp| |nbsp| :ref:`[WHERE expression]; <sql_update>` |br|
|nbsp| :ref:`VALUES (expression [, expression ...]; <sql_values>` |br|
|nbsp| :ref:`WITH [RECURSIVE] common-table-expression; <sql_with>`

.. _sql_data_type_conversion:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Data Type Conversion
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Data type conversion, also called casting, is necessary for any operation involving two operands X and Y,
when X and Y have different data types. |br|
Or, casting is necessary for assignment operations
(when INSERT or UPDATE is putting a value of type X into a column defined as type Y). |br|
Casting can be "explicit" when a user uses the :ref:`CAST <sql_function_cast>` function, or "implicit" when Tarantool does a conversion automatically.

The general rules are fairly simple: |br|
Assignments and operations involving NULL cause NULL or UNKNOWN results. |br|
For arithmetic, convert to the data type which can contain both operands and the result. |br|
For explicit casts, if a meaningful result is possible, the operation is allowed. |br|
For implicit casts, if a meaningful result is possible and the data types on both sides
are either STRINGs or numbers (that is, are STRING or INTEGER or UNSIGNED or DOUBLE or NUMBER),
the operation is sometimes allowed.

The specific situations in this chart follow the general rules:

.. code-block:: none

    ~                To BOOLEAN | To number  | To STRING | To VARBINARY
    ---------------  ----------   ----------   ---------   ------------
    From BOOLEAN   | AAA        | S--        | A--       | ---
    From number    | A--        | SSA        | A-A       | ---
    From STRING    | S--        | S-S        | AAA       | A--
    From VARBINARY | ---        | ---        | A--       | AAA

Where each entry in the chart has 3 characters: |br|
Where A = Always allowed, S = Sometimes allowed, - = Never allowed. |br|
The first character of an entry is for explicit casts, |br|
the second character is for implicit casts for assignment, |br|
the third character is for implicit cast for comparison. |br|
So AAA = Always for explicit, Always for Implicit (assignment), Always for Implicit (comparison).

The S "Sometimes allowed" character applies for these special situations: |br|
From STRING To BOOLEAN is allowed if UPPER(string-value) = ``'TRUE'`` or ``'FALSE'``. |br|
From number to INTEGER or UNSIGNED is allowed for cast and assignment only if the result is not out of range,
and the number has no post-decimal digits. |br|
From STRING to INTEGER or UNSIGNED is allowed only if the string has a representation of a number,
and the result is not out of range,
and the number has no post-decimal digits. |br|
From STRING to DOUBLE or NUMBER is allowed only if the string has a representation of a number. |br|
From BOOLEAN to number is allowed only if the number is not DOUBLE.
The chart does not show To|From SCALAR because the conversions depend on the type of the value,
not the type of the column definition.
Explicit cast to SCALAR is allowed but has no effect, the result data type is always the same as the original data type.
But comparisons of values of different types are allowed if the definition is SCALAR.

..  note::

    Since version :doc:`2.4.1 </release/2.4.1>`, the NUMBER type is processed
    in the same way as the :ref:`number <index-box_number>` type in
    NoSQL Tarantool.

Examples of casts, illustrating the situations in the chart:

``CAST(TRUE AS INTEGER)`` is legal because the intersection of the  "From BOOLEAN" row with the "To number"
column is ``A--`` and the first letter of ``A--`` is for explicit cast and A means Always Allowed.
The result is 1.

``UPDATE ... SET varbinary_column = 'A'`` is illegal because the intersection of the "From STRING" row with the "To VARBINARY"
column is ``A--`` and the second letter of ``A--`` is for implicit cast (assignment) and - means not allowed.
The result is an error message.

``1.7E-1 > 0`` is legal because the intersection of the "From number" row with the "To number"
column is SSA, and the third letter of SSA is for implicit cast (comparison) and A means Always Allowed.
The result is TRUE.

``11 > '2'`` is legal because the intersection of the "From number" row with the "To STRING"
column is AAA and the third letter of AAA is for implicit cast (comparison) and A means Always Allowed.
The result is TRUE.  For detailed explanation see the following section.

``CAST('5' AS INTEGER)`` is legal because the intersection of the "From STRING" row with the "To number"
column is S-S and the first letter of S-S is for explicit cast and S means Sometimes Allowed.
However, ``CAST('5.5' AS INTEGER)`` is illegal because 5.5 is not an integer --
if the string contains post-decimal digits and the target is INTEGER or UNSIGNED,
the assignment will fail.

.. _sql-implicit_cast:

********************************************************************************
Implicit string/numeric cast
********************************************************************************

Special considerations may apply for casting STRINGs
to/from INTEGERs/DOUBLEs/NUMBERs/UNSIGNEDs (numbers) for comparison or assignment.

``1 = '1' /* compare a STRING with a number */`` |br|
``UPDATE ... SET string_column = 1 /* assign a number to a STRING */``

For comparisons, the cast is always from STRING to number. |br|
Therefore ``1e2 = '100'`` is TRUE, and ``11 > '2'`` is TRUE. |br|
If the cast fails, then the number is less than the STRING. |br|
Therefore ``1e400 < ''`` is TRUE. |br|
Exception: for BETWEEN the cast is to the data type of the first and last operands. |br|
Therefore ``'66' BETWEEN 5 AND '7'`` is TRUE.

For assignments, due to a change in behavior starting with Tarantool
:doc:`2.5.1 </release/2.5.1>`,
implicit casts from strings to numbers are not legal. Therefore
``INSERT INTO t (integer_column) VALUES ('5');`` is an error.

Implicit cast does happen if STRINGS are used in arithmetic. |br|
Therefore ``'5' / '5' = 1``. If the cast fails, then the result is an error. |br|
Therefore ``'5' / ''`` is an error.

Implicit cast does NOT happen if numbers are used in concatenation, or in LIKE. |br|
Therefore ``5 || '5'`` is illegal.

In the following examples, implicit cast does not happen for values in SCALAR columns: |br|
``DROP TABLE scalars;`` |br|
``CREATE TABLE scalars (scalar_column SCALAR PRIMARY KEY);`` |br|
``INSERT INTO scalars VALUES (11), ('2');`` |br|
``SELECT * FROM scalars WHERE scalar_column > 11;   /* 0 rows. So 11 > '2'. */`` |br|
``SELECT * FROM scalars WHERE scalar_column < '2';  /* 1 row. So 11 < '2'. */`` |br|
``SELECT max(scalar_column) FROM scalars; /* 1 row: '2'. So 11 < '2'. */`` |br|
``SELECT sum(scalar_column) FROM scalars; /* 1 row: 13. So cast happened. */`` |br|
These results are not affected by indexing, or by reversing the operands.

Implicit cast does NOT happen for :ref:`GREATEST() <sql_function_greatest>`
or :ref:`LEAST() <sql_function_least>`.
Therefore ``LEAST('5',6)`` is 6.

For function arguments: |br|
If the function description says that a parameter has a specific data type,
and implicit assignment casts are allowed, then arguments which are not passed with that
data type will be converted before the function is applied. |br|
For example, the :ref:`LENGTH() <sql_function_length>` function expects a
STRING or VARBINARY,
and INTEGER  can be converted to STRING, therefore LENGTH(15) will return
the length of ``'15'``, that is, 2. |br|
But implicit cast sometimes does NOT happen for parameters.
Therefore ``ABS('5')`` will cause an error message after
`Issue#4159 <https://github.com/tarantool/tarantool/issues/4159>`_ is fixed.
However, :ref:`TRIM(5) <sql_function_trim>` will still be legal.

Although it is not a requirement of the SQL standard, implicit cast is supposed to help compatibility
with other DBMSs. However, other DBMSs have different rules about what can be converted
(for example they may allow assignment of ``'inf'`` but disallow comparison with ``'1e5'``).
And, of course, it is not possible to be compatible with other DBMSs and at the same
time support SCALAR, which other DBMSs do not have.

Limitations (`Issue#3809 <https://github.com/tarantool/tarantool/issues/3809>`_): |br|
Result of concatenation, or out-of-bound result, may have wrong type. |br|
Parameter conversion behavior will change (`Issue#4159 <https://github.com/tarantool/tarantool/issues/4159>`_). After issue#4159 is done, LENGTH(15) will be illegal.
