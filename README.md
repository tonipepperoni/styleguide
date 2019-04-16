## Code Style

### Laravel

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
```

type Menu {
    id: ID!
    name: String!
    display_name: String!
    link: String!
    icon: String!
    order: Int! 
}


type Role {
    id: ID!
    name: String!
    menus: [Menu]! @belongsToMany
}

```

### Vuex ORM
1. Make sure you use camelCase when naming the entity, this is not the same as Laravel snake_case
2. You have to also create a pivot model for vuex, you don't need to do this in graphql/laravel

```javascript
import { Model } from '@vuex-orm/core'
import Menu from '../menu/menu'
import MenuRole from '../menu/menu_role'

export default class Role extends Model {

  static entity = 'role'

  static eagerLoad = ['menus']

  static fields () {
    return {
      id: this.attr(null),
      name: this.string(''),
      menus: this.belongsToMany(Menu, MenuRole, 'role_id', 'menu_id')
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
      id: this.attr(null),
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

