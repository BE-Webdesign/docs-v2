---
title: JavaScript Client
---
The REST API includes a JavaScript/Backbone client library.

The library provides an interface for the WP REST API by providing Backbone Models and Collections for all endpoints exposed the API Schema.

## Using

Activate the WP-API plugin. Enqueue the script directly:

```php
wp_enqueue_script( 'wp-api' );
```

or as a dependency for your script:

```php
wp_enqueue_script( 'my_script', 'path/to/my/script', array( 'wp-api' ) );
```

The library parses the root endpoint (the 'Schema') and creates matching Backbone models and collections. You will now have two root objects available to you: `wp.api.models` and `wp.api.collections`.

The models and collections include:

```
Models:
 * Category
 * Comment
 * Media
 * Page
 * PageMeta
 * PageRevision
 * Posts
 * PostMeta
 * PostRevision
 * Schema
 * Status
 * Tag
 * Taxonomy
 * Type
 * User

Collections:
 * Categories
 * Comments
 * Media
 * PageMeta
 * PageRevisions
 * Pages
 * Posts
 * Statuses
 * Tags
 * Taxonomies
 * Types
 * Users
```

You can use these endpoints as-is to read, update, create and delete items using standard Backbone methods (fetch, sync, save & destroy for models, sync for collections). You can also extend these objects to make them your own, and build your views on top of them.

### Default values

Each model and collection includes a reference to its default values, for example:

```
wp.api.models.Posts.defaults
 * author: null
 * comment_status: null
 * content: null
 * date: null
 * date_gmt: null
 * excerpt: null
 * featured_image: null
 * format: null
 * modified: null
 * modified_gmt: null
 * password: null
 * ping_status: null
 * slug: null
 * status: null
 * sticky: null
 * title: null
```

### Available methods

Each model and collection contains a list of methods the corrosponding endpoint supports. For example, models created from `wp.api.models.Posts` have a method array of:

```
["GET", "POST", "PUT", "PATCH", "DELETE"]
```

### Accepted options

Each model and collection contains a list of options the corrosponding endpoint accepts (note that options are passed as the second parameter when creating models or collections), for example:

```
wp.api.collections.Posts.options
 * author
 * context
 * filter
 * order
 * orderby
 * page
 * per_page
 * search
 * status
```
### Model examples:

To create a post and edit its categories, make sure you are logged in, then:

```js
// Create a new post
var post = new wp.api.models.Posts( { title: 'This is a test post' } );
post.save();

// Create a new post
var post = new wp.api.models.Posts({ title:'new test' } );
post.save();

// Get a collection of the post's categories
var postCategories = post.getCategories();

// Get a collection of the post's categories (returns a promise)
// Uses _embedded data if available, in which case promise resolves immediately.
post.getCategories().done( function( postCategories ) {
	// ... do something with the categories.
	// The new post has an single Category: Uncategorized
	postCategories.at( 0 ).get( 'name' );
	// response -> "Uncategorized"
} );

// Get a posts author User model.
post.getAuthorUser().done( function( user ){
	// ... do something with user
} );

// Get a posts featured image Media model.
post.getFeaturedImage().done( function( image ){
	// ... do something with image
} );

// Set the post categories.
post.setCategories( [ 'apples', 'oranges' ] );

// Get all the categories
var allCategories = new wp.api.collections.Categories()
allCategories.fetch();

var appleCategory = allCategories.findWhere( { slug: 'apples' } );

// Add the category to the postCategories collection we previously fetched.
appleCategory.set( 'parent_post', post.get( 'id' ) );

// Use the POST method so Backbone will not PUT it even though it has an id.
postCategories.create( appleCategory.toJSON(), { type: 'POST' } );

// Remove the Uncategorized category
postCategories.at( 0 ).destroy();

// Check the results - re-fetch
postCategories = post.getCategories();

postCategories.at( 0 ).get( 'name' );
// response -> "apples"
```

### Collection examples:

to get the last 10 posts:

```js
var postsCollection = new wp.api.collections.Posts();
postsCollection.fetch();
```

to get the last 25 posts:

```js
postsCollection.fetch( { data: { per_page: 25 } } );
```

use filter to change the order & orderby options:

```js
postsCollection.fetch( { data: { 'filter': { 'orderby': 'title', 'order': 'ASC' } } } );
```

All collections support pagination automatically, and you can get the next page of results using `more`:

```js
postsCollection.more();
```

If you add custom endpoints to the api they will also become available as models/collections. For example, you will get new models and collections when you [add REST API support to your custom post type](http://v2.wp-api.org/extending/custom-content-types/). Note: because the schema is stored in the user's session cache to avoid re-fetchingyou may need to open a new tab to get a new read of the Schema.
