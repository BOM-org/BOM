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
  * [design explanation](#design)

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
  * **null** is used in 2 usecases. 
   * **value** case, it represents a null value, f.e. object{special:null}
   * **type** case, it represents any types, like Any in Scala, Object in Java. f.e. Tuple(int,float,string,null) means Tuple(int,float,string,Any)
  * **magic** represents a magic value, used to handle schema, reference, headers, and so on.
  
<a name="format"/>
## format

<a name="overview"/>
### overview

<table>
 <tr><th>bits format</th><th>hex</th><th>description</th><th>data representation</th></tr>
 <tr><td>0xxx xxxx</td><td>00-7F</td><td>small integer range 0..127, compatible with singed and unsigned byte integer</td><td>just one byte</td></tr>
 <tr><td>1000 xxxx</td><td>80-8F</td><td>type tags</td><td>denpends</td></tr>
 <tr><td>1001 xxxx</td><td>9_X</td><td>object(n), n for number of pairs</td><td>[9N][N pairs]</td></tr>
 <tr><td>1010 xxxx</td><td>A_X</td><td>array(T), follows number of values</td><td>[AT][N][N values]</td></tr>
 <tr><td>1011 xxxx</td><td>B_X</td><td>string(n), 0..15</td><td>[x_N][N bytes]</td></tr>
 <tr><td>110x xxxx</td><td>C0-DF</td><td>string(n), n for length of UTF bytes, 16..47</td><td>[x_N][N bytes]</td></tr>
 <tr><td>111x xxxx</td><td>E0-FF</td><td>negative byte integer, range -1..-32</td><td>just one byte</td></tr>
</table>

### type tags

<table>
 <tr><th>xxxx format</th><th>hex</th><th>description</th><th>followed data representation</th></tr>
 <tr><td>0000-0011</td><td>0-3</td><td>singed integers, byte 1, 2, 3, 4</td><td>[byte 1 2 4 8]</td></tr>
 <tr><td>0100-0111</td><td>4-7</td><td>unsigned integers, ubyte 1, 2, 3, 4</td><td>[byte 1 2 4 8]</td></tr>
 <tr><td>1000</td><td>8</td><td>float</td><td>[4 bytes]</td></tr>
 <tr><td>1001</td><td>9</td><td>double</td><td>[8 bytes]</td></tr>
 <tr><td>1010</td><td>A</td><td>string</td><td>[N][N bytes]</td></tr>
 <tr><td>1011</td><td>B</td><td>array, T[], 1 or multi Dimensions</td><td>1D: [T][+N][N values]<br/>mD: [T][-i][D0..Di][E(Di) values]</td></tr>
 <tr><td>1100</td><td>C</td><td>tuple<T0, T1...Tn>, n>=2</td><td>[N][T0..Tn][N values]</td></tr>
 <tr><td>1101</td><td>D</td><td>object, array of key-value pair</td><td>[N][N pairs]</td></tr>
 <tr><td>1110</td><td>E</td><td>null</td><td>just single byte</td></tr>
 <tr><td>1111</td><td>F</td><td>magic</td><td>extensions</td></tr>
</table>
There are most 16 kinds of type tag, so it's just a half byte.
 * **single byte tag** used in object.pair.value or any[], the type tag is 80-8F, or fast style, 9X,AX,BX,CX,DX, see [table](#overview)
 * **half byte tag** used in array type, T[] is encoded as AT[N][n values]; used in tuple (T0, T1, T2, ... Tn), each two type tag form a byte, encoded as CTTTTT[n values]. maybe used in [schema declarations](#schema)

### length representation
there are 2 kinds of length reprententation
#### full bytes length 
the length starts with a full byte, and follows variable-length bytes
<table>
 <tr><th>format</th><th>bits</th><th> max</th></tr>
 <tr><td>0xxx xxxx</td><td>7</td><td>`(2^7)-1`</td></tr>
 <tr><td>100x xxxx byte</td><td>5+8=13</td><td>`(2^13)-1`</td></tr>
 <tr><td>101x xxxx byte byte</td><td>5+16=21</td><td>`(2^21)-1`</td></tr>
 <tr><td>11xx xxxx byte byte byte</td><td>6+24=30</td><td>`(2^30)-1`</td></tr>
</table>

#### partial bytes length
the length starts with a partial byte, and maybe follow bytes
<table>
 <tr><th>format</th><th>bits</th><th> range</th></tr>
 <tr><td>string1</td><td>BX 0..15</td><td>byte[0]..byte[15]</td></tr>
 <tr><td>string2</td><td>C0-DF, 16..47</td><td>byte[16]..byte[43], 44,45,46,47 follow 1,2,3,4 bytes</td></tr>
 <tr><td>object</td><td>9X 0..15</td><td>0..13 fixed value, 14,15 follow 1,2 bytes</td></tr>
</table>

<a name="schema"/>
## schema


<a name="design"/>
## design explanation

There are many binary structured message formats. BOM learn many aspects from [MsgPack.org](http://msgpack.org).
* little integer
 BOM encode little integer in just one byte, range from -32..-1, 0..127, there is no additional byte to need.
* string
 string, only UTF-8 encoding directly supported. In most usecases, there are only short strings. I did a simple statistics, most string length less than 64. MsgPack use 32 fast length. BOM use 0..47 as fast length, just one byte header.
* array
 a sequence of value. the best name should T[], where T is type of array element, all 14 type supported.
 * **number** includes integers and floats, 10.
 * **string** represents string[]
 * **any** uses null tag _E_, represents any[], some like Object[] in java, but primary allowed.
 * **complicated** include object, tuple.
 * <b>NOTE T cannot be T[] self, but multi-dimension array still supported.</b> f.e.  T[2][3][4] is encoded as  A_T_-3_2_3_4_V[12]
* object
 associcated-array is a key-value pair array. some same as string, object use 0..16 fast length, just one byte header. (FMV, I not use objects with more than 16 fields).<br/>
 <b>NOTE MsgPack name this type as map. because BOM introduce tuple(T...) to do generics work, Tuple(K,V)[] is same as java.util.Map type, so the "map" name not used in BOM</b>
 <b>NOTE the key of object is NOT limited string type, it can be any type except the null.</b>Through the key type is not string, there is no additional byte cost because most object.key string is short-length, just one byte header BX, CX or DX.
* tuple
 * **definition** A tuple is a finite ordered list of elements. In mathematics, an n-tuple is a sequence (or ordered list) of n elements, where n is a non-negative integer. see the definition from [wikipedia](https://en.wikipedia.org/wiki/Tuple)
 * **format** A tuple header encoded as _D_N_XXXX, if n is odd, the last half byte use magic value, f.e. _D_3_xxxF. <b>again NOTE 0xE used in tuple, indicated any type</b>
 * **object simplification** in object, all its pair use typed value. when many objects in message, the header byte complicating is a big waste. so the most efficient way is converting the object to tuple. f.e. point{x:float,y:float}[]==>points{field string[]:x,y;values:tuple<float,float>:1,2,3,4...
 * **map** (https://en.wikipedia.org/wiki/Hash_table) object is equivelent to Tuple(Key,Any)[] where key cannot be null.
