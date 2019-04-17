## Code Style

### Laravel
1. Always define polymorphic relationships here and follow proper laravel style-guide
```php
class Menu extends Model
{
    protected $table = 'menus';

    protected $fillable = ['name', 'display_name', 'description', 'link', 'icon', 'order'];

    public $timestamps = true;

    public function roles(): BelongsToMany
    {
        return $this->belongsToMany(Role::class);
    }
    
    public function comments(): MorphToMany
    {
        return $this->morphToMany(Comment:class);
    }
} 

class Comment extends Model 
{
        protected $table= 'comments';
        protected $fillable = ['id', 'commetable_id', 'commentable_type', 'comment_body', 'created_at', 'deleted_at'];
        
        protected function commentable()
        {
            return $this->morphTo()
        }
}
```


```php
class Role extends \Spatie\Permission\Models\Role
{
    public function menus(): BelongsToMany
    {
        return $this->belongsToMany(Menu::class);
    }
}

```

### Graphql
1. Always use ID! when representing primary keys or foreign keys
2. Denote [] for array/list data in the case of Many objects
3. Use ! only when field is non nullable in database schema and required
4. It's not necessary to define polymorphic relationships but always query the fields commentable_id, commentable_type, and use it for inputs. 
```

type Menu {
    id: ID!
    name: String!
    display_name: String!
    link: String!
    icon: String!
    order: Int!
    comments: [Comment]
}


type Role {
    id: ID!
    name: String!
    menus: [Menu]! @belongsToMany
}

type CreateCommentInput {
    commentable_id: ID!
    commentable_type: String!
    comment_body: String!
}

type Comment {
    id: ID!
    commentable_id: ID!
    commentable_type: String!
    comment_body: String
    created_at: String
    updated_at: String  
}

```

### Vuex ORM
1. Make sure you use camelCase when naming the entity, this is not the same as Laravel snake_case
2. You have to also create a pivot model for vuex, you don't need to do this in graphql/laravel
3. Use this.increment() for primary keys (not for foreign), when you create the model and persis it, it will change this auto generated id with the one from the server
4. Always reference the polymorphic relationship and it's fields.

```javascript
import { Model } from '@vuex-orm/core'
import Menu from '../menu/menu'
import MenuRole from '../menu/menu_role'

export default class Role extends Model {

  static entity = 'role'

  static eagerLoad = ['menus']

  static fields () {
    return {
      id: this.increment(),
      name: this.string(''),
      menus: this.belongsToMany(Menu, MenuRole, 'role_id', 'menu_id'),
      comments: this.morphMany(Comment, 'commentable_id', 'commentable_type')

    }
  }
}

class Comment extends Model {
  static entity = 'comments'

  static fields () {
    return {
      id: this.attr(null),
      comment_body: this.attr(''),
      commentable_id: this.attr(null),
      commentable_type: this.attr(null)
    }
  }
}

```

```javascript
import {Model} from '@vuex-orm/core'

export default class Menu extends Model {
  static entity = 'menus'

  static fields() {
    return {
      id: this.increment(),
      deleted_at: this.attr(null),
      created_at: this.attr(null),
      updated_at: this.attr(null),
      name: this.string(''),
      display_name: this.string(''),
      description: this.string(''),
      link: this.string(''),
      icon: this.string(''),
      order: this.number(''),
    }
  }
}
```

```javascript
import {Model} from '@vuex-orm/core'

export default class MenuRole extends Model {
  static entity = 'menuRole'
  static primaryKey = ['role_id', 'menu_id']

  static fields() {
    return {
      role_id: this.attr(null),
      menu_id: this.attr(null),
    }
  }
}
```

