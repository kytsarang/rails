## Rails 4.0.0 (unreleased) ##

*   You can now override the generated accessor methods for stored attributes
    and reuse the original behavior with `read_store_attribute` and `write_store_attribute`,
    which are counterparts to `read_attribute` and `write_attribute`.

    *Matt Jones*

*   Accept belongs_to (including polymorphic) association keys in queries

    The following queries are now equivalent:

        Post.where(:author => author)
        Post.where(:author_id => author)

        PriceEstimate.where(:estimate_of => treasure)
        PriceEstimate.where(:estimate_of_type => 'Treasure', :estimate_of_id => treasure)

    *Peter Brown*

*   Use native `mysqldump` command instead of `structure_dump` method
    when dumping the database structure to a sql file. Fixes #5547.

    *kennyj*

*   Attribute predicate methods, such as `article.title?`, will now raise
    `ActiveModel::MissingAttributeError` if the attribute being queried for
    truthiness was not read from the database, instead of just returning false.

    *Ernie Miller*

*   `ActiveRecord::SchemaDumper` uses Ruby 1.9 style hash, which means that the
    schema.rb file will be generated using this new syntax from now on.

    *Konstantin Shabanov*

*   Map interval with precision to string datatype in PostgreSQL. Fixes #7518. *Yves Senn*

*   Fix eagerly loading associations without primary keys. Fixes #4976. *Kelley Reynolds*

*   Rails now raise an exception when you're trying to run a migration that has an invalid
    file name. Only lower case letters, numbers, and '_' are allowed in migration's file name.
    Please see #7419 for more details.

    *Jan Bernacki*

*   Fix bug when calling `store_accessor` multiple times.
    Fixes #7532.

    *Matt Jones*

*   Fix store attributes that show the changes incorrectly.
    Fixes #7532.

    *Matt Jones*

*   Fix `ActiveRecord::Relation#pluck` when columns or tables are reserved words.

    *Ian Lesperance*

*   Allow JSON columns to be created in PostgreSQL and properly encoded/decoded.
    to/from database.

    *Dickson S. Guedes*

*   Fix time column type casting for invalid time string values to correctly return nil.

    *Adam Meehan*

*   Allow to pass Symbol or Proc into :limit option of #accepts_nested_attributes_for.

    *Mikhail Dieterle*

*   ActiveRecord::SessionStore has been extracted from Active Record as `activerecord-session_store`
    gem. Please read the `README.md` file on the gem for the usage. *Prem Sichanugrist*

*   Fix `reset_counters` when there are multiple `belongs_to` association with the
    same foreign key and one of them have a counter cache.
    Fixes #5200.

    *Dave Desrochers*

*   `serialized_attributes` and `_attr_readonly` become class method only. Instance reader methods are deprecated.

    *kennyj*

*   Round usec when comparing timestamp attributes in the dirty tracking.
    Fixes #6975.

    *kennyj*

*   Use inversed parent for first and last child of has_many association.

    *Ravil Bayramgalin*

*   Fix Column.microseconds and Column.fast_string_to_date to avoid converting
    timestamp seconds to a float, since it occasionally results in inaccuracies
    with microsecond-precision times. Fixes #7352.

    *Ari Pollak*

*   Raise `ArgumentError` if list of attributes to change is empty in `update_all`.

    *Roman Shatsov*

*   Fix AR#create to return an unsaved record when AR::RecordInvalid is
    raised. Fixes #3217.

    *Dave Yeu*

*   Fixed table name prefix that is generated in engines for namespaced models.

    *Wojciech Wnętrzak*

*   Make sure `:environment` task is executed before `db:schema:load` or `db:structure:load`.
    Fixes #4772.

    *Seamus Abshere*

*   Allow Relation#merge to take a proc.

    This was requested by DHH to allow creating of one's own custom
    association macros.

    For example:

        module Commentable
          def has_many_comments(extra)
            has_many :comments, -> { where(:foo).merge(extra) }
          end
        end

        class Post < ActiveRecord::Base
          extend Commentable
          has_many_comments -> { where(:bar) }
        end

    *Jon Leighton*

