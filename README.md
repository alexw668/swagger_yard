# SwaggerYard #

The SwaggerYard gem is a Rails Engine designed to parse your Yardocs API controller.
It'll create a Swagger-UI complaint JSON to be served out through where you mount SwaggerYard.
You can mount this to the Rails app serving the REST API or you could mount it as a separate Rails app.
Parsing of the Yardocs happens during the server startup and the data will be subsequently cached to the Rails cache you have defined.
If your API is large expect a slow server start up time.

## Installation ##

Put SwaggerYard in your Gemfile:

    gem 'swagger_yard'

Install the gem with Bunder:

    bundle install


## Getting Started ##

### Place your configuration in a your rails initializers ###

    # config/initializers/swagger_yard.rb
    SwaggerYard.configure do |config|
      config.swagger_version = "1.2"
      config.api_version = "1.0"
      config.reload = Rails.env.development?

      # where your swagger spec json will show up
      config.swagger_spec_base_path = "http://localhost:3000/swagger/api"
      # where your actual api is hosted from
      config.api_base_path = "http://localhost:3000/api"
    end

### Mount your engine ###

	# config/routes.rb
	mount SwaggerYard::Engine, at: "/swagger"

## Example Documentation ##

### Here is an example of how to use SwaggerYard in your Controller ###

**Note:** Model references should be Capitalized or CamelCased, basic types (integer, boolean, string, etc) should be lowercased everywhere.

```ruby
# @resource Account ownership
#
# @resource_path /accounts/ownerships
#
# This document describes the API for creating, reading, and deleting account ownerships.
#
class Accounts::OwnershipsController < ActionController::Base
  ##
  # Returns a list of ownerships associated with the account.
  #
  # @notes Status can be -1(Deleted), 0(Inactive), 1(Active), 2(Expired) and 3(Cancelled).
  #
  # @path [GET] /accounts/ownerships.{format_type}
  #
  # @parameter offset   [integer]               Used for pagination of response data (default: 25 items per response). Specifies the offset of the next block of data to receive.
  # @parameter status   [array<string>]                 Filter by status. (e.g. status[]=1&status[]=2&status[]=3).
  # @parameter_list     [String]    sort_order        Orders response by fields. (e.g. sort_order=created_at).
  #                     [List]      id
  #                     [List]      begin_at
  #                     [List]      end_at
  #                     [List]      created_at
  # @parameter sort_descending    [boolean]     Reverse order of sort_order sorting, make it descending.
  # @parameter begin_at_greater   [date]        Filters response to include only items with begin_at >= specified timestamp (e.g. begin_at_greater=2012-02-15T02:06:56Z).
  # @parameter begin_at_less      [date]        Filters response to include only items with begin_at <= specified timestamp (e.g. begin_at_less=2012-02-15T02:06:56Z).
  # @parameter end_at_greater     [date]        Filters response to include only items with end_at >= specified timestamp (e.g. end_at_greater=2012-02-15T02:06:56Z).
  # @parameter end_at_less        [date]        Filters response to include only items with end_at <= specified timestamp (e.g. end_at_less=2012-02-15T02:06:56Z).
  #
  def index
    ...
  end

  ##
  # Returns an ownership for an account by id
  # 
  # @path [GET] /accounts/ownerships/{id}.{format_type}
  # @response_type [Ownership]
  # @error_message [EmptyOwnership] 404 Ownership not found
  # @error_message 400 Invalid ID supplied
  #
  def show
    ...
  end
end
```

### Here is an example of how to use SwaggerYard in your Model ###

```ruby
#
# @model Pet
#
# @property id(required)    [integer]   the identifier for the pet
# @property name  [Array<string>]    the names for the pet
# @property age   [integer]   the age of the pet
# @property relatives(required) [Array<Pet>] other Pets in its family
#
class Pet
end
```

To then use your `Model` in your `Controller` documentation, add `@parameter`s:

```ruby
# @parameter [Pet] pet The pet object
```

## Authorization ##

Currently, SwaggerYard only supports API Key auth descriptions. Start by adding `@authorization` to your `ApplicationController`.

```ruby
#
# @authorization [api_key] header X-APPLICATION-API-KEY
#
class ApplicationController < ActionController::Base
end
```

Then you can use these authorizations from your controller or actions in a controller. The name comes from either header or query plus the name of the key downcased/underscored.

```ruby
#
# @authorize_with header_x_application_api_key
#
class PetController < ApplicationController
end
```

![Web UI](https://raw.github.com/tpitale/swagger_yard/master/example/web-ui.png)

## Seeing the Swagger JSON ##

You should now be able to start your application and visit `http://local-dev.domain:port/swagger/doc` (if the engine is mounted at `/swagger` as above). You should see the output list each of the controllers you documented, and the paths to see their specific APIs.

To see a specific controller visit `http://local-dev.domain:port/swagger/api/accounts/ownerships` as given in the `@resource_path` in the example above.

## Generators ##

There are two generators that you can use, if you need to customize the UI/JS (optional)

     rails g swagger_yard:ui
     rails g swagger_yard:js

They both copy over their respective files over to your Rails app to be customized.
See [rails engines overriding views](http://guides.rubyonrails.org/engines.html#overriding-views) for more info

Copying over JS requires that ActionDispatch::Static middleware be used (by default it should in use).

## Notes ##

By default SwaggerYard will use a slightly modify version of the swagger-ui. Changes to the JS code are indicated with "SwaggerYard changes" comments. The changes are mainly to support Rails way of supporting an array of parameters.

## More Information ##

* [Swagger-ui](https://github.com/wordnik/swagger-ui)
* [Yard](https://github.com/lsegal/yard)
* [Swagger-spec version 1.2](https://github.com/wordnik/swagger-spec/blob/master/versions/1.2.md)
