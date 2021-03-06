## `@` (amend)

Syntax: `@[x;y;f;z]`

Where

- `x` is a list (or file symbol, see Tip)
- `y` is an int vector of indexes of `x`
- `f` is a binary function
- `z` is a value in the domain of the second argument of `f`

returns `x` with its values at indexes `y` changed: for `i` in `til count y`, `x[y i]` becomes `f[x[y i];z]` 
```q
q)d:((1 2 3;4 5 6 7);(8 9;10;11 12);(13 14;15 16 17 18;19 20))
q)@[d;1 1 1;+;3]
((1 2 3;4 5 6 7);(17 18;19;20 21);(13 14;15 16 17 18;19 20))
```

!!! tip "Do it on disk"
    Since V3.4 certain vectors can be updated directly on disk without the need to fully rewrite the file. Such vectors must have no attribute, be of a mappable type, not nested, and not compressed. e.g.
    ```q
    q)`:data set til 20;
    q)@[`:data;3 6 8;:;100 200 300]; 
    q)get`:data 
    0 1 2 100 4 5 200 7 300 9 10 11 12 13 14 15 16 17 18 19
    ```

## `count`

Syntax: `count x` (unary, aggregate)  

Returns the number of items in `x` (rows for a table and entries for dictionary). 
Use with `each` to count the number of items at each level of a list or dictionary.
```q
q)count 42  / an atom has 1 item
1
q)count 1 2 3
3
q)RaggedArray:(1 2 3;4 5;6 7 8 9;0)
q)count RaggedArray
4
q)count each RaggedArray
3 2 4 1
q)RaggedDict:`a`b`c!(1 2;3 4 5;"hello")
q)count RaggedDict
3
q)count each RaggedDict
a| 2
b| 3
c| 5
q)\l sp.q
q)count sp
12
```


## `cross`

Syntax: `x cross y`

Returns the cross-product (i.e. all possible combinations) of `x` and `y`.
```q
q)1 2 3 cross 10 20
1 10
1 20
2 10
2 20
3 10
3 20
q)(cross/)(2 3;10;"abc")
2 10 "a"
2 10 "b"
2 10 "c"
3 10 "a"
3 10 "b"
3 10 "c"
```
`cross` can work on tables and dictionaries. 
```q
q)s:`IBM`MSFT`AAPL
q)v:1 2
q)([]s:s)cross([]v:v)
s    v
------
IBM  1
IBM  2
MSFT 1
MSFT 2
AAPL 1
AAPL 2
```


## `cut`

Syntax: `x cut y`

Where 

- `x` is an integer atom, returns `y` splits into a list of items, all (except perhaps the last) of count `x`.
```q
q)4 cut til 10
0 1 2 3
4 5 6 7
8 9
```

- otherwise `cut` is synonymous with `_` _cut_.



## `_` (cut)

Syntax: `x _ y` 

Where `x` is a non-decreasing list of integers, and `y` is a list or table, returns `y` cut at the indexes given in `x`. The result is a list with the same count as `x`.
```q
q)2 4 9 _ til 10           /first result item starts at index 2
2 3
4 5 6 7 8
,9
q)
q)2 4 4 9 _ til 10         /cuts are empty for duplicate indexes
2 3
`long$()
4 5 6 7 8
,9
```
If the number of items to cut is larger than the list specified, the empty list is returned. 
```
q)show each 2 5 7 _ til 12
2 3 4
5 6
7 8 9 10 11
::
::
::
q)\l sp.q
q)count sp
12
q)count 1_sp
11
q)show 10_sp
s  p  qty
---------
s4 p4 300
s1 p5 400
q)show each 2 5 7_sp
s  p  qty
---------
s1 p3 400
s1 p4 200
s4 p5 100
s  p  qty
---------
s1 p6 100
s2 p1 300
s  p  qty
---------
s2 p2 400
s3 p2 200
s4 p2 200
s4 p4 300
s1 p5 400
::
::
::
```


## `_` (drop)

Syntax: `x _ y`

Drops items from a list, entries from a dictionary or columns from a table. Where

