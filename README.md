# Origin Selectable extensions

For generating more dynamic queries with Mongoid :)

# Usage

`gem 'origin-selectable_ext'`

Or include in `.gemspec` file as dependency ;)

## Example usage

From the `money-mongoid` gem (also used in [Timespan](https://github.com/kristianmandrup/timespan))

```ruby
class Money
  class << self  
    def custom_serialization?(operator)
      return false unless operator
      case operator
        when "$gte", "$gt", "$lt", "$lte"
          true
      else
        false
      end
    end

    def custom_specify(name, operator, value, options = {})
      money = value.__evolve_to_money__
      case operator
        when "$gte", "$gt", "$lt", "$lte"
          specify_with_multiple_currencies(name, operator, money, options)
      else
        raise RuntimeError, "Unsupported operator"
      end
    end

    def custom_between(field, value, options = {})
      { "$gte" => ::Money.new(value.min, value.iso_code), "$lte" => ::Money.new(value.max, value.iso_code) }
    end

    def custom_between? field, value
      true
    end

    private

    def specify_with_multiple_currencies(name, operator, value, options)
      currencies = [value.currency.iso_code]
      currencies.concat options[:compare_using] if options && options[:compare_using]
      multiple_money_values = value.exchange_to_currencies(currencies.to_set)
      subconditions = multiple_money_values.collect {|money| specify_with_single_currency(name, operator, money)}
      if subconditions.count > 0
        {"$or" => subconditions}
      else
        subconditions[0]
      end
    end

    def specify_with_single_currency(name, operator, value)
      {"#{name}.cents" => {operator => value.cents}, "#{name}.currency_iso" => value.currency.iso_code}
    end
  end
end
```

See https://github.com/mongoid/origin/issues/47 for more details.

## Contributing to origin-selectable_ext
 
* Check out the latest master to make sure the feature hasn't been implemented or the bug hasn't been fixed yet.
* Check out the issue tracker to make sure someone already hasn't requested it and/or contributed it.
* Fork the project.
* Start a feature/bugfix branch.
* Commit and push until you are happy with your contribution.
* Make sure to add tests for it. This is important so I don't break it in a future version unintentionally.
* Please try not to mess with the Rakefile, version, or history. If you want to have your own version, or is otherwise necessary, that is fine, but please isolate to its own commit so I can cherry-pick around it.

## Copyright

Copyright (c) 2012 Kristian Mandrup. See LICENSE.txt for
further details.

