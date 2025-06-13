# Functions

+ Functions are evaluated during compilation
* Functions are evaluated on the Server
* Commonly used for:
  * Interfacing with external tools
  * Expand additional functionality to the Puppet DSL
* Example functions:
  * include
  * epp
  * create_resources
  * fail
* Built-in or extend by a module

Two types of funfions:

* **:statement**
  * Perform an action like output or log information
  * This is the default
* **:rvalue** returns a result
  * Assigned to a variable
  * Assigned to a resource parameter
  * Used to chose a conditional branch

## Iterative functions for manipulating data structures

* Some built-in **:rvalue** lambda functions
  * map - transform values to some data structure
  * filter - remove non-matching elements
  * reduce - combine values to a new data structure
  * slice - repeat a code block a given number of times

### Function map

Applies a lambda to every value in a data structure and returns an array containing the results.

* For the array `$data`, return an array containing each value multiplied by 10

  ```puppet
  $data = [1,2,3]
  $transformed_data = $data.map |$items| { $items * 10 }
  ```

  ```puppet
  $data = [1,2,3]
  $transformed_data = $data.map |$index, $value| { $value }
  ```

* For the hash `$data`, return an array containing the values

  ```puppet
  $data = {'a'=>1,'b'=>2,'c'=>3}
  $transformed_data = $data.map |$items| { $items[1] }
  ```

  * `$item[0]` corresponds to the keys of `$data`
  * `$item[1]` equals to the values

  ```puppet
  $data = {'a'=>1,'b'=>2,'c'=>3}
  $transformed_data = $data.map |$key, $value| { $value }
  ```

### Function filter

Applies to every value and returns an array or hash containing any elements for which evaluates to a truthy value (not false or undef).

* For the array `$data`, return an array containing the values that end with "berry"

  ```puppet
  $data = ["orange", "blueberry", "raspberry"]
  $filtered_data = $data.filter |$items| { $items =~ /berry$/ }
  ```

* For the array `$data`, return an array of all keys that both end with "berry" and have an even-numbered index

  ```puppet
  $data = ["orange", "blueberry", "raspberry"]
  $filtered_data = $data.filter |$index, $value| { $index % 2 == 0 and $value =~ /berry$/ }
  ```

* For the hash `$data`, return a hash containing all values of keys that end with "berry"

  ```puppet
  $data = { "orange" => 0, "blueberry" => 1, "raspberry" => 2 }
  $filtered_data = $data.filter |$items| { $items[0] =~ /berry$/ }

* For the hash `$data`, return a hash of all keys that both end with "berry" and have values less than or equal to 1

  ```puppet
  $data = { "orange" => 0, "blueberry" => 1, "raspberry" => 2 }
  $filtered_data = $data.filter |$key, $value| { $key =~ /berry$/ and $value <= 1 }
  ```

### Function reduce

Applies to every value in a data structure from the first argument, carrying over the returned value of each iteration, and returns the result. This lets you create a new data structure by combining values from the first argument's data structure.

* Returning the sum of all values in the array `$data`

  ```puppet
  $data = [1, 2, 3]
  $sum = $data.reduce |$memo, $value| { $memo + $value }
  ```

* Same with a start value of 4

  ```puppet
  $data = [1, 2, 3]
  $sum = $data.reduce(4) |$memo, $value| { $memo + $value }
  ```

* Reduce a hash of hashes $data, merging defaults into the inner hashes

  ```puppet
  $data = {
    'connection1' => {
      'username' => 'user1',
      'password' => 'pass1',
    },
    'connection2' => {
      'username' => 'user2',
      'password' => 'pass2',
    },
  }
  
  $defaults = {
    'maxActive' => '20',
    'maxWait'   => '10000',
  }
  
  $merged = $data.reduce({}) |$memo, $item| {
    $memo + { $item[0] => $defaults + $data[$item[0]] }
  }
  ```

  Results to a hash:

  ```puppet
  $merged = {
    'connection1' => {
      'maxActive' => '20',
      'maxWait'   => '10000',
      'username' => 'user1',
      'password' => 'pass1',
    },
    'connection2' => {
      'maxActive' => '20',
      'maxWait'   => '10000',
      'username' => 'user2',
      'password' => 'pass2',
    },
  }
  ```

## Developing Functions

* The code consists of Ruby or pure Puppet code
* The name of the function and the filename must match

```console
$ tree {MODULE NAME}
|-- lib
|   `-- puppet
|       `-- functions
|           `-- {MODULE NAME}
|               `-- myfunc.rb
|-- functions
|   |-- mypuppetfunc.pp
|   |-- sub
|   |   `-- myotherpuppetfunc.pp
```

### Puppet DSL Functions

* Located in a modules subfolder functions
* Always a return value function
* Useable like every builtin or custom function

Syntax:

```puppet
function <MODULE NAME>::<NAME>(<PARAMETER LIST>) >> <RETURN TYPE> {
  ... body of function ...
  final expression, which is the returned value of the function
}
```

Example:

```puppet
function apache::bool2http(Variant[String, Boolean] $arg) >> String {
  case $arg {
    false, undef, /(?i:false)/ : { 'Off' }
    true, /(?i:true)/          : { 'On' }
    default                    : { "$arg" }
  }
}
```

### Custom Functions

* It will always return the output of the last line executed

Syntax:

```ruby
Puppet::Functions.create_function(:<FUNCTION NAME>) do
  dispatch :<METHOD NAME> do
    param '<DATA TYPE>', :<ARGUMENT NAME (displayed in docs/errors)>
    ...
  end

  def <METHOD NAME>(<ARGUMENT NAME (for local use)>, ...)
    <IMPLEMENTATION>
  end
end
```

Example:

```ruby
Puppet::Functions.create_function(:'mymodule::upper') do
  dispatch :up do
    param 'String', :some_string
  end

  def up(some_string)
    some_string.upcase
  end
end
```
