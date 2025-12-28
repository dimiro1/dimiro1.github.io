+++
title = 'Quick debugging in Elixir'
date = 2025-10-18
description = 'Using dbg/2 to debug expressions in Elixir pipelines.'
tags = ['Elixir']
+++

I just learned that we can use `dbg/2` (https://hexdocs.pm/elixir/1.19.0/debugging.html#dbg-2) to debug expressions in Elixir. It's quite useful when used with the pipe operator - unlike `IO.inspect`, it automatically shows each step in the pipeline.

```elixir
defmodule MathUtils do
  defp perfect_square?(n) do
    sqrt = :math.sqrt(n)
    sqrt == Float.floor(sqrt)
  end

  def filter_perfect_squares(str) do
    str
    |> String.split(",")
    |> Enum.map(&String.to_integer/1)
    |> Enum.filter(&perfect_square?/1)
    |> Enum.join(",")
    |> dbg() # <- the dbg in the pipeline
  end
end
```

This prints not only the last result, but also each step of the pipeline:
```elixir
MathUtils.filter_perfect_squares("1,2,3,4,5,6,7,8,9")
```

Output:
```elixir
str #=> "1,2,3,4,5,6,7,8,9"
|> String.split(",") #=> ["1", "2", "3", "4", "5", "6", "7", "8", "9"]
|> Enum.map(&String.to_integer/1) #=> [1, 2, 3, 4, 5, 6, 7, 8, 9]
|> Enum.filter(&perfect_square?/1) #=> [1, 4, 9]
|> Enum.join(",") #=> "1,4,9"
```

Result: `"1,4,9"`

---

**Other debugging options:**
- `IO.inspect/2` - prints a value and returns it (good for pipelines, but shows only one value)
- `IO.puts/1` - simple print for strings
- `dbg/2` - shows each pipeline step automatically
