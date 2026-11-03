# LikeC4 quick reference

Everything this skill needs to write a valid model. Checked with LikeC4 1.59.4.
Files end in `.c4`. All `.c4` files in a folder form one model, so a model can be split
across several files.

## The three blocks

- `specification { }`: the kinds of elements you use, and their default style.
- `model { }`: the elements, nested to show containment, and the relationships.
- `views { }`: which diagrams to draw.

## Elements

```
id = kind 'Title' {
  description 'One sentence'
  technology 'PostgreSQL'      // optional
  metadata {
    status 'draft'             // on subsystems; humans change it to 'approved'
    source 'docs/orders.md'    // where the docs say this
    code 'services/orders'     // folder with its code (checked by archtrace)
    service 'orders-api'       // docker-compose / Kubernetes service that runs it
  }
}
```

- Ids: camelCase letters and digits, starting with a letter (`ordersDb`, not `orders-db`).
  Avoid LikeC4 keywords as ids, such as `model`, `views`, `element`, `style`.
- Nest an element inside another to show it is part of it. Refer to nested elements with
  dots from outside: `shop.orders.checkout`.
- Strings use single quotes. Escape a quote inside with a backslash: `'Customer\'s cart'`.

## Relationships (allowed dependencies)

```
web -> orders 'places orders through'           // inside the same parent: short ids
shop.orders.checkout -> payments 'charges cards through'   // from outside: full ids
```

Every arrow is an allowed dependency. Anything not drawn is reported as a violation by
`archtrace check`. Label each arrow with a short verb phrase.

## Views

```
view index {                     // the top-level picture
  title 'Overview'
  include *
}
view shopContainers of shop {    // what is inside one element
  title 'Shop: containers'
  include *
}
dynamic view placeOrder {        // one flow, step by step, in order
  title 'Placing an order'
  customer -> shop.web 'submits the cart'
  shop.web -> shop.orders 'creates the order'
}
```

`include *` shows the element's direct children and what they connect to.

LikeC4 accepts dynamic-view steps that have no matching arrow in the model. For
archtrace, though, a flow step with no matching arrow describes a dependency that isn't
allowed. When the docs describe the step, also add the arrow to the model (a reply going
back the other way doesn't need one).

## Useful shapes

Declare kinds with a shape in `specification`, then use them like any kind:

```
element database {
  style {
    shape storage
  }
}
element queue {
  style {
    shape queue
  }
}
```

## Validate and view

```
npx likec4@1.59.4 validate <model folder>   # lists errors with file and line
npx likec4@1.59.4 start <model folder>      # opens the diagrams in a browser
```

## A complete example

This example passes validation. Copy its structure.

```
specification {
  element person
  element system
  element container
  element component
  element database {
    style {
      shape storage
    }
  }
  element queue {
    style {
      shape queue
    }
  }
}

model {
  customer = person 'Customer' {
    description 'Buys products online'
  }

  shop = system 'Shop' {
    description 'Online shop for customers'
    metadata {
      status 'draft'
      source 'docs/overview.md#shop'
    }

    web = container 'Web app' {
      description 'Pages the customer sees'
      technology 'React'
      metadata {
        code 'src/web'
        source 'docs/overview.md#frontend'
      }
    }

    orders = container 'Orders service' {
      description 'Places and tracks orders'
      metadata {
        code 'services/orders'
        service 'orders-api'
        source 'docs/orders.md'
      }

      checkout = component 'Checkout' {
        description 'Turns a cart into an order'
        metadata {
          code 'services/orders/checkout'
        }
      }
    }

    ordersDb = database 'Orders database' {
      description 'Stores orders'
      technology 'PostgreSQL'
      metadata {
        service 'postgres'
      }
    }

    events = queue 'Order events' {
      description 'Order placed and shipped events'
      technology 'Kafka'
    }

    web -> orders 'places orders through'
    orders -> ordersDb 'stores orders in'
    orders -> events 'publishes order events to'
  }

  payments = system 'Payments provider' {
    description 'Charges cards (external)'
  }

  customer -> shop.web 'shops using'
  shop.orders.checkout -> payments 'charges cards through'
}

views {
  view index {
    title 'Overview'
    include *
  }

  view shopContainers of shop {
    title 'Shop: containers'
    include *
  }

  view ordersComponents of shop.orders {
    title 'Orders service: components'
    include *
  }

  dynamic view placeOrder {
    title 'Placing an order'
    customer -> shop.web 'submits the cart'
    shop.web -> shop.orders 'creates the order'
    shop.orders -> shop.ordersDb 'saves it'
    shop.orders -> shop.events 'publishes OrderPlaced'
  }
}
```
