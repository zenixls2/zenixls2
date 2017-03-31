---
layout: post
title: "C++ STL wrapper in go (I)"
description: "implementation on iterator wrapping and other container"
tags: [note]
---
Actually there are already some implementation based on SWIG that you can use directly, but the STL methods provided by SWIG does not contain any iterator operation. You may follow my [first project](https://github.com/zenixls2/stl) to wrap by C in hand, or follow my [second project](https://github.com/zenixls2/stl2) to create a new SWIG file to define those iterator methods. However, If you were only to pursue speed, you had better re-implement the key algorithm by yourself instead of calling cgo wrapped C/C++ implementations. The serialization/deserialization of Go parameter objects may take too much time for a function call either from C to Go or from Go to C.

## Look into C Wrapper
#### The C/C++ Part

The cgo compiler cannot call the C++ class methods directly. Instead, it requires you to create a C wrapper on top of it. Notice that currently we still have no smart solution to cope with templates (parametric polymorphism).

```c++
// here we chose to implement map<int, object_ptr>
#ifdef __cplusplus
extern "C" {
#endif /* __cplusplus */
  typedef void* Map;
  typedef void* MapIterator;
  Map MapInit(void);
  void MapAdd(Map m, int key, uintptr_t value);
  uintptr_t MapGet(Map m, int key);
  MapIterator MapLowerBound(Map m, int key);
  MapIterator MapUpperBound(Map m, int key);
  uintptr_t MapIteratorGet(MapIterator iter);
  int MapIteratorGetKey(MapIterator iter);
  MapIterator MapIteratorNext(MapIterator iter);
  MapIterator MapIteratorPrev(MapIterator iter);
#ifdef __cplusplus
}
#endif /* __cplusplus */
```
<br>

Next create files implement C wrapper functions:

```c++
#include <map>
#include <iterator>
#include <utility>
#include <iostream>
#include "map.h"

#define MAP_TYPE std::map<int, uintptr_t>
typedef MAP_TYPE::iterator iter_type;

Map MapInit()
{
  return (Map)(new MAP_TYPE());
}

void MapFree(Map m)
{
  std::map<int, uintptr_t>* mptr =
      static_cast<MAP_TYPE*>(m);
  delete mptr;
}

void MapAdd(Map m, int key, uintptr_t value)
{
  static_cast<MAP_TYPE*>(m)
    ->insert(std::make_pair(key, value));
}

// map operator[]
uintptr_t MapGet(Map m, int key)
{
  return (static_cast<MAP_TYPE*>(m)->find(key))
      ->second;
}

// self-defined data structure to hold iterators
typedef struct _iterator {
  iter_type iter; // cursor iterator
  iter_type begin; // holds the map begin() iterator
  iter_type end; // holds the map end() iterator
} _iterator;

// implement map::lower_bound
MapIterator MapLowerBound(Map m, int key)
{
  auto ret = new _iterator();
  auto container = static_cast<MAP_TYPE*>(m);
  ret->iter = container->lower_bound(key);
  ret->begin = container->begin();
  ret->end = container->end();
  return static_cast<MapIterator>(ret);
}

// implement map::upper_bound
MapIterator MapUpperBound(Map m, int key)
{
  auto ret = new _iterator();
  auto container = static_cast<MAP_TYPE*>(m);
  ret->iter = container->upper_bound(key);
  ret->begin = container->begin();
  ret->end = container->end();
  return static_cast<MapIterator>(ret);
}

// extract value from _iterator
uintptr_t MapIteratorGet(MapIterator iter)
{
  return (static_cast<_iterator*>(iter))->iter->second;
}

// extract key from _iterator
int MapIteratorGetKey(MapIterator iter)
{
  return (static_cast<_iterator*>(iter))->iter->first;
}

// Iterate to the next position
MapIterator MapIteratorNext(MapIterator iter)
{
  auto ret = (static_cast<_iterator*>(iter));
  ret->iter++;
  if (ret->iter == ret->end)
    return NULL;
  return iter;
}

// Iterate to the previous position
MapIterator MapIteratorPrev(MapIterator iter)
{
  auto ret = (static_cast<_iterator*>(iter));
  if (ret->iter == ret->begin)
    return NULL;
  ret->iter--;
  return iter;
}
```
<br>

If you're using Mac for development, maybe you would like to add some pragma when you try to wrap c++11/14/17 extensions:

```c++
#pragma clang diagnostic push
#pragma clang diagnostic ignore "-Wc++11-extensions"

<Code Segments>

#pragma clang diagnostic pop
```
<br>

#### The Golang Part

In this part, we would define the golang wrapper for the C interface. All things are the same as wrapping other C API, you have to define the cgo flags, include the headers, and call the functions from C.

```golang
// +build cgo
package stl

// we only define Mac related flags here. define yours based on your needs.
// #cgo darwin LDFLAGS: -lstdc++ -L ./
// #cgo darwin CFLAGS: -O2
// #include <stdint.h>
// #include "map.h"
import "C"  // no blank lines allowed between the declarations and import
import (
  "reflect"
)

type Map struct {
  m C.Map // keep private
}

// creates a new empty map
func NewMap() Map {
  var ret Map
  ret.m = C.MapInit()
  return ret
}

// Free release memory hold by map containers.
// be sure to call this to prevent memory leak.
func (sm Map) Free() {
  C.MapFree(sm.m)
}

// Add key, value pair to map
// We extract the memory address in golang as value in C.
// Make sure your C code doesn't do direct access to golang memory,
// notice that this a forbidden behavior in golang design if you do so.
func (sm Map) Add(key int, value interface{}) {
  C.MapAdd(sm.m, C.int(key), C.uintptr_t(reflect.ValueOf(value).Pointer()))
}

// Get value by key from map
// Due to the limitation of Golang,
// we are not able to do type casting based on the added type.
// we have to do casting handy after address is got.
func (sm Map) Get(key int) uintptr {
  return uintptr(C.MapGet(sm.m, C.int(key)))
}

// Here we create a Iterator type to hold iterators we defined in C.
// Would explain the implementation detail later in the paragraph.
// returns an iterator to the first element not less than the given key.
func (sm Map) LowerBound(key int) Iterator {
  iter := MapIterator{}
  iter.i = C.MapLowerBound(sm.m, C.int(key))
  return iter
}

// returns a iterator to the first element greater than the given key
func (sm Map) UpperBound(key int) Iterator {
  iter := MapIterator{}
  iter.i = C.MapLowerBound(sm.m, C.int(key))
  return iter
}
```
<br>

As for Iterator, we first define a Interface for it:

```golang
type Iterator interaface {
  Value() uintptr
  Key() int
  Next() Iterator
  Prev() Iterator
}
```
<br>

And this should be reusable in other STL implementation.
Now start to implement the iterator for map:

```golang
// +build cgo
package stl

/// we only define Mac related flags here. define yours based on your needs.
// #cgo darwin LDFLAGS: -lstdc++ -L ./
// #cgo darwin CFLAGS: -O2
// #include <stdint.h>
// #include "map.h"
import "C"

// struct that implements Iterator interface
type MapIterator struct {
  i C.MapIterator
}

// get element pointer from iterator
func (si MapIterator) Value() uintptr {
  return uintptr(C.MapIteratorGet(si.i))
}

// get element key from iterator
func (si MapIterator) Key() int {
  return int(C.MapIteratorGetKey(si.i))
}

// iterator move to next
func (si MapIterator) Next() Iterator {
  if C.MapIteratorNext(si.i) == nil {
    return nil
  }
  return si
}

// iterator move to prev
func (si MapIterator) Prev() Iterator {
  if C.MapIteratorPrev(si.i) == nil {
    return nil
  }
  return si
}
```
<br>

And that's all!
Let's play with our newly created wrapper:

```golang
package main
import (
  "github.com/zenixls2/stl"
  "fmt"
)
func main() {
  m := stl.NewMap()
  defer m.Free()
  a := []string{"xdd", "123", "ohohoh", "abc"}
  m.Add(1, &a[0])
  m.Add(3, &a[1])
  m.Add(5, &a[2])
  m.Add(7, &a[3])

  // should be "xdd"
  fmt.Println(*(*string)(unsafe.Pointer(m.Get(1))))

  // should output "ohohoh" and "abc"
  iter := m.LowerBound(5)
  for end := iter; end != nil; end = iter.Next() {
    fmt.Println(*(*string)(unsafe.Pointer(iter.Value())))
  }
}
```
<br>

Just make sure if you are using Xcode 8.3, compile with `go build -ldflags -s xxx.go` to avoid this bug: [issue 19734](https://github.com/golang/go/issues/19734)

There should be a backport in the future after 1.8.1 released.
