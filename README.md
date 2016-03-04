# BOM
Binary Object Message specification

BOM is a binary object message format specification like ON, MsgPack

# BOM specification
  * [type system](#types)
    * [primary types](#primary)
    * [complicated types](#complicated)
    * [special values](#special)
  * [format](#format)
  * [schema](#schema)

<a name="types"/>
## type system

<a name="primary"/>
### primary types
  * **Integer** represents a signed integer
  * **UInteger** represents an unsigned integer
  * **float** represents a single-precision float point number
  * **double** represents a double-precision float point number
  * **string** represents a UTF-8 string
  
<a name="complicated"/>
### complicated types
  * **object** represents an associated-array, a.k.a. key-value pair array
  * **array** represents a single-typed value sequence
  * **tuple** represents a multiple-typed value sequence
  
<a name="special"/>
### special values
  * **null** represents a void value, a.k.a. nil, when it's used as value place. and as type, it represents any type, like Any in Scala, Object in Java.
  * **magic** represents a magic value, used to handle schema, reference, headers, and so on.
  
<a name="format"/>
## format

### overview

<table>
 <tr><th>bits format</th><th>hex</th><th>description</th><th>data representation</th></tr>
 <tr><td>0xxx xxxx</td><td>00-7F</td><td>small integer range 0..127, compatible with singed and unsigned byte integer</td><td>just one byte</td></tr>
 <tr><td>1000 xxxx</td><td>80-8F</td><td>type tags</td><td>denpends</td></tr>
 <tr><td>1001 xxxx</td><td>9_X</td><td>object(n), n for number of pairs</td><td>[9N][N pairs]</td></tr>
 <tr><td>1010 xxxx</td><td>A_X</td><td>array(T) follow number of values</td><td>[AT][N][N values]</td></tr>
 <tr><td>1011 xxxx</td><td>B_X</td><td>string(n), 0..15</td><td>[x_N][N bytes]</td></tr>
 <tr><td>110x xxxx</td><td>C0-DF</td><td>string(n), n for length of UTF bytes, 16..47</td><td>[x_N][N bytes]</td></tr>
 <tr><td>111x xxxx</td><td>E0-FF</td><td>negative byte integer, range -1..-32</td><td>just one byte</td></tr>
</table>

### type tags

<table>
 <tr><th>xxxx format</th><th>hex</th><th>description</th><th>data representation</th></tr>
 <tr><td>0000-0011</td><td>0-3</td><td>singed integers, byte 1, 2, 3, 4</td><td>[_X][byte 1 2 4 8]</td></tr>
 <tr><td>0100-0111</td><td>4-7</td><td>unsigned integers, ubyte 1, 2, 3, 4</td><td>[_X][byte 1 2 4 8]</td></tr>
 <tr><td>1000</td><td>8</td><td>float</td><td>[_8][4 bytes]</td></tr>
 <tr><td>1001</td><td>9</td><td>double</td><td>[_9][8 bytes]</td></tr>
 <tr><td>1010</td><td>A</td><td>string</td><td>[N][N bytes]</td></tr>
 <tr><td>1011</td><td>B</td><td>array, T[], 1 or multi Dimensions</td><td>1D:[T][+N][N values]<br/>multi-D: [T][-i][D0..Di][E(Di) values]</td></tr>
 <tr><td>1100</td><td>C</td><td>tuple<T0, T1...Tn></td><td>[N][T0..Tn][N values]</td></tr>
 <tr><td>1101</td><td>D</td><td>object, array of key-value pair</td><td>[N][N pairs]</td></tr>
 <tr><td>1110</td><td>E</td><td>null</td><td>just single byte</td></tr>
 <tr><td>1111</td><td>F</td><td>magic</td><td>extensions</td></tr>
</table>

### length representation
there are 2 kind of length reprententations
#### full bytes length 
the length start with a full byte, follow variable-length bytes used
<table>
 <tr><th>format</th><th>bits</th><th> max</th></tr>
 <tr><td>0xxx xxxx</td><td>7</td><td>`(2^7)-1`</td></tr>
 <tr><td>100x xxxx byte</td><td>5+8=13</td><td>`(2^13)-1`</td></tr>
 <tr><td>101x xxxx byte byte</td><td>5+16=21</td><td>`(2^21)-1`</td></tr>
 <tr><td>11xx xxxx byte byte byte</td><td>6+24=30</td><td>`(2^30)-1`</td></tr>
</table>

#### partial bytes length
the length start with a partial byte
<table>
 <tr><th>format</th><th>bits</th><th> range</th></tr>
 <tr><td>string1</td><td>BX 0..15</td><td>byte[0]..byte[15]</td></tr>
 <tr><td>string2</td><td>C0-DF, 16..47</td><td>byte[16]..byte[43], 44,45,46,47 follow 1,2,3,4 bytes</td></tr>
 <tr><td>object</td><td>9X 0..15</td><td>0..13 fixed value, 14,15 follow 1,2 bytes</td></tr>
</table>

<a name="schema"/>
## schema
