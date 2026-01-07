---
title: C++ Algorithm Library
description: some usage of the algorithm
date: 2025-07-15
categories: ["C++"]
tags: ["Algorithm"]
image: yoru.jpg
---

##  sort
Arranges the elements in a specified range into a non-descending order or according to an ordering criterion specified (指定标准) by a binary predicate (二元谓词)

It generally takes `2 + 1 parameters`  
- The `1` one being the point of the vector from where the sorting needs to `begin` 
- The `2` parameter being the `length` up to which we want the vector to get sorted  
- The `(3)` parameter is `optional` and can be used in cases for comp 

By `default`, the sort() function sorts the elements in `ascending order`  

If the equivalent elements exist , try `stable_sort` instead  
It can be called as 'Introsort' combine QuickSort \ HeapSort \ InsertionSort  

### Sort by Vector's Subscript

```cpp
int arr[] = { 0, 1, 5, 8, 9, 6, 7, 3, 4, 2 };
int n = sizeof(arr) / sizeof(arr[0]);
sort(arr + 2, arr + n);  // default std::less<int>() -> ascending order
// sort(arr + 2, arr + n, greater<int>()) -> descending order
// #include <functional> -> greater<int>( )
```

### Sort in a Particular order

We can also write our own comparator function and pass it as a third parameter. 

This `comparator` function `also be called as 'binary predicate'`

- convertible to `bool`
- returns a value whether the passed "first" argument should be placed before the passed "second" argument or not

```cpp
bool compareInterval(Interval i1, Interval i2){
    return (i1.start < i2.start);
}
...
Interval arr[] = { { 6, 8 }, { 1, 9 }, { 2, 4 }, { 4, 7 } };
int n = sizeof(arr) / sizeof(arr[0]);
sort(arr, arr + n, compareInterval);
```

### Why do we use the term "non-descending" instead of "ascending" in sort algorithm ?

`non-ascending` and `non-descending` include the possibility of `adjacent equal`

-  for `[1,2,2]` 's attribute is `non-descending` not `ascending`

### Also we have these sort below

- sort_heap ( make_heap first )  -> the same usage of sort ( contain non-stable )
- stable_partition -> divide array by comparator with origin order -> same usage of sort
- stable_sort -> same of sort but stable -> slow than sort

## swap
exchange two array's element
```cpp
vector<int> v1, v2;
swap( v1, v2 );
```
### swap_ranges

```cpp
vector<int> v1;
deque<int> d1;
swap_ranges ( v1.begin( ), v1.end( ), d1.begin( ) );
```

## unique
Removes duplicate elements that are next to each other in a specified range

```cpp
bool mod_equal ( int elem1, int elem2 ){
	if ( elem1 < 0 )
		elem1 = - elem1;
	if ( elem2 < 0 )
		elem2 = - elem2;
	return elem1 == elem2;
};
...
v1_NewEnd1 = unique ( v1.begin( ), v1.end( ) );
v1_NewEnd2 = unique ( v1.begin( ), v1_NewEnd1 , mod_equal );
v1_NewEnd3 = unique ( v1.begin( ), v1_NewEnd2, greater<int>( ) );
```

### unique_copy
Copies elements from a「source range 」into a「 destination range 」except for the duplicate elements that are next to each other.

```cpp
v1_NewEnd1 = unique_copy ( v1.begin( ), v1.begin( ) + 8, v1.begin( ) + 8 );
v1_NewEnd2 = unique_copy ( v1.begin( ), v1.begin( ) + 14,
	v1.begin( ) + 14 , mod_equal );
```

## upper_bound
Finds the position of the first element in an ordered range that has a value that's greater than a specified value.  
A binary predicate specifies the ordering criterion (指定排序标准).  

```cpp
bool mod_lesser( int elem1, int elem2 ){
	if ( elem1 < 0 )
		elem1 = - elem1;
	if ( elem2 < 0 )
		elem2 = - elem2;
	return elem1 < elem2;
}
...
sort(v1.begin(), v1.end());
sort(v2.begin(), v2.end(), greater<int>());
sort(v3.begin(), v3.end(), mod_lesser);
Result = upper_bound(v1.begin(), v1.end(), 3);
Result = upper_bound(v2.begin(), v2.end(), 3, greater<int>());
Result = upper_bound(v3.begin(), v3.end(), 3, mod_lesser);
```

Starting vector v1 = ( -1 0 1 2 3 4 -3 -2 -1 0 ). 

Original vector v1 with range sorted by the binary predicate less than is v1 = ( -3 -2 -1 -1 0 0 1 2 3 4 ).  
Original vector v2 with range sorted by the binary predicate greater is v2 = ( 4 3 2 1 0 0 -1 -1 -2 -3 ).  
Original vector v3 with range sorted by the binary predicate mod_lesser is v3 = ( 0 0 -1 -1 1 -2 2 -3 3 4 ).  

The upper_bound in v1 for the element with a value of 3 is: 4.  
The upper_bound in v2 for the element with a value of 3 is: 2.  
The upper_bound in v3 for the element with a value of 3 is: 4.  

对于v1，upper_bound 在 v1 中找到第一个大于3的元素，也就是4  
对于v2，由于使用 greater<int>() 作为排序谓词，upper_bound 实际上在查找小于3的第一个元素的位置，即2  
对于v3，使用自定义的 mod_lesser 谓词进行排序，upper_bound 实际上在查找大于3的第一个元素，即4  

Also we have `lower_bound`

## adjacent_find
Searches for two adjacent elements that are either equal or satisfy a specified condition.

```cpp
bool twice (int elem1, int elem2 ){
	return elem1 * 2 == elem2;
}
...
result1 = adjacent_find( L.begin( ), L.end( ) );
result2 = adjacent_find( L.begin( ), L.end( ), twice );

/*
L = ( 50 40 10 20 20 )
There are two adjacent elements that are equal.
They have a value of 20.
There are two adjacent elements where the second is twice the first.
They have values of 10 & 20.
*/
```

## fill 
Assigns the same new value to every element in a specified range

```cpp
vector<int> v1;
fill( v1.begin( ) + 5, v1.end( ), 2 );
```

### full_n
Assigns a new value to a specified number of elements in a range beginning with a particular element

```cpp
auto pos = fill_n( v.begin(), 3, 1 );
fill_n( pos, 3, 2 );
fill_n( v.end()-3, 3, 3 );
```

## is_permutation
Returns true if both ranges contain the same elements, whether or not the elements are in the same order

```cpp
cout << is_permutation(vec_3.begin(), vec_3.end(),
	vec_4.begin(), vec_4.end(),
	[](int lhs, int rhs) { return lhs % 2 == rhs % 2; }) << endl; // true
```

Lambda表达式

```cpp
[capture](parameters) -> return_type {
	// function body
}
```
- capture 是捕获列表，用于在 lambda 表达式中引入外部变量
- parameters 是函数参数列表
- return_type 是返回类型
- function body 是函数体

Also, we have「 is_function」such as:

- is_heap -> is_heap_until
- is_partitioned
- is_sorted -> is_sorted_until

## max
Compare two element \ array \ expression

```cpp
bool abs_greater ( int elem1, int elem2 ){
	if ( elem1 < 0 )
		elem1 = -elem1;
	if ( elem2 < 0 )
		elem2 = -elem2;
	return elem1 < elem2;
};
...
	const int& result1 = max(a, b, abs_greater);
const int& result2 = max(a, b);

vector<int> v1, v2;
v4 = max ( v1, v2 );
```

### max_element
Finds the first occurrence of largest element in a specified range where the ordering criterion may be specified by a binary predicate

```cpp
bool mod_lesser ( int elem1, int elem2 ){
	if ( elem1 < 0 )
		elem1 = - elem1;
	if ( elem2 < 0 )
		elem2 = - elem2;
	return elem1 < elem2;
};
...
	v1_R1_Iter = max_element ( v1.begin( ), v1.end( ) );
v1_R2_Iter = max_element ( v1.begin( ), v1.end( ), mod_lesser);
```

Also, we have `min` and `min_element`

Even, we can use `minmax` and `minmax_element` which return `pair<itn, itx>(min(), max())`

## merge
Combines all of the elements from two sorted source ranges into a single, sorted destination range, where the ordering criterion may be specified by a binary predicate

```c
merge ( v1a.begin( ), v1a.end( ), v1b.begin( ), v1b.end( ), v1.begin( ) );
merge ( v2a.begin( ), v2a.end( ), v2b.begin( ), v2b.end( ),
        v2.begin( ), greater<int>( ) );
merge ( v3a.begin( ), v3a.end( ), v3b.begin( ), v3b.end( ),
        v3.begin( ), mod_lesser );
```

## next_permutation
Reorders the elements in a range so that the original ordering is replaced by the lexicographically next greater permutation if it exists.  
The sense of lexicographically next may be specified with a binary predicate

```c
bool deq1Result;
deq1Result = next_permutation ( deq1.begin( ), deq1.end( ) );
next_permutation ( v1.begin( ), v1.end( ), mod_lesser );

std::vector<int> elements = {1, 2, 3, 4};
do {
    // 在这里处理当前排列
    for (int element : elements) {
        std::cout << element << ' ';
    }
    std::cout << std::endl;
} while (std::next_permutation(elements.begin(), elements.end()));
```

## nth_element
Partitions a range of elements, correctly locating the nth element of the sequence in the range that meets these criteria: All the elements in front of it are less than or equal to it, and all the elements that follow it are greater than or equal to it

```c
nth_element(v1.begin( ), v1.begin( ) + 3, v1.end( ) );
nth_element( v1.begin( ), v1.begin( ) + 4, v1.end( ),greater<int>( ) );
nth_element( v1.begin( ), v1.begin( ) + 5, v1.end( ), UDgreater );
```

## reverse
```c
vector<int> v1;
reverse (v1.begin( ), v1.end( ) );
```

## rotate
Exchanges the elements in two adjacent ranges

```c
rotate ( d1.begin( ), d1.begin( ) + 1 , d1.end( ) );
//( 1 2 3 4 5 0 ) -> ( 2 3 4 5 0 1 )
```