*   Add CollectionProxy#scope.

    This can be used to get a Relation from an association.

    Previously we had a #scoped method, but we're deprecating that for
    AR::Base, so it doesn't make sense to have it here.

    This was requested by DHH, to facilitate code like this:

        Project.scope.order('created_at DESC').page(current_page).tagged_with(@tag).limit(5).scoping do
          @topics      = @project.topics.scope
          @todolists   = @project.todolists.scope
          @attachments = @project.attachments.scope
          @documents   = @project.documents.scope
        end

    *Jon Leighton*

*   Add `Relation#load`.

    This method explicitly loads the records and then returns `self`.

    Rather than deciding between "do I want an array or a relation?",
    most people are actually asking themselves "do I want to eager load
    or lazy load?" Therefore, this method provides a way to explicitly
    eager-load without having to switch from a `Relation` to an array.

    Example:

        @posts = Post.where(published: true).load

    *Jon Leighton*

*   `Model.all` now returns an `ActiveRecord::Relation`, rather than an
    array of records. Use `Relation#to_a` if you really want an array.

    In some specific cases, this may cause breakage when upgrading.
    However in most cases the `ActiveRecord::Relation` will just act as a
    lazy-loaded array and there will be no problems.

    Note that calling `Model.all` with options (e.g.
    `Model.all(conditions: '...')` was already deprecated, but it will
    still return an array in order to make the transition easier.

    `Model.scoped` is deprecated in favour of `Model.all`.

    `Relation#all` still returns an array, but is deprecated (since it
    would serve no purpose if we made it return a `Relation`).

    *Jon Leighton*

*   `:finder_sql` and `:counter_sql` options on collection associations
    are deprecated. Please transition to using scopes.

    *Jon Leighton*

*   `:insert_sql` and `:delete_sql` options on `has_and_belongs_to_many`
    associations are deprecated. Please transition to using `has_many
    :through`.

    *Jon Leighton*

*   The migration generator now creates a join table with (commented) indexes every time
    the migration name contains the word `join_table`:

        rails g migration create_join_table_for_artists_and_musics artist_id:index music_id

    *Aleksey Magusev*

*   Add `add_reference` and `remove_reference` schema statements. Aliases, `add_belongs_to`
    and `remove_belongs_to` are acceptable. References are reversible.
    Examples:

        # Create a user_id column
        add_reference(:products, :user)
        # Create a supplier_id, supplier_type columns and appropriate index
        add_reference(:products, :supplier, polymorphic: true, index: true)
        # Remove polymorphic reference
        remove_reference(:products, :supplier, polymorphic: true)

    *Aleksey Magusev*

*   Add `:default` and `:null` options to `column_exists?`.

        column_exists?(:testings, :taggable_id, :integer, null: false)
        column_exists?(:testings, :taggable_type, :string, default: 'Photo')

    *Aleksey Magusev*

*   `ActiveRecord::Relation#inspect` now makes it clear that you are
    dealing with a `Relation` object rather than an array:.

        User.where(:age => 30).inspect
        # => <ActiveRecord::Relation [#<User ...>, #<User ...>, ...]>

        User.where(:age => 30).to_a.inspect
        # => [#<User ...>, #<User ...>]

    The number of records displayed will be limited to 10.

    *Brian Cardarella, Jon Leighton & Damien Mathieu*

*   Add `collation` and `ctype` support to PostgreSQL. These are available for PostgreSQL 8.4 or later.
    Example:

        development:
          adapter: postgresql
          host: localhost
          database: rails_development
          username: foo
          password: bar
          encoding: UTF8
          collation: ja_JP.UTF8
          ctype: ja_JP.UTF8

    *kennyj*

*   Changed validates_presence_of on an association so that children objects
    do not validate as being present if they are marked for destruction. This
    prevents you from saving the parent successfully and thus putting the parent
    in an invalid state.

    *Nick Monje & Brent Wheeldon*

*   `FinderMethods#exists?` now returns `false` with the `false` argument.

    *Egor Lynko*

*   Added support for specifying the precision of a timestamp in the postgresql
    adapter. So, instead of having to incorrectly specify the precision using the
    `:limit` option, you may use `:precision`, as intended. For example, in a migration:

        def change
          create_table :foobars do |t|
            t.timestamps :precision => 0
          end
        end

    *Tony Schneider*

*   Allow `ActiveRecord::Relation#pluck` to accept multiple columns. Returns an
    array of arrays containing the typecasted values:

        Person.pluck(:id, :name)
        # SELECT people.id, people.name FROM people
        # [[1, 'David'], [2, 'Jeremy'], [3, 'Jose']]

    *Jeroen van Ingen & Carlos Antonio da Silva*

*   Improve the derivation of HABTM join table name to take account of nesting.
    It now takes the table names of the two models, sorts them lexically and
    then joins them, stripping any common prefix from the second table name.

    Some examples:

        Top level models (Category <=> Product)
        Old: categories_products
        New: categories_products

        Top level models with a global table_name_prefix (Category <=> Product)
        Old: site_categories_products
        New: site_categories_products

        Nested models in a module without a table_name_prefix method (Admin::Category <=> Admin::Product)
        Old: categories_products
        New: categories_products

        Nested models in a module with a table_name_prefix method (Admin::Category <=> Admin::Product)
        Old: categories_products
        New: admin_categories_products

        Nested models in a parent model (Catalog::Category <=> Catalog::Product)
        Old: categories_products
        New: catalog_categories_products

        Nested models in different parent models (Catalog::Category <=> Content::Page)
        Old: categories_pages
        New: catalog_categories_content_pages

    *Andrew White*

*   Move HABTM validity checks to `ActiveRecord::Reflection`. One side effect of
    this is to move when the exceptions are raised from the point of declaration
    to when the association is built. This is consistant with other association
    validity checks.

    *Andrew White*

*   Added `stored_attributes` hash which contains the attributes stored using
    `ActiveRecord::Store`. This allows you to retrieve the list of attributes
    you've defined.

       class User < ActiveRecord::Base
         store :settings, accessors: [:color, :homepage]
       end

       User.stored_attributes[:settings] # [:color, :homepage]

    *Joost Baaij & Carlos Antonio da Silva*

*   PostgreSQL default log level is now 'warning', to bypass the noisy notice
    messages. You can change the log level using the `min_messages` option
    available in your config/database.yml.

    *kennyj*

*   Add uuid datatype support to PostgreSQL adapter. *Konstantin Shabanov*

*   Added `ActiveRecord::Migration.check_pending!` that raises an error if
    migrations are pending. *Richard Schneeman*

*   Added `#destroy!` which acts like `#destroy` but will raise an
    `ActiveRecord::RecordNotDestroyed` exception instead of returning `false`.

    *Marc-André Lafortune*

*   Allow blocks for `count` with `ActiveRecord::Relation`, to work similar as
    `Array#count`:

        Person.where("age > 26").count { |person| person.gender == 'female' }

    *Chris Finne & Carlos Antonio da Silva*

*   Added support to `CollectionAssociation#delete` for passing `fixnum`
    or `string` values as record ids. This finds the records responding
    to the `id` and executes delete on them.

        class Person < ActiveRecord::Base
          has_many :pets
        end

        person.pets.delete("1") # => [#<Pet id: 1>]
        person.pets.delete(2, 3) # => [#<Pet id: 2>, #<Pet id: 3>]

    *Francesco Rodriguez*

*   Deprecated most of the 'dynamic finder' methods. All dynamic methods
    except for `find_by_...` and `find_by_...!` are deprecated. Here's
    how you can rewrite the code:

      * `find_all_by_...` can be rewritten using `where(...)`
      * `find_last_by_...` can be rewritten using `where(...).last`
      * `scoped_by_...` can be rewritten using `where(...)`
      * `find_or_initialize_by_...` can be rewritten using
        `where(...).first_or_initialize`
      * `find_or_create_by_...` can be rewritten using
        `where(...).first_or_create`
      * `find_or_create_by_...!` can be rewritten using
        `where(...).first_or_create!`

    The implementation of the deprecated dynamic finders has been moved
    to the `activerecord-deprecated_finders` gem. See below for details.

    *Jon Leighton*

*   Deprecated the old-style hash based finder API. This means that
    methods which previously accepted "finder options" no longer do. For
    example this:

        Post.find(:all, :conditions => { :comments_count => 10 }, :limit => 5)

    Should be rewritten in the new style which has existed since Rails 3:

        Post.where(comments_count: 10).limit(5)

    Note that as an interim step, it is possible to rewrite the above as:

        Post.all.merge(:where => { :comments_count => 10 }, :limit => 5)

    This could save you a lot of work if there is a lot of old-style
    finder usage in your application.

    `Relation#merge` now accepts a hash of
    options, but they must be identical to the names of the equivalent
    finder method. These are mostly identical to the old-style finder
    option names, except in the following cases:

      * `:conditions` becomes `:where`
      * `:include` becomes `:includes`
      * `:extend` becomes `:extending`

    The code to implement the deprecated features has been moved out to
    the `activerecord-deprecated_finders` gem. This gem is a dependency
    of Active Record in Rails 4.0. It will no longer be a dependency
    from Rails 4.1, but if your app relies on the deprecated features
    then you can add it to your own Gemfile. It will be maintained by
    the Rails core team until Rails 5.0 is released.

    *Jon Leighton*

*   It's not possible anymore to destroy a model marked as read only.

    *Johannes Barre*

*   Added ability to ActiveRecord::Relation#from to accept other ActiveRecord::Relation objects

      Record.from(subquery)
      Record.from(subquery, :a)

    *Radoslav Stankov*

*   Added custom coders support for ActiveRecord::Store. Now you can set
    your custom coder like this:

        store :settings, accessors: [ :color, :homepage ], coder: JSON

    *Andrey Voronkov*

*   `mysql` and `mysql2` connections will set `SQL_MODE=STRICT_ALL_TABLES` by
    default to avoid silent data loss. This can be disabled by specifying
    `strict: false` in your `database.yml`.

    *Michael Pearson*

*   Added default order to `first` to assure consistent results among
    diferent database engines. Introduced `take` as a replacement to
    the old behavior of `first`.

    *Marcelo Silveira*

*   Added an :index option to automatically create indexes for references
    and belongs_to statements in migrations.

    The `references` and `belongs_to` methods now support an `index`
    option that receives either a boolean value or an options hash
    that is identical to options available to the add_index method:

      create_table :messages do |t|
        t.references :person, :index => true
      end

      Is the same as:

      create_table :messages do |t|
        t.references :person
      end
      add_index :messages, :person_id

    Generators have also been updated to use the new syntax.

    [Joshua Wood]

*   Added bang methods for mutating `ActiveRecord::Relation` objects.
    For example, while `foo.where(:bar)` will return a new object
    leaving `foo` unchanged, `foo.where!(:bar)` will mutate the foo
    object

    *Jon Leighton*

*   Added `#find_by` and `#find_by!` to mirror the functionality
    provided by dynamic finders in a way that allows dynamic input more
    easily:

        Post.find_by name: 'Spartacus', rating: 4
        Post.find_by "published_at < ?", 2.weeks.ago
        Post.find_by! name: 'Spartacus'

    *Jon Leighton*

*   Added ActiveRecord::Base#slice to return a hash of the given methods with
    their names as keys and returned values as values.

    *Guillermo Iguaran*

*   Deprecate eager-evaluated scopes.

    Don't use this:

        scope :red, where(color: 'red')
        default_scope where(color: 'red')

    Use this:

        scope :red, -> { where(color: 'red') }
        default_scope { where(color: 'red') }

    The former has numerous issues. It is a common newbie gotcha to do
    the following:

        scope :recent, where(published_at: Time.now - 2.weeks)

    Or a more subtle variant:

        scope :recent, -> { where(published_at: Time.now - 2.weeks) }
        scope :recent_red, recent.where(color: 'red')

    Eager scopes are also very complex to implement within Active
    Record, and there are still bugs. For example, the following does
    not do what you expect:

        scope :remove_conditions, except(:where)
        where(...).remove_conditions # => still has conditions

    *Jon Leighton*

*   Remove IdentityMap

    IdentityMap has never graduated to be an "enabled-by-default" feature, due
    to some inconsistencies with associations, as described in this commit:

       https://github.com/rails/rails/commit/302c912bf6bcd0fa200d964ec2dc4a44abe328a6

    Hence the removal from the codebase, until such issues are fixed.

    *Carlos Antonio da Silva*

*   Added the schema cache dump feature.

    `Schema cache dump` feature was implemetend. This feature can dump/load internal state of `SchemaCache` instance
    because we want to boot rails more quickly when we have many models.

    Usage notes:

      1) execute rake task.
      RAILS_ENV=production bundle exec rake db:schema:cache:dump
      => generate db/schema_cache.dump

      2) add config.active_record.use_schema_cache_dump = true in config/production.rb. BTW, true is default.

      3) boot rails.
      RAILS_ENV=production bundle exec rails server
      => use db/schema_cache.dump

      4) If you remove clear dumped cache, execute rake task.
      RAILS_ENV=production bundle exec rake db:schema:cache:clear
      => remove db/schema_cache.dump

    *kennyj*

