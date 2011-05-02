# Mapping Keywords to/from Entity Names
Entity names in SQL may be specified as strings or as keywords. It's convenient to represent records as Clojure maps with keywords for the keys but this means that a mapping is required when moving from Clojure to SQL and back again. Historically, clojure.contrib.sql simply called (name) on keywords passed in and used clojure.core/resultset-seq to convert Java ResultSet objects back to Clojure maps, which had the side-effect of lowercasing all entity names as they became keywords. Whilst that is still the default behavior of clojure.java.jdbc, it is now possible to override this behavior in a couple of ways.
## Quoted Identifiers
The first problem that the old approach exposed was when table names or column names were the same as a SQL keyword. Databases provide a way to quote identifier names so that entity names do not get treated as SQL keywords (often referred to as 'stropping'). Unfortunately, the way quoting works tends to be vendor-specific so that Microsoft SQL Server accepts \[name\] and "name" whilst MySQL traditionally uses `name` (although recent versions also accept "name"). In order to support multiple approaches to quoted, clojure.java.jdbc now has a function *as-quoted-identifier* and a macro *with-quoted-identifiers* to allow you to specify how quoting should be handled.

A quote spec is either a single character or a pair of characters in a vector. For the former, the entity name is wrapped in a pair of that character. For the latter, the entity name is wrapped in the specified pair:

```clj
(as-quoted-identifier \\\` :name) ;; produces "\`name\`"
(as-quoted-identifier \[\[ \]\] :name) ;; produces "\[name\]"
```
Any code can be wrapped in the *with-quoted-identifiers* macro to influence how keywords are mapped to entity names:

```clj
(sql/with-quoted-identifiers \\\`
  (sql/insert-record
    :fruit
    { :name "Pear" :appearance "green" :cost 99}))
;; INSERT INTO \`fruit\` ( \`name\`, \`appearance\`, \`cost\` )
;; VALUES ( 'Pear', 'green', 99 )
```