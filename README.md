ObjectBox for Dart/Flutter
==========================
ObjectBox for Dart is a standalone database storing Dart objects locally, with strong ACID semantics.

Help wanted
-----------
ObjectBox for Dart is still in a prototype stage supporting only the most basic database tasks, like putting and getting objects.
However, the ObjectBox core supports many more features, e.g. queries, indexing, async operations, transaction control.
To bring all these features to Dart, we're asking the community to help out. PRs are more than welcome!
The ObjectBox team will try its best to guide you and answer questions. 

### Feedback
Also, please let us know your feedback by opening an issue:
for example, if you experience errors or if you have ideas for how to improve the API.
Thanks!

Getting started
---------------
To try out the demo code in this repository, follow these steps:

1. Install [objectbox-c](https://github.com/objectbox/objectbox-c) system-wide:
   * macOS/Linux: `bash <(curl -s https://raw.githubusercontent.com/objectbox/objectbox-c/master/download.sh) 0.7` (answer Y when it asks about installing to /usr/lib).
   * Windows: 
        * use "Git Bash" or similar to execute `bash <(curl -s https://raw.githubusercontent.com/objectbox/objectbox-c/master/download.sh) 0.7`
        * copy the downloaded `lib/objectbox.dll` to `C:\Windows\System32\` (requires admin privileges)
2. Back in this repository, run `pub get`.
3. Execute `pub run build_runner build`. This regenerates the ObjectBox model to make it usable in Dart 
   (i.e. the file `test/test.g.dart`) and is necessary each time you add or change a class annotated with `@Entity(...)`.
4. Finally run `pub run test` to run the unit tests.

Mac OS signed dart binary issue
-------------------------------

If dart complains that it cannot find the `libobjectbox.dylib`, after you've installed by following step 1,
then you probably have to unsign the dart binary:

```bash
sudo xcode --remove-signature `which dart`
``` 

source: [dart issue](https://github.com/dart-lang/sdk/issues/38314#issuecomment-534102841)

Dart integration
----------------
In general, Dart class annotations are used to mark classes as ObjectBox entities and provide meta information.
Note that right now, only a limited set of types is supported; this will be expanded upon in the near future.
Entity IDs and UIDs that are defined in their respective annotations need to be unique across all entities, while 
property IDs only need to be unique in their respective entity; property UIDs also need to be globally unique.

### Object IDs

Each entity is required to have an _Id_ property of type _Long_.
Already persisted entities have an ID greater or equal to 1.
New (not yet persisted) objects typically have _Id_ value of `0` or `null`: calling `Box.put` automatically assigns a new ID to the object.

### Example

```dart
import "package:objectbox/objectbox.dart";
part "note.g.dart";

@Entity()
class Note {
    @Id()       // automatically always 'int' in Dart code and 'Long' in ObjectBox
    int id;

    String text;

    Note();             // empty default constructor needed
    Note.construct(this.text);
    toString() => "Note{id: $id, text: $text}";
}
```

In your main function, you can then create a _store_ which needs an array of your entity classes and definitions to be constructed. If you have several entities, construct your store like `Store([[Entity1, Entity1_OBXDefs], [Entity2, Entity2_OBXDefs]])` etc.
Finally, you need a _box_, representing the interface for objects of one specific entity type.

```dart
var store = Store([Note_OBXDefs]);
var box = Box<Note>(store);

var note = Note.construct("Hello");
note.id = box.put(note);
print("new note got id ${note.id}");
print("refetched note: ${box.get(note.id)}");

store.close();
```

Query and QueryBuilder
----------------------

Basic querying can be done with e.g.:

```dart
// var store ...
// var box ...

box.putMany([Note(), Note(), Note()]);
box.put(Note.construct("Hello world!"));

final queryNullText = box.query(Note_.text.isNull()).build();

assert (queryNullText.count() == 3);

queryNullText.close(); // We have to manually close queries and query builders.
```

More complex queries can be constructed using `and/or` operators.
Also there is basic operator overloading support for `equal`, `greater`, `less`, `and` and `or`,
respectively `==`, `>`, `<`, `&`, `|`.

```dart
// final box ...

box.query(text.equal("meh").or(text.equal("bleh")).or(text.contains("Hello"))).build();

// equivalent to

final overloaded = ((text == "meh") | (text == "bleh")) | text.contains("Hello");
box.query(overloaded as Condition).build(); // the cast is necessary due to the type analyzer
```

Ordering
--------

The results from a query can be ordered using the `order` method, e.g.

```dart
final q = box.query(Entity_.number > 0)
  .order(Type_.number)
  .build();

// ...

final qt = box.query(Entity_.text.notNull())
  .order(Entity_.text, flags: OBXOrderFlag.DESCENDING | OBXOrderFlag.CASE_SENSITIVE)
  .build();
```

Basic technical approach
------------------------
ObjectBox offers a [C API](https://github.com/objectbox/objectbox-c) which can be called by [Dart FFI](https://dart.dev/server/c-interop).
The C API is is also used by the ObjectBox language bindings for [Go](https://github.com/objectbox/objectbox-go), [Swift](https://github.com/objectbox/objectbox-swift), and [Python](https://github.com/objectbox/objectbox-python).
These language bindings currently serve as an example for this Dart implementation.

Internally, ObjectBox uses [FlatBuffers](https://google.github.io/flatbuffers/) to store objects.
There are two basic ways to make the conversion: generated binding code, or implicit FlatBuffers conversion.
The latter is used at the moment (helped us to get started quickly).
A future version will exchange that with code generation.  

See also
---------
* [Changelog](CHANGELOG.md)
* [Contribution guidelines](CONTRIBUTING.md)

License
-------
    Copyright 2019 ObjectBox Ltd. All rights reserved.
    
    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at
    
        http://www.apache.org/licenses/LICENSE-2.0
    
    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

