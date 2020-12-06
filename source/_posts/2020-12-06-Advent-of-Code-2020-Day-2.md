---
title: Advent of Code 2020 - Day 2
date: 2020-12-06 21:14:07
tags: aoc2020, elixir
---

## Part 1
Three different ways to solve part 1

### Method 1
Remove all the required letter in the password and compare with the original password and count how many letters are missing.

This number is essentially the number of occurences of the required letters in the password.
```elixir

defmodule Password do
  @input_file_path Path.expand('./input.txt', __DIR__)

  def validate_method1(entry) do
    [range, required_letter, password] = tokenize_rule(entry)

    password_without_required = String.replace(password, required_letter, "")
    required_letter_count = String.length(password) - String.length(password_without_required)

    [min, max] = parse_range(range)
    min <= required_letter_count and required_letter_count <= max
  end

  defp parse_range(range) do
    range
    |> String.split("-")
    |> Enum.map(&String.to_integer/1)
  end

  defp tokenize_rule(entry) do
    [range, required_letter, password] = String.split(entry, " ")

    # convert 'a:' to 'a'
    required_letter = String.first(required_letter)
    [range, required_letter, password]
  end

  def load_report do
    File.read!(@input_file_path)
    |> String.split("\n", trim: true)
  end
end
```
### Method 2
Split the password string into a list of letters and use [`Enum.frequencies/1`](https://hexdocs.pm/elixir/Enum.html#frequencies/1) to get the number of occurences of the required letter.

One edge case is the the required letter may not appear at all. In this case, the pattern-match
on the result map will throw an error and therefore requires a `try-catch` block.
```elixir
defmodule Password do
  # ...
  def validate_method2(entry) do
    [range, required_letter, password] = tokenize_rule(entry)

    try do
      %{^required_letter => count} =
        password
        |> String.graphemes()
        |> Enum.frequencies()

      [min, max] = parse_range(range)
      min <= count and max >= count
    rescue
      _ ->
        false
    end
  end

  # ...
end
```
### Method 3
Using regex pattern like `/^[^#{required_letter}]*#{required_letter}{#{range}}[^#{required_letter}]*$/`.
For exmaple, to validate this password entry `7-9 q: qqqqlqmqqq`
1. sort the password, `qqqqlqmqqq` -> `lmqqqqqqqq`
2. construct the regex, `/^[^q]*q{7,9}[^q]*/$`, we need to make sure the sorted password starts and ends with letters that are different from the required letter (if any) to avoid the false-positive case where required letter excceeds the upper limit
```elixir
defmodule Password do
  # ...
  def validate_method3(entry) do
    [range, required_letter, password] = tokenize_rule(entry)

    range = String.replace(range, "-", ",")

    validation_regex =
      Regex.compile!("^[^#{required_letter}]*#{required_letter}{#{range}}[^#{required_letter}]*$")

    sorted_password =
      password
      |> String.graphemes()
      |> Enum.sort()
      |> Enum.join()

    String.match?(sorted_password, validation_regex)
  end
  # ...
end
```
## Part 2
The new validation rule is simpler to validate against.
```elixir
defmodule Password do
  # ...
  def validate_by_new_rule(entry) do
    [range, required_letter, password] = tokenize_rule(entry)

    [pos1, pos2] = parse_range(range)
    is_at_pos1 = String.at(password, pos1 - 1) == required_letter
    is_at_pos2 = String.at(password, pos2 - 1) == required_letter

    is_at_pos1 != is_at_pos2
  end
  # ...
end
```

Full solution can be found [here](https://github.com/ycswaves/aoc-2020/blob/main/lib/day2)