*   Added support for partial indices to PostgreSQL adapter

    The `add_index` method now supports a `where` option that receives a
    string with the partial index criteria.

        add_index(:accounts, :code, :where => "active")

        Generates

        CREATE INDEX index_accounts_on_code ON accounts(code) WHERE active

    *Marcelo Silveira*

*   Implemented ActiveRecord::Relation#none method

    The `none` method returns a chainable relation with zero records
    (an instance of the NullRelation class).

    Any subsequent condition chained to the returned relation will continue
    generating an empty relation and will not fire any query to the database.

    *Juanjo Bazán*

*   Added the `ActiveRecord::NullRelation` class implementing the null
    object pattern for the Relation class. *Juanjo Bazán*

*   Added new `:dependent => :restrict_with_error` option. This will add
    an error to the model, rather than raising an exception.

    The `:restrict` option is renamed to `:restrict_with_exception` to
    make this distinction explicit.

    *Manoj Kumar & Jon Leighton*

*   Added `create_join_table` migration helper to create HABTM join tables

        create_join_table :products, :categories
        # =>
        # create_table :categories_products, :id => false do |td|
        #   td.integer :product_id, :null => false
        #   td.integer :category_id, :null => false
        # end

    *Rafael Mendonça França*

*   The primary key is always initialized in the @attributes hash to nil (unless
    another value has been specified).

