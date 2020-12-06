---
title: Advent of Code 2020 - Day 1
date: 2020-12-05 21:01:12
tags: aoc2020, elixir
---
Challenges in Day 1 are essentially the ["Two Sum" problem](https://leetcode.com/problems/two-sum/).

## Solution to part 1
```elixir
defmodule Report do
  @input_file_path Path.expand('./input.txt', __DIR__)

  def report(report_in_list, target_sum) do
    report_in_map = Map.new(report_in_list, &{&1, &1})

    report_in_list
    |> Enum.reduce(-1, calculate(report_in_map, target_sum))
  end

  def calculate(map, target_sum) do
    fn x, result ->
      target = Map.get(map, target_sum - x)

      cond do
        result > -1 -> result
        target -> target * x
        true -> result
      end
    end
  end

  def load_report do
    File.read!(@input_file_path)
    |> String.split("\n", trim: true)
    |> Enum.map(&String.to_integer/1)
  end
```
## Solution to part 2
```elixir
  def report_three(report_in_list, target_sum) do
    report_in_list
    |> Enum.reduce(-1, fn x, result ->
      two_sum = report(report_in_list, target_sum - x)

      cond do
        result > -1 -> result
        two_sum > -1 -> two_sum * x
        true -> result
      end
    end)
  end
  ```
### Key takeaway
- A list can be converted into a map using `Map.new/2` and return a two-element tuple ({key, val}) in the callback function
- Use `cond do ...` instead of `if-elseif-else`
- `String.split` may return empty strings, add `trim: true` (default is `false`) to avoid that

Full solution can be found [here](https://github.com/ycswaves/aoc-2020/blob/main/lib/day1)
