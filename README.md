# smol_json
A single-header, tiny, non-allocating, streamed JSON parser library for modern C++

## Features
- Single header
- Zero heap allocations
- Incredibly fast iterator access
- No exceptions

## Examples
Constructing a parser:
```cpp
smol_json::Parser parser(std::string_view("[ 1, 2 ]"));
```
Reading a simple string-keyed json object:
```json
{ "integer_key": 42, "boolean_key": false }
```
```cpp
bool success = parser.stringKeyIter([&](auto key) {
	if (key == "integer_key" && parser.expectInt())
		printf("integer_key is %i\n", parser.getInt());
	else if (key == "boolean_key" && parser.expectBool())
		printf("boolean_key is %s\n", parser.getBool() ? "ON" : "OFF");
	else return false;

	return true;
});
```
Output:
```
integer_key is 42
boolean_key is OFF
```
Parsing an array:
```json
[ -1, true, { ... } ]
```
```cpp
bool success = parser.arrayIter([&](int index) {
	if (!parser.readTok()) return false;

	printf("[%i] ", index);

	if (parser.isInt())
		printf("%i\n", parser.getInt());
	else if (parser.isBool())
		printf("%s\n", parser.getBool() ? "ON" : "OFF");
	else if (parser.isObject())
		return parse_some_object(parser);
	else
		return printf("Error.\n") && false;

	return true;
});
```

## Configuration
### SMOL_JSON_DEBUG:
- Define this to enable assertions.

### SMOL_JSON_ABORT(msg):
- Define this for a user abort function.
- Called when SMOL_JSON_DEBUG is defined and an assertion fails.

### SMOL_JSON_LOG_FN(msg, ...):
- Define this for a user log function.
- Only called when SMOL_JSON_DEBUG is defined.
- Defaults to printf with message prefix "[smol_json] ".

### SMOL_JSON_STRING_VIEW:
- Define this to std::string_view/eastl::string_view.
- Defaults to internal implementation smol_json::string_view

## Classes
### smol_json::string_view:
- Simple string_view implementation for standalone.
- Implements generic comparison operators against any class that implements data() and length().

### smol_json::Stream:
- Simple stream based on a string_view.

### smol_json::Token:
- A simple token type.
- When read, values are not cleared immediately: reading a string and then a colon will still keep the string value.

### smol_json::Lexer(smol_json::Token, smol_json::Stream):
- Simple JSON lexer. Currently does not support floating point data.

### smol_json::Parser(smol_json::Lexer):
- Simple JSON parser interface.
- Defines easy iterative APIs