- `x` is an int atom and `y` a list or dictionary, returns `y` without the first or last `x` items.
```q
q)5_0 1 2 3 4 5 6 7 8      /drop the first 5 items
5 6 7 8
q)-5_0 1 2 3 4 5 6 7 8     /drop the last 5 items
0 1 2 3
q)1 _ `a`b`c!1 2 3
b| 2
c| 3
```

!!! tip "Drop strings"
    ```q
    q)b:"apple: banana: cherry"
    q)/find the first ":" and remove the prior portion of the sentence.
    q)(b?":") _ b
    ": banana: cherry"
    ```

- `x` is a list or dictionary and `y` is an index or key of `x`, returns `x` without `y`.
```q
q)0 1 2 3 4 5 6 7 8_5      /drop the 5th item
0 1 2 3 4 6 7 8
q)(`a`b`c!1 2 3)_`a        /drop the entry for `a`
b| 2
c| 3
```

- `x` is an atom or vector of keys to dictionary `y`, returns `y` without the entries for `x`.
```q
q)`a _`a`b`c!1 2 3
b| 2
c| 3
q)`a`b _`a`b`c!1 2 3
c| 3
q)(`a`b`c!1 2 3)_`a`b
'type
```

!!! warning "Dropping dictionary entries with integer arguments"
    With dictionaries, distinguish the roles of integer arguments to _drop_.
    ```q
    q)d:100 200!`a`b
    q)1 _ d            /drop the first entry
    200| b
    q)d _ 1            /drop where key=1
    100| a
    200| b
    q)d _ 100          /drop where key=100
    200| b
    q)enlist[1] _ d    /drop where key=1
    100| a
    200| b
    q)enlist[100] _ d  /drop where key=100
    200| b
    q)100 _ d          /drop first 100 entries
    ```

- `x` is a vector of keys and `y` is a table returns `y` without columns `x`.
```q
q)t:([]a:1 2 3;b:4 5 6;c:`d`e`f)
q)`a`b _t
c
-
d
e
f
q)t _`a`b
'type
q)`a _t
'type
q)t _`a
'type
```


## `enlist`

Syntax: `enlist x` 

Returns its argument/s in a list. Where `x` is a dictionary, the result is the corresponding table.

An atom is not a one-item list. `enlist` and `first` will convert between the two.
```q
q)a:10
q)b:enlist a
q)c:enlist b
q)type each (a;b;c)
-6 6 0h
q)a~b
0b
q)a~first b
1b
q)b~c
0b
q)b~first c
1b
```
With multiple arguments returns a single list.
```q
q)show a:enlist[til 5;`ibm`goog;"hello"]
0 1 2 3 4
`ibm`goog
"hello"
q)count a
3
```
Returns a table from a dictionary.
```q
q)enlist `a`b`c!1 3 4
a b c
-----
1 3 4
```


## `^` (fill)

Syntax: `x^y` (atomic) 

Returns `y` with any nulls replaced by the corresponding item of `x`.
```q
q)0^1 2 3 0N
1 2 3 0
q)100^1 2 -5 0N 10 0N
1 2 -5 100 10 100
q)1.0^1.2 -4.5 0n 0n 15
1.2 -4.5 1 1 15
q)`nobody^`tom`dick``harry
`tom`dick`nobody`harry
q)1 2 3 4 5^6 0N 8 9 0N
6 2 8 9 5
```
Integer `x` items are promoted when `y` is float or real.
```q
q)a:11.0 2.1 3.1 0n 4.5 0n
q)type a
9h
q)10^a
11 2.1 3.1 10 4.5 10
q)type 10^a
9h
```
When `x` and `y` are dictionaries, both null and missing values in `y` are filled with those from `x`.
```q
q)(`a`b`c!1 2 3)^`b`c!0N 30
a| 1
b| 2
c| 30
```

<i class="fa fa-hand-o-right"></i> [`^` _coalesce_](joins/#coalesce) where `x` and `y` are keyed tables


## `fills`

Syntax: `fills x` (uniform)

Returns `x` with any nulls replaced by their preceding non-null values, if any.
```q
q)fills 0N 2 3 0N 0N 7 0N
0N 2 3 3 3 7 7
```

!!! Tips
    To back-fill, reverse the list and the result:
    ```q
    q)reverse fills reverse 0N 2 3 0N 0N 7 0N
    2 2 3 7 7 7 0N
    ```
    For a similar function on infinities, first replace them with nulls:
    ```q
    q)fills {(x where x=0W):0N;x} 0N 2 3 0W 0N 7 0W
    0N 2 3 3 3 7 7
    ```


## `flip`

Syntax: `flip x`

Returns `x` transposed, where `x` may be a list of lists, a dictionary or a table. 

In a list of lists, each list must be the same length.
```q
q)flip (1 2 3;4 5 6)
1 4
2 5
3 6
```
The flip of a dictionary is a table, and vice versa. If `x` is a dictionary where the keys are a list of symbols, and the values are lists of the same count (or atoms), then `flip x` will return a table. The flip of a table is a dictionary.
```q
q)D:`sym`price`size!(`IBM`MSFT;10.2 23.45;100 100)
q)flip D
sym  price size
---------------
IBM  10.2  100
MSFT 23.45 100
q)D~flip flip D
1b
```


## `,` (join)

Syntax: `x join y`

Where `x` and `y` are atoms, lists, dictionaries or tables returns `x` joined to `y`. 
```q
q)1 2 3,4
1 2 3 4
q)1 2,3 4
1 2 3 4
q)(0;1 2.5;01b),(`a;"abc")
(0;1.00 2.50;01b;`a;"abc")
```
The result is a vector if both arguments are vectors or atoms of the same type.
```q
q)1 2.4 5,-7.9 10               /float vectors
1.00 2.40 5.00 -7.90 10.00
q)1 2.4 5,-7.9                  /float vector and atomatom 
1.00 2.40 5.00 -7.90
q)1 2.4 5, -7.9 10e             /float and real vectors
(1.00;2.40;5.00;-7.90e;10.00e)
```
Cast arguments to ensure vector results.
```q
q)v:1 2.34 -567.1 20e
q)v,(type v)$789                / cast an int to a real
1.00 2.34 -567.1 20.00 789e
q)v,(type v)$1b                 / cast a boolean to a real
1.00 2.34 -567.1 20 1e
q)v,(type v)$0xab
1.00 2.34 -567.1 20.00 171e
```
Tables can be joined row-wise. 
```q
q)t:([]a:1 2 3;b:`a`b`c)
q)s:([]a:10 11;b:`d`e)
q)show t,s
a  b
----
1  a
2  b
3  c
10 d
11 e
```
Tables of the same count can be joined column-wise with `,'`.
```q
q)r:([]c:10 20 30;d:1.2 3.4 5.6)
q)show t,'r
q)show t,'r
a b c  d
----------
1 a 10 1.2
2 b 20 3.4
3 c 30 5.6
```
_Join_ for keyed tables is strict; both the key and data columns must match in names and datatypes.

<i class="fa fa-hand-o-right"></i> [Joins](joins), [`.Q.dd`](dotq/#qdd-join-symbols) (join symbols)


## `raze`

Syntax: `raze x` 

Returns the items of `x` joined, collapsing one level of nesting. 

To collapse all levels, use [converge](adverbs/#converge-iterate) i.e. `raze/[x]`.
```q
q)raze (1 2;3 4 5)
1 2 3 4 5
q)b:(1 2;(3 4;5 6);7;8)
q)raze b                 / flatten one level
1
2
3 4
5 6
7
8
q)raze/[b]               / flatten all levels
1 2 3 4 5 6 7 8
q)raze 42                / atom returned as a list
,42
```
Returns the flattened values from a dictionary.
```q
q)d:`q`w`e!(1 2;3 4;5 6)
q)value d
1 2
3 4
5 6
q)raze d
1 2 3 4 5 6
```

!!! warning "Use only on items that can be joined"
    `raze` is defined in k as `,/` and requires items that can be joined together. 
    ```q
    q)d:`a`b!(1 2;3 5)
    q)10,d          / cannot join integer and dictionary
    'type
    q)raze (10;d)   / raze will not work
    'type
    ```


## `reverse`

Syntax: `reverse x` (uniform) 

Returns the items of `x` in reverse order.
```q
q)reverse 1 2 3 4
4 3 2 1
```
On atoms, returns the atom; on dictionaries, reverses the keys; and on tables, reverses the columns:
```q
q)d:`a`b!(1 2 3;"xyz")
q)reverse d
b| x y z
a| 1 2 3
q)reverse each d
a| 3 2 1
b| z y x
q)reverse flip d
a b
---
3 z
2 y
1 x
```


## `rotate`

Syntax: `x rotate y` (uniform)

Returns list or table `y` rotated by `x` items: to the ‘left’ for positive `x`, to the ‘right’ for negative `x`.
```q
q)2 rotate 2 3 5 7 11    / rotate a list
5 7 11 2 3
q)-2 rotate 2 3 5 7 11
7 11 2 3 5
q)t:([]a:1 2 3;b:"xyz")
q)1 rotate t             / rotate a table
a b
---
2 y
3 z
1 x
```


## `sv`

Syntax: `x sv y` 

Where:

- (**join strings**) `y` is a list of strings, and `x` is a character or string, returns the strings in `y`, separated by `x`. Where `x` is the back tick `` ` ``, the strings are separated by the host line separator  – `\n` on Unix, `\r\n` on Windows.
```q
q)"," sv ("one";"two";"three")    / comma separated
"one,two,three"
q)"\t" sv ("one";"two";"three")   / tab separated
"one\ttwo\tthree"
q)", " sv ("one";"two";"three")   / x may be a string
"one, two, three"
q)"." sv string 192 168 1 23      / form IP address
"192.168.1.23"
q)` sv ("one";"two";"three")      / use host line separator
"one\ntwo\nthree\n"
```

- (**join path components**) `y` is a symbol list of which the first item is a file handle, it returns a file handle where the items of the list are joined, separated by slashes. This is useful when building file paths.
```q
q)` sv `:/home/kdb/q`data`2010.03.22`trade
`:/home/kdb/q/data/2010.03.22/trade
```
If the first item is not a file handle, returns a symbol where the items are joined, separated by `.` (dot). This is useful for building filenames with a given extension:
```q
q)` sv `mywork`dat
`mywork.dat
```
<i class="fa fa-hand-o-right"></i> [`sv` decode](casting/#sv)


## `#` (take)