*   In previous releases, the following would generate a single query with
    an `OUTER JOIN comments`, rather than two separate queries:

        Post.includes(:comments)
            .where("comments.name = 'foo'")

    This behaviour relies on matching SQL string, which is an inherently
    flawed idea unless we write an SQL parser, which we do not wish to
    do.

    Therefore, it is now deprecated.

    To avoid deprecation warnings and for future compatibility, you must
    explicitly state which tables you reference, when using SQL snippets:

        Post.includes(:comments)
            .where("comments.name = 'foo'")
            .references(:comments)

    Note that you do not need to explicitly specify references in the
    following cases, as they can be automatically inferred:

        Post.where(comments: { name: 'foo' })
        Post.where('comments.name' => 'foo')
        Post.order('comments.name')

    You also do not need to worry about this unless you are doing eager
    loading. Basically, don't worry unless you see a deprecation warning
    or (in future releases) an SQL error due to a missing JOIN.

    [Jon Leighton]

*   Support for the `schema_info` table has been dropped.  Please
    switch to `schema_migrations`.

*   Connections *must* be closed at the end of a thread.  If not, your
    connection pool can fill and an exception will be raised.

*   Added the `ActiveRecord::Model` module which can be included in a
    class as an alternative to inheriting from `ActiveRecord::Base`:

        class Post
          include ActiveRecord::Model
        end

    Please note:

      * Up until now it has been safe to assume that all AR models are
        descendants of `ActiveRecord::Base`. This is no longer a safe
        assumption, but it may transpire that there are areas of the
        code which still make this assumption. So there may be
        'teething difficulties' with this feature. (But please do try it
        and report bugs.)

      * Plugins & libraries etc that add methods to `ActiveRecord::Base`
        will not be compatible with `ActiveRecord::Model`. Those libraries
        should add to `ActiveRecord::Model` instead (which is included in
        `Base`), or better still, avoid monkey-patching AR and instead
        provide a module that users can include where they need it.

      * To minimise the risk of conflicts with other code, it is
        advisable to include `ActiveRecord::Model` early in your class
        definition.

    *Jon Leighton*

*   PostgreSQL hstore records can be created.

*   PostgreSQL hstore types are automatically deserialized from the database.

Please check [3-2-stable](https://github.com/rails/rails/blob/3-2-stable/activerecord/CHANGELOG.md) for previous changes.
