# Cucumber Rest BDD

[![Build Status](https://travis-ci.org/graze/cucumber-rest-bdd.svg?branch=master)](https://travis-ci.org/graze/cucumber-rest-bdd)
[![image](https://images.microbadger.com/badges/image/graze/cucumber-rest-bdd.svg)](https://microbadger.com/images/graze/cucumber-rest-bdd "Get your own image badge on microbadger.com")
[![image version](https://images.microbadger.com/badges/version/graze/cucumber-rest-bdd.svg)](https://microbadger.com/images/graze/cucumber-rest-bdd "Get your own version badge on microbadger.com")
[![image license](https://images.microbadger.com/badges/license/graze/cucumber-rest-bdd.svg)](https://microbadger.com/images/graze/cucumber-rest-bdd "Get your own license badge on microbadger.com")
[![Gem Version](https://badge.fury.io/rb/cucumber-rest-bdd.svg)](https://badge.fury.io/rb/cucumber-rest-bdd)

A set of Behavioural tests that can be run against a REST API.

![Giphy](https://media3.giphy.com/media/Tv7VPg6Os488g/giphy.gif)

This is based from: [Effective API Testing With Cucumber](https://github.com/gregbeech/website/blob/master/blog/2014/effective-api-testing-with-cucumber.markdown)

A list of [Steps](STEPS.md) shows the comparison between Behavioural and Functional tests provided by this package.

## Usage

You can include this as a gem in your features, or run directly through docker

**Gem:**

```bash
~$ gem install cucumber-rest-bdd
```

**Docker:**

```bash
~$ docker run --rm -v $(pwd):/opt/src -e endpoint=http://server/ graze/cucumber-rest-bdd
```

## Configuration

The following environment variables modify how this will operate:

- `endpoint` - (string) the base url to call for each request
- `data_key` - (string) the root data key (if applicable) (for example: `"data"` if all responses have a `{"data":{}}` field)
- `error_key` - (string) this will ignore the `data_key` when checking for errors
- `field_separator` - (string) the separator used between words by the api
- `field_camel` - (bool [`true`|`false`]) does this endpoint use camelCase for fields (default: `false`)
- `resource_single` - (bool [`true`|`false`]) if each resource should be singularized or not (default: `false`)
- `set_parent_id` - (bool [`true`|`false`]) when creating sub resources, automatically add parent ids

## Examples

- For a full list of steps see: [STEPS](STEPS.md)
- These examples are taken from the test [features](features)

### Retrieve items

```gherkin
Given I am a client
When I request the post "1"
Then the request was successful
And the response has the following attributes:
    | attribute | type    | value |
    | User Id   | numeric | 1     |
    | Id        | numeric | 1     |
    | Title     | string  | sunt aut facere repellat provident occaecati excepturi optio reprehenderit |
    | Body      | string  | quia et suscipit\\nsuscipit recusandae consequuntur expedita et cum\\nreprehenderit molestiae ut ut quas totam\\nnostrum rerum est autem sunt rem eveniet architecto |
```

```gherkin
Given I am a client
When I request the photo "1" for album "1" for user "1"
Then the request was successful
And the response has the attributes:
    | attribute | type   | value                                              |
    | title     | string | accusamus beatae ad facilis cum similique qui sunt |
```

```gherkin
Given I am a client
When I request a list of posts with:
    | User Id | 2 |
Then the request is successful
And the response is a list of at least 2 posts
And one response has the following attributes:
    | attribute | type    | value |
    | User Id   | numeric | 2     |
    | Id        | numeric | 11    |
    | Title     | string  | et ea vero quia laudantium autem |
    | Body      | string  | delectus reiciendis molestiae occaecati non minima eveniet qui voluptatibus\\naccusamus in eum beatae sit\\nvel qui neque voluptates ut commodi qui incidunt\nut animi commodi |
```

You can inspect child objects by using `:` in between the names

```gherkin
Given I am a client
When I request the comment "1" with:
    | `_expand` | post |
Then the response has the following attributes:
    | attribute    | type   | value |
    | name         | string | id labore ex et quam laborum |
    | email        | string | Eliseo@gardner.biz |
    | body         | string | laudantium enim quasi est quidem magnam voluptate ipsam eos\\ntempora quo necessitatibus\\ndolor quam autem quasi\\nreiciendis et nam sapiente accusantium |
    | post : title | string | sunt aut facere repellat provident occaecati excepturi optio reprehenderit |
    | post : body  | string | quia et suscipit\\nsuscipit recusandae consequuntur expedita et cum\\nreprehenderit molestiae ut ut quas totam\\nnostrum rerum est autem sunt rem eveniet architecto |
```

Alternatively you can inspect child arrays and objects by describing the path of the object with attributes

```gherkin
Given I am a client
When I set JSON request body to:
    """
    {"title":"test","body":"multiple",
    "comments":[
        {"common":1,"id":1,"title":"fish","body":"cake","image":{"href":"some_url"}},
        {"common":1,"id":2,"title":"foo","body":"bar","image":{"href":"some_url"}}
    ]}
    """
And I send a POST request to "http://test-server/posts"
Then the response has the attributes:
    | attribute | type   | value    |
    | title     | string | test     |
    | body      | string | multiple |
And the response has a list of comments
And the response has a list of 2 comments
And the response has two comments with attributes:
    | attribute | type    | value |
    | common    | integer | 1     |
And the response has two comments with an image with attributes:
    | attribute | type    | value    |
    | href      | string  | some_url |
And the response has one comment with attributes:
    | attribute | type    | value |
    | Id        | integer | 1     |
    | Title     | string  | fish  |
    | Body      | string  | cake  |
And the response has one comment with attributes:
    | attribute | type    | value |
    | Id        | integer | 2     |
    | Title     | string  | foo   |
    | Body      | string  | bar   |
```

Each numeric request can be prefixed with a modifier to modify the number specified

```gherkin
Given I am a client
When I request a list of posts with:
    | `_embed` | comments |
Then the response is a list of posts
Then the response is a list of more than 5 posts
Then the response is a list of at least 10 posts
Then more than three posts have the attributes:
    | attribute | type    | value |
    | User Id   | integer | 5     |
Then less than 200 posts have more than four comments
Then more than 50 posts have less than six comments
Then more than 80 posts have a list of comments
Then at least 90 posts have a list of five comments
Then more than 10 posts have five comments
Then less than 200 posts have five comments
```

#### Errors

If the `error_key` environment variable is set, if that key is used as the initial step it will ignore any `data_key`
setting.

Example: `error_key=error`, `data_key=data`

```gherkin
Then the response has a list of posts       | {"data":[{"id": 12}]}

Then the response has one error             | {"errors":[{"message": "Error"}]}

Then the response has an error              | {"error": {"message": "Error}}
```

Example: `error_key=`, `data_key=data`

```gherkin
Then the response has a list of posts       | {"data":[{"id": 12}]}

Then the response has one error             | {"data": {"errors":[{"message": "Error"}]}}

Then the response has an error              | {"data": {"error": {"message": "Error}}}
```

### Creation

```gherkin
Given I am a client
When I request to create a post with:
    | attribute | type    | value |
    | Title     | string  | foo   |
    | Body      | string  | bar   |
    | User Id   | numeric | 1     |
Then the request is successful and a post was created
And the response has the following attributes:
    | attribute | type    | value |
    | User Id   | numeric | 1     |
    | Title     | string  | foo   |
    | Body      | string  | bar   |
```

If the environment variable: `set_parent_id` is set to `true` then when you create sub resources it will add the level aboves id into the json, otherwise it will rely on the api to do it for you

```gherkin
Given I am a client
When I request to create a photo in album "2" for user "1" with:
    | attribute | type   | value |
    | title     | string | foo   |
Then the comment was created
And the response has the attributes:
    | attribute | type   | value |
    | Album Id  | int    | 2     |
    | Title     | string | foo   |
When I request a list of photos for album "2" for user "1"
Then the request was successful
And one comment has the attributes:
    | attribute | type   | value |
    | Title     | string | foo   |
```

### Removal

```gherkin
Given I am a client
When I request to remove the post "20"
Then the request is successful
```

### Modification

```gherkin
Given I am a client
When I request to modify the post "21" with:
    | attribute | type    | value |
    | Title     | string  | foo   |
Then the request is successful
And the response has the following attributes:
    | attribute | type    | value |
    | User Id   | numeric | 3     |
    | Title     | string  | foo   |
    | Body      | string  | repellat aliquid praesentium dolorem quo\\nsed totam minus non itaque\\nnihil labore molestiae sunt dolor eveniet hic recusandae veniam\\ntempora et tenetur expedita sunt |
```

### Multiple Requests

```gherkin
Given I am a client
When I request to create a post with:
    | attribute | type    | value |
    | Title     | string  | foo   |
    | Body      | string  | bar   |
    | User Id   | integer | 12    |
Then the request is successful
When I save "id"
And I request the post "{id}"
Then the request is successful
And the response has the following attributes:
    | attribute | type    | value |
    | Title     | string  | foo   |
    | Body      | string  | bar   |
    | User Id   | numeric | 12    |
    | Id        | numeric | {id}  |
```

## Resources

A resource "name" is attempted to be retrieved from the given name of the item to be retrieved. This pluralises, ensures everything is lower case, removes any unparameterisable characters and uses a `-` separator.

```text
Token -> tokens
User -> users
Big Life -> big-lives
octopus -> octopi
```

If the environment variable: `resource_single` is set to `true` then it will not attempt to pluralise the resources.

```text
Token -> token
User -> user
```

You can directly pass what you want using:

```text
`field`
```

this will not modify the field.

## Attributes

### Types

Attribute types:
The following types are supported:

| type    | other names           | example   |
|---------|-----------------------|-----------|
| integer | numeric, number, long | 12        |
| float   | double, decimal       | 4.8       |
| string  | text                  | "text"    |
| array   | array                 | ["a"]     |
| object  | object                | {"a":"b"} |
| null    | nil                   |           |
| bool    | boolean               | true      |

### Name conversion

attributes are converted into singular parametrised versions of the provided name:

The conversion is based on the provided environment variables `field_camel` and `field_separator`

#### Default

```text
field_camel=false
field_separator=_

Someid -> someid
Product Id -> product_id
Bodies -> body
```

#### CamelCase

```text
field_camel=true
field_separator=_

Someid -> someid
Product Id -> productId
Bodies -> body
```

#### Other Separators

```text
field_camel=false
field_separator=-

Someid -> someid
Product Id -> product-id
Bodies -> body
```