Syntax: `x # y`

Returns `y` as a list, dictionary or table described or selected by `x`. 

Where `x` is 

- an int atom, and `y` is an atom or list, returns a list of length `x` filled from `y`, starting at the front if `x` is positive and the end if negative.
```q
q)5#0 1 2 3 4 5 6 7 8      /take the first 5 items
0 1 2 3 4
q)-5#0 1 2 3 4 5 6 7 8     /take the last 5 items
4 5 6 7 8
```
If `x>count y`, `y` is treated as circular.
```q
q)5#`Arthur`Steve`Dennis
`Arthur`Steve`Dennis`Arthur`Steve
q)-5#`Arthur`Steve`Dennis
`Steve`Dennis`Arthur`Steve`Dennis
q)3#9
9 9 9
q)2#`a
`a`a
```
If `x` is 0, an empty list is returned.
```q
q)trade:([]time:();sym:();price:();size:())  /columns can hold anything
q)trade
+`time`sym`price`size!(();();();())
q)/idiomatic way to initialise columns to appropriate types
q)trade:([]time:0#0Nt;sym:0#`;price:0#0n;size:0#0N)
q)trade
+`time`sym`price`size!(`time$();`symbol$();`float$();`int$())
```

- an int atom and `y` is a dictionary, `x` entries are returned.
```q
q)d:`a`b`c!1 2 3
q)2#d
a| 1
b| 2
```

- an int atom and `y` is a table, `x` rows are returned.
```q
q)\l sp.q
..
q)5#sp
s  p  qty
---------
s1 p1 300
s1 p2 200
s1 p3 400
s1 p4 200
s4 p5 100
```

