# WorkingHours

[![Build Status](https://travis-ci.org/Intrepidd/working_hours.svg?branch=master)](https://travis-ci.org/Intrepidd/working_hours)

A modern ruby gem allowing to do time calculation with working hours.

Compatible and tested with:
- Ruby `2.0`, `2.1`
- ActiveSupport `3.2.x`, `4.0.x` and `4.1.x`

## Installation

Gemfile:

```ruby
gem 'working_hours'
```

## Usage

```ruby
require 'working_hours'

# Move forward
1.working.day.from_now
2.working.hours.from_now
15.working.minutes.from_now

# Move backward
1.working.day.ago
2.working.hours.ago
15.working.minutes.ago

# Start from custom Date or Time
Date.new(2014, 12, 31) + 8.working.days # => Mon, 12 Jan 2015
Time.utc(2014, 8, 4, 8, 32) - 4.working.hours # => 2014-08-01 13:00:00

# Compute working days between two dates
friday.working_days_until(monday) # => 1
# Time is considered at end of day, so:
# - friday to saturday = 0 working days
# - sunday to monday = 1 working days

# Compute working duration (in seconds) between two times
from = Time.utc(2014, 8, 3, 8, 32) # sunday 8:32am
to = Time.utc(2014, 8, 4, 10, 32) # monday 10:32am
from.working_time_until(to) # => 5520 (1.hour + 32.minutes)

# Know if a day is worked
Date.new(2014, 12, 28).working_day? # => false

# Know if a time is worked
Time.utc(2014, 8, 4, 7, 16).in_working_hours? # => false
```

## Configuration

The working hours configuration is thread local and consists of a hash defining working periods for each day, a time zone and a list of days off.

```ruby
# Configure working hours
WorkingHours::Config.working_hours = {
  :tue => {'09:00' => '12:00', '13:00' => '17:00'},
  :wed => {'09:00' => '12:00', '13:00' => '17:00'},
  :thu => {'09:00' => '12:00', '13:00' => '17:00'},
  :fri => {'09:00' => '12:00', '13:00' => '17:00'},
  :sat => {'10:00' => '15:00'}
}

# Configure timezone (uses activesupport, defaults to UTC)
WorkingHours::Config.time_zone = 'Paris'

# Configure holidays
WorkingHours::Config.holidays = [Date.new(2014, 12, 31)]
```

> Once the config has been set, internal objects are frozen and you **can't** modify them (ex: `holidays << Date.today`). This is because the configuration is compiled into a calculation friendly form and these changes would not be taken into account. To alter the config you **must use** one of the 3 setters shown above.

## No core extensions / monkey patching

Core extensions (monkey patching to add methods on Time, Date, Numbers, etc.) are handy but not appreciated by everyone. WorkingHours can also be used **without any monkey patching**:

```ruby
require 'working_hours/module'

# Move forward
WorkingHours::Duration.new(1, :days).from_now
WorkingHours::Duration.new(2, :hours).from_now
WorkingHours::Duration.new(15, :minutes).from_now

# Move backward
WorkingHours::Duration.new(1, :days).ago
WorkingHours::Duration.new(2, :hours).ago
WorkingHours::Duration.new(15, :minutes).ago

# Start from custom Date or Time
WorkingHours::Duration.new(8, :days).since(Date.new(2014, 12, 31)) # => Mon, 12 Jan 2015
WorkingHours::Duration.new(4, :hours).until(Time.utc(2014, 8, 4, 8, 32)) # => 2014-08-01 13:00:00

# Compute working days between two dates
WorkingHours.working_days_between(friday, monday) # => 1
# Time is considered at end of day, so:
# - friday to saturday = 0 working days
# - sunday to monday = 1 working days

# Compute working duration (in seconds) between two times
from = Time.utc(2014, 8, 3, 8, 32) # sunday 8:32am
to = Time.utc(2014, 8, 4, 10, 32) # monday 10:32am
WorkingHours.working_time_between(from, to) # => 5520 (1.hour + 32.minutes)

# Know if a day is worked
WorkingHours.working_day?(Date.new(2014, 12, 28)) # => false

# Know if a time is worked
WorkingHours.in_working_hours?(Time.utc(2014, 8, 4, 7, 16)) # => false
```

## Use in your class/module

If you want to use working hours inside a specific class or module, you can include its computation methods like this:

```ruby
require 'working_hours'

class Order
  include WorkingHours::Computation

  def shipping_date_estimate
    order_date + 2.working.days
  end
end
```

> This also works with zero monkey patch by requiring `working_hours/module`

## Timezones

This gem uses a simple but efficient approach in dealing with timezones. When you define your working hours **you have to choose** a timezome associated with it (in the config example, the working hours are in Paris time). Then, any time used in calcultation will be converted to this timezone first, so you don't have to worry if your times are local or UTC as long as they are correct :)

## Alternatives

There is a gem called [business_time](https://github.com/bokmann/business_time) already available to do this kind of computation and it was of great help to us. But we decided to start another one because business_time is suffering from a few [bugs](https://github.com/bokmann/business_time/pull/84) and [inconsistencies](https://github.com/bokmann/business_time/issues/50). It also lacks essential features to us (like working minutes computation).

## Contributing

1. Fork it ( http://github.com/intrepidd/working_hours/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
