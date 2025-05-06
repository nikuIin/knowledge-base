https://www.postgresql.org/docs/6.3/c09.htm – cool article!

Table 9-1. Postgres Operators

| Operator | Description                              | Usage                                |
| :------- | :--------------------------------------- | :----------------------------------- |
| <        | Less than?                               | 1 < 2                                |
| <=       | Less than or equal to?                   | 1 <= 2                               |
| <>       | Not equal?                               | 1 <> 2                               |
| =        | Equal?                                   | 1 = 1                                |
| >        | Greater than?                            | 2 > 1                                |
| >=       | Greater than or equal to?                | 2 >= 1                               |
| \|       | Concatenate strings                      | 'Postgre' \| 'SQL'                   |
| !!=      | NOT IN                                   | 3 !!= i                              |
| ~~       | LIKE                                     | 'scrappy,marc,hermit' ~~ '%scrappy%' |
| !~~      | NOT LIKE                                 | 'bruce' !~~ '%al%'                   |
| ~        | Match (regex), case sensitive            | 'thomas' ~ '*.thomas*.'              |
| ~*       | Match (regex), case insensitive          | 'thomas' ~* '*.Thomas*.'             |
| !~       | Does not match (regex), case sensitive   | 'thomas' !~ '*.Thomas*.'             |
| !~*      | Does not match (regex), case insensitive | 'thomas' !~ '*.vadim*.'              |
Table 9-2. Postgres Numerical Operators

| Operator | Description               | Usage     |
| :------- | :------------------------ | :-------- |
| !        | Factorial                 | 3 !       |
| !!       | Factorial (left operator) | !! 3      |
| %        | Modulo                    | 5 % 4     |
| %        | Truncate (doesn't work?)  | % 4.5     |
| *        | Multiplication            | 2 * 3     |
| +        | Addition                  | 2 + 3     |
| -        | Subtraction               | 2 - 3     |
| /        | Division                  | 4 / 2     |
| :        | Natural Exponentiation    | : 3.0     |
| ;        | Natural Logarithm         | (; 5.0)   |
| @        | Absolute value            | @ -5.0    |
| ^        | Exponentiation            | 2.0 ^ 3.0 |
| \|/      | Square root               | \|/ 25.0  |
| \|/      | Cube root                 | \|/ 27.0  |

Table 9-3. Postgres Geometric Operators

| Operator | Description                 | Usage                                                |
| :------- | :-------------------------- | :--------------------------------------------------- |
| +        | Translation                 | '((0,0),(1,1))'::box + '(2.0,0)'::point              |
| -        | Translation                 | '((0,0),(1,1))'::box - '(2.0,0)'::point              |
| *        | Scaling/rotation            | '((0,0),(1,1))'::box * '(2.0,0)'::point              |
| /        | Scaling/rotation            | '((0,0),(2,2))'::box / '(2.0,0)'::point              |
| #        | Intersection                | '((1,-1),(-1,1))' # '((1,1),(-1,-1))'                |
| #        | Number of points in polygon | # '((1,0),(0,1),(-1,0))'                             |
| ##       | Point of closest proximity  | '(0,0)'::point ## '((2,0),(0,2))'::lseg              |
| &&       | Overlaps?                   | '((0,0),(1,1))'::box && '((0,0),(2,2))'::box         |
| &<       | Overlaps to left?           | '((0,0),(1,1))'::box &< '((0,0),(2,2))'::box         |
| &>       | Overlaps to right?          | '((0,0),(3,3))'::box &> '((0,0),(2,2))'::box         |
| <->      | Distance between            | '((0,0),1)'::circle <-> '((5,0),1)'::circle          |
| <<       | Left of?                    | '((0,0),1)'::circle << '((5,0),1)'::circle           |
| <^       | Is below?                   | '((0,0),1)'::circle <^ '((0,5),1)'::circle           |
| >>       | Is right of?                | '((5,0),1)'::circle >> '((0,0),1)'::circle           |
| >^       | Is above?                   | '((0,5),1)'::circle >^ '((0,0),1)'::circle           |
| ?#       | Intersects or overlaps      | '((-1,0),(1,0))'::lseg ?# '((-2,-2),(2,2))'::box;    |
| ?-       | Is horizontal?              | '(1,0)'::point ?- '(0,0)'::point                     |
| ?-\|     | Is perpendicular?           | '((0,0),(0,1))'::lseg ?-\| '((0,0),(1,0))'::lseg     |
| @-@      | Length or circumference     | @-@ '((0,0),(1,0))'::path                            |
| ?\|      | Is vertical?                | '(0,1)'::point ?\| '(0,0)'::point                    |
| ?\|      | Is parallel?                | '((-1,0),(1,0))'::lseg ?\| '((-1,2),(1,2))'::lseg    |
| @        | Contained or on             | '(1,1)'::point @ '((0,0),2)'::circle                 |
| @@       | Center of                   | @@ '((0,0),10)'::circle                              |
| ~=       | Same as                     | '((0,0),(1,1))'::polygon ~= '((1,1),(0,0))'::polygon |

The time interval data type tinterval is a legacy from the original date/time types and is not as well supported as the more modern types. There are several operators for this type.

Table 9-4. Postgres Time Interval Operators

| Operator | Description                        | Usage |
| :------- | :--------------------------------- | :---- |
| #<       | Interval less than?                |       |
| #<=      | Interval less than or equal to?    |       |
| #<>      | Interval not equal?                |       |
| #=       | Interval equal?                    |       |
| #>       | Interval greater than?             |       |
| #>=      | Interval greater than or equal to? |       |
| <#>      | Convert to time interval           |       |
| <<       | Interval less than?                |       |
| \|       | Start of interval                  |       |
| ~=       | Same as                            |       |
| <?>      | Time inside interval?              |       |

Users may invoke operators using the operator name, as in