# JSONAPI::Authorization

[![Build Status](https://travis-ci.org/venuu/jsonapi-authorization.svg?branch=master)](https://travis-ci.org/venuu/jsonapi-authorization) [![Gem Version](https://badge.fury.io/rb/jsonapi-authorization.png)](http://badge.fury.io/rb/jsonapi-authorization)

`JSONAPI::Authorization` adds authorization to the [jsonapi-resources][jr] (JR) gem using [Pundit][pundit].

***PLEASE NOTE:*** This gem currently handles only a subset of operations available in JR. This gem is still considered to be ***alpha quality*** and therefore you shouldn't rely on it on production (yet).

  [jr]: https://github.com/cerebris/jsonapi-resources "A resource-focused Rails library for developing JSON API compliant servers."
  [pundit]: https://github.com/elabs/pundit "Minimal authorization through OO design and pure Ruby classes"

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'jsonapi-authorization'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install jsonapi-authorization

## Usage

Make sure you have a Pundit policy specified for every backing model that your JR resources use. Then hook this gem up to your application like so:

```ruby
JSONAPI.configure do |config|
  config.operations_processor = '::JSONAPI::Authorization::Pundit'
end
```

Make all your JR controllers specify the following in the `context`:

```ruby
class BaseResourceController < ActionController::Base
  include JSONAPI::ActsAsResourceController

  private

  def context
    {user: current_user, action: action_name}
  end
end
```

Have your JR resources include the `JSONAPI::Authorization::ResourcePolicyAuthorization` module.

```ruby
class BaseResource < JSONAPI::Resource
  include JSONAPI::Authorization::ResourcePolicyAuthorization
  abstract
end
```

If you want to send a custom response for unauthorized requests, add a `rescue_from` hook to your `BaseResourceController` and whitelist `Pundit::NotAuthorizedError` in your JR configuration.

## Known bugs

There is a bug affecting `jsonapi-resources` error whitelisting, see https://github.com/cerebris/jsonapi-resources/pull/573. To make your whitelisting and `rescue_from` to work properly, here is a potential workaround:

```ruby
JSONAPI.configure do |config|
  config.exception_class_whitelist = [Pundit::NotAuthorizedError]
end
```

```ruby
class BaseResourceController < ActionController::Base
  rescue_from Pundit::NotAuthorizedError, with: :user_not_authorized

  private

  # https://github.com/cerebris/jsonapi-resources/pull/573
  def handle_exceptions(e)
    if JSONAPI.configuration.exception_class_whitelist.any? { |k| e.class.ancestors.include?(k) }
      raise e
    else
      super
    end
  end

  def user_not_authorized
    head :forbidden
  end
end
```

## Configuration

You can use a custom authorizer class by specifying a configure block in an initializer file. If using a custom authorizer class, be sure to require them at the top of the initializer before usage.

```ruby
JSONAPI::Authorization.configure do |config|
  config.authorizer = MyCustomAuthorizer
end
```

## Development

After checking out the repo, run `bundle install` to install dependencies. Then, run `bundle exec rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Credits

Originally based on discussion and code samples by [@barelyknown](https://github.com/barelyknown) and others in [cerebris/jsonapi-resources#16](https://github.com/cerebris/jsonapi-resources/issues/16).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/venuu/jsonapi-authorization.
