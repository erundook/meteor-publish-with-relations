# Publish with relations

Publish with relations builds on Tom's [gist](https://gist.github.com/tmeasday/4042603) 
to provide a convenient way to publish associated records.

## Installation

Publish with relations can be installed with [Meteorite](https://github.com/oortcloud/meteorite/).
From inside a Meteorite-managed app:

``` sh
$ mrt add publish-with-relations
```

## API

### How to use it
Let's say you want to render a page that requires data from three collections: Posts, Comments, and Users.

Post JSON
```javascript
"post" : {
  "_id" : [objectId],
  "authorId" : [objectId]
}
```
Comment JSON
```javascript
"comment" : {
  "_id" : [objectId],
  "userId" : [objectId],
  "postId" : [postId],
  "approved" : [Boolean]
}
```
User JSON
```javascript
"user" : {
  "_id" : [objectId]
}
```

The publish-with-relations package will allow you to publish all three under a single publication/subscription. 

For example, the publication below will return the post (specified by the id parameter), along with the user profile of the auther and 10 approved comments with their author profiles as well.

On the client
```javascript
if( Meteor.isClient() ){
  Meteor.subscribe('post', postId);
}
```
On the server
```javascript
if(Meteor.isServer()){
  Meteor.publish('post', function(id) {
    Meteor.publishWithRelations({
      handle: this,
      collection: Posts,
      filter: id,
      mappings: [{
        key: 'authorId',
        collection: Meteor.users
      }, {
        reverse: true,
        key: 'postId',
        collection: Comments,
        filter: { approved: true },
        options: {
          limit: 10,
          sort: { createdAt: -1 }
        },
        mappings: [{
          key: 'userId',
          collection: Meteor.users
        }]
      }]
    });
  });
}
```
What you'll notice is that you can actually nest mappings/relationships within one another. So for example, a post has many comments, and each comment has an author. 

### Properties of Meteor.publishWithRelations
```collection``` (Collection) 
Add the name of the collection

```key``` (String)
This is the foreign key that relates two collections. For example, a Post has a User through ```authorId```, so the key would be ```authorId```.

```reverse``` (Boolean, [false])
Reverse tells the relationship of the key between the collections. Reverse should be set to true if the foreign key is a property of the mapped Collection. For example, ```postId``` is a property of Comments, which is mapped to Posts. Therefore, reverse should be true.

```filter``` (Object)
The selectors for a Meteor mongo query

```options``` (Object)
Options on the Meteor mongo query

### Changelog
#### v0.1.4
- Foreign key can be an array now. Thanks [Tom](http://github.com/tmeasday)!

### Compatibility notes
Use <= v0.1.1 for meteor version < 0.5.7