- an int vector and `y` is an atom or list, returns a matrix or higher-dimensional array; `count x` gives the number of dimensions. (Since V2.3)
```q
q)2 5#"!"
"!!!!!"
"!!!!!"
q)2 3#til 6
(0 1 2;3 4 5)
```
A 2&times;4 matrix taken from the list `` `Arthur`Steve`Dennis``
```q
q)2 4#`Arthur`Steve`Dennis
(`Arthur`Steve`Dennis`Arthur;`Steve`Dennis`Arthur`Steve)
```
Higher dimensions are not always easy to see.
```q
q)2 3 4#"a"
"aaaa" "aaaa" "aaaa"
"aaaa" "aaaa" "aaaa"
q)show five3d:2 3 4#til 5
0 1 2 3 4 0 1 2 3 4 0 1
2 3 4 0 1 2 3 4 0 1 2 3
q)count each five3d
3 3
q)first five3d
0 1 2 3
4 0 1 2
3 4 0 1
```
A null in `x` will cause that dimension to be maximal.
```q
q)0N 3#til 10
0 1 2
3 4 5
6 7 8
,9
```

!!! note "Changes since V3.3"
    From V3.4, if `x` is a list of length 1, the result has a single dimension. 
    ```q
    q)enlist[2]#til 10
    0 1
    ```
    From V3.4, `x` can have length greater than 2 – but may not contain nulls.
    ```q
    q)(2 2 3#til 5)~((0 1 2;3 4 0);(1 2 3;4 0 1))
    1b
    q)(enlist("";""))~1 2 0#"a"
    1b
    q)all`domain=@[;1 2;{`$x}]each(#)@'(1 0 2;2 3 0N;0N 2 1;-1 2 3)
    1b
    ```
    The effect of nulls in `x` changed in V3.3.
        
    Prior to V3.3:
    ```q
    q)3 0N # til 10
    (0 1 2 3;4 5 6 7;8 9)
    q)(10 0N)#(),10
    10
    q)4 0N#til 9
    0 1 2
    3 4 5
    6 7 8
    ```
    From V3.3:
    ```q
    q)3 0N#til 10
    (0 1 2;3 4 5;6 7 8 9)
    q)2 0N#0#0
    (`long$();`long$())
    q)(10 0N)#(),10
    (`long$();`long$();`long$();`long$();`long$();`long$();`long$();`long$();`long$();,10)
    q)4 0N#til 9
    0 1
    2 3
    4 5
    6 7 8
    ```

- a symbol vector and `y` is a dictionary, returns entries for `x`.
```q
q)d:`a`b`c!1 2 3
q)`a`b#d
a| 1
b| 2
```

- a symbol vector and `y` is a table, returns columns `x`.
```q
q)`p`qty#sp
p  qty
------
p1 300
p2 200
p3 400
p4 200
p5 100
p6 100
p1 300
p2 400
p2 200
p2 200
p4 300
p5 400
```

- a table and `y` is a table, returns matching rows, together with the respective keys. This is similar to retrieving multiple records through the square brackets syntax, except _take_ also returns the keys. 
```q
q)([]s:`s1`s2)#s
s | name  status city  
--| -------------------
s1| smith 20     london
s2| jones 10     paris 
```
<i class="fa fa-hand-o-right"></i> [_Q for Mortals_: Retrieving Multiple Records](http://code.kx.com/q4m3/8_Tables/#845-retrieving-multiple-records)



## `?` (vector conditional)

Syntax: `?[x;y;z]`

Where `x`, `y` and `z` are conforming vectors or atoms, `x` is boolean, and `y` and `z` are of the same type, returns a vector with items of `y` where `x` is `1b`, otherwise of `z`. All three arguments are evaluated.
```q
q)?[1100b;"abcd";"ABCD"]
"abCD"
q)?[1100b;"a";"ABCD"]
"aaCD"
q)?[1100b;"abcd";"X"]
"abXX"
q)?[1b;"abcd";"X"]
"abcd"
q)?[0b;"abcd";"X"]
"X"
q)?[0b;"abcd";"ABCD"]
"ABCD"
```

!!! tip "Not this, not that"
    It can be useful to have more than just a true/false selection, e.g. match1/match2/match3/others mapping to result1/result2/result3/default. This can be achieved with _find_.
    ```q
    q)input:10?`m1`m2`m3`other`yetanother
    q)input
    `yetanother`m1`m3`m2`m3`m2`m3`other`m3`yetanother
    q)`r1`r2`r3`default `m1`m2`m3?input
    `default`r1`r3`r2`r3`r2`r3`default`r3`default
    ```
    This avoids nesting _vector conditional_, and scales better.


## `vs`

Syntax: `x vs y` 

Partition: returns list `y` partitioned according to `x`.

- Where `x` is a char atom or string, and `y` is a **string**, returns a list of strings: `y` cut using `x` as the delimiter.
```q
q)"," vs "one,two,three"
"one"
"two"
"three"
q)", " vs "spring, summer, autumn, winter"
"spring"
"summer"
"autumn"
"winter"
q)"|" vs "red|green||blue"
"red"
"green"
""
"blue"
```

- Where `x` is the empty symbol `` ` ``, and `y` is a **symbol**, returns as a symbol vector `y` split on `` `.` ``.
```q
q)` vs `mywork.dat 
`mywork`dat
```
- Where `x` is the empty symbol `` ` ``, and `y` is a **file handle**, returns as a symbol vector `y` split into directory and  file parts.
```q
q)` vs `:/home/kdb/data/mywork.dat
`:/home/kdb/data`mywork.dat
```
- Where `x` is the empty symbol `` ` ``, and `y` is a **string**, returns as a list of strings `y` partitioned on embedded line terminators into lines. (Recognizes both Unix `\n` and Windows `\r\n` terminators).
```q
q)` vs "abc\ndef\nghi"
"abc"
"def"
"ghi"
q)` vs "abc\r\ndef\r\nghi"
"abc"
"def"
"ghi"
```

