---
layout: post
title:  "C++17 Early Favorites"
date: 2018-01-09
---

C++ holds a special place in my heart and mind because it was my first
programming language and I still use it heavily today. I was lucky
enough to _really_ start using the language when the 2011 standard was
established but in the growing pain stage with respect to regular
use. I learned a lot following that process and I continue to keep up
with the developments of the language and the standard library.

C++17 was [formally
approved](https://herbsutter.com/2017/09/06/c17-is-formally-approved/)
a few months ago. These days the GCC and Clang developers do an
awesome job implementing features early on, so a lot of C++17 language
and standard library features have been available in a number of
recent releases of both compilers (and their respective STL
implementations). Unfortunately it'll be a little while before I can
use C++17 in my experiment's production code. Nevertheless, I've still
identified a few of my early favorite features. I'll discuss them a
bit with a couple of high energy physics analysis use cases.

## Structured Bindings

When writing Python I always appreciate the ease of writing functions
with multiple returns and the intuitive looping over
dictionaries. Adding structured bindings to C++ makes it easy to use
multiple returns and intuitive to loop over an
[`std::map`](http://en.cppreference.com/w/cpp/container/map).

For multiple returns in C++ 11 and 14 we had to use
[`std::tie`](http://en.cppreference.com/w/cpp/utility/tuple/tie):

```cpp
auto foo() {
  return std::make_tuple(1,2.0,'3');
}

int main() {
  int i;
  float j;
  char k;
  std::tie(i,j,k) = foo();
  // ...
}
```

Now, with structured bindings, it's as easy as:

```cpp
int main() {
  auto [i, j, k] = foo();
  // ...
}
```

Before C++17, when looping over the `std::map` container, we were
locked into using the `first` and `second` members of
[`std::pair`](http://en.cppreference.com/w/cpp/utility/pair):

```cpp
int main() {
  std::map<int,float> myMap {{1,1.1},{2,2.2}};
  for ( const auto& entry : myMap ) {
    doSomething(entry.first,entry.second);
  }
  // ...
}
```

With structured bindings, we have something a bit more intuitive with
less boilerplate:

```cpp
int main() {
  std::map<int,float> myMap {{1,1.1},{2,2.2}};
  for ( const auto& [i, j] : myMap ) {
    doSomething(i,j);
  }
  // ...
}
```

For even more boilerplate, go back to C++03 container looping with
iterators :)

## The Filesystem Library

Consistent with a number of existing C++ STL features...
[Boost](https://www.boost.org) was the original supplier of a C++
filesystem library. It's not always desirable to carry Boost around as
a dependency; avoiding that dependency makes the
[filesystem](http://en.cppreference.com/w/cpp/filesystem) library a
very welcome inclusion to the standard. Having a way to interact with
the filesystem is a very common feature to most langauges' standard
libraries. With respect to C++, It's about time!

The library allows users to parse and modify the filesystem. As an
example, I'll use the task of selecting files with a specific
extension in a given directory. This is a useful piece of code for
processing a large dataset that's broken into many files (I do this a
lot for my physics analysis).

```cpp
namespace fs = std::filesystem;

std::vector<std::string> createDataset(const std::string& path_name,
                                       const std::string& exten) {
  std::vector<std::string> dataset;
  auto itr = fs::directory_iterator(path_name);
  for ( const auto& itr : fs::directory_iterator(path_name) ) {
    auto ext = itr.path().extension().string();
    if ( ext == exten && !fs::is_directory(itr.path()) ) {
      dataset.push_back(itr.path().string());
    }
  }
  return dataset;
}

PhysicsResult graduate() {
  auto dataset = createDataset("/path/to/dir/with/files",".root");
  // dataset will be a a vector containing strings ending in .root, e.g.
  // -- /path/to/dir/with/files/file1.root
  // -- /path/to/dir/with/files/file2.root
  // ...

  // ...
  PhysicsResult thesis = doPhysicsOnDataset(dataset);
  return thesis; // yay!
}

```

## `if constexpr`

The [`constexpr`](http://en.cppreference.com/w/cpp/language/constexpr)
specifier (for constant expressions) was introduced in C++11. The
purpose of the specifier is to communicate to the compiler that the
expression should be evaluated at compile time.

Some awesome things about `if constexpr` are the reduction of
boilerplate and decrease in compile time. `if constexpr` tells the
compiler what to actually compile based on templates, and to ignore
the rest.

Let's say I have three different objects I can analyze, but one of
them is a component of the other two. In particle physics terminology,
I can analyze an electron, a muon, or a track; but, all electrons and
muons have an associated track. If I have an API which supplies a
feature to analyze tracks from containers of all three of these types,
`if constexpr` is great:

```cpp

// API code

void analyzeTrack(const Track& trk) {
  // ...
}

template <typename T>
void analyzeTracks(const std::vector<T>& container) {
  for ( const auto& object : container ) {
    if constexpr ( std::is_same<T,Electron>::value ) {
      analyzeTrack(getTrackFromElectron(object));
    }
    else if constexpr ( std::is_same<T,Muon>::value ) {
      analyzeTrack(getTrackFromMuon(object));
    }
    else if constexpr ( std::is_same<T,Track>::value ) {
      analyzeTrack(object)
    }
  }
}

// user code

int main() {
  analyzeTracks(electronContainer());
  analyzeTracks(muonContainer());
  analyzeTracks(trackContainer());
  // ...
}
```

The `if constexpr` feature of C++17 allows me (while wearing my API
developer hat) to avoid writing the boilerplate of multiple function
overloads and still supply the same easy to use API.

That's it for now - Since we just started 2018 and the compiler
developers are so quick these days, C++20 early favorites are probably
around the corner.