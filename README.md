# uritemplate
This library is a Java implementation of RFC 6570 (URI template) with a number of
additional features as shown in examples. To avoid introducing security issues as
much as possible, this implementation relies on standard Java implementation
of URI (java.util.URI class).

Example 1: basics of RFC 6570 support
```java
	String s = new URITemplate("http://www.example.com/api{?param*}")
			.expand("param", asList("1","2"))
			.toString();
    assertEquals("http://www.example.com/api?param=1&param=2", s);
```

Example 2: URI builder DSL - paths, variables and expansions
```java
String s2 = new URIBuilder("http://www.example.com")
		.relative("api", "items", var("name"))
		.asTemplate()
		.expand("name", "1")
		.toString();
assertEquals("http://www.example.com/api/items/1", s2);
```

Example 3: URI builder DSL - pre-encoded variables
```java
String s3 = new URIBuilder("http://www.example")
		.append(var("tld").preEncoded())
		.toString();
assertEquals("http://www.example{+tld}", s3);
	
String expanded = new URITemplate("http://www.example{+tld}")
		.expand("tld", "{.domain}")
		.toString();
assertEquals("http://www.example{.domain}", expanded);
```

Example 4: URI builder DSL - path variables
```java
String s4 = new URIBuilder("http://www.example.com")
		.append(pathVariable("name"))
		.toString();
assertEquals("http://www.example.com{/name}", s4);
```

Example 5: URI builder DSL - escaping in path
```java
String s5 = new URIBuilder("http://www.example.com")
		.relative("api", "items", "{name}")
		.toString();
assertEquals("http://www.example.com/api/items/%7Bname%7D", s5);
```

Example 6: escaping of query parameters
```java
String s6 = new URIBuilder("http://www.example.com?val1=%25")
		.queryParam("val2", "%", "$", "#")
		.queryParam("val3", "hello")
		.toString();
assertEquals("http://www.example.com?val1=%25&val2=%25&val2=$&val2=%23&val3=hello", s6);
```

Example 7: ':' as control character
```java
String s = new URITemplate("rel{:param*}")
		.expand("param", asList("1","2"))
		.toString();
assertEquals("rel:1:2", s);
```

Example 8: partial template expansion and URITemplateParser
```java
String uri = "http://www.example.com{?p1,p2,p3*}";
String expected = "http://www.example.com?p2=v2&p2=v3{&p1,p3*}";

Map<String, Object> map = new HashMap<>();
map.put("p2", asList("v2", "v3"));

String result = URITemplateParser.parseAndExpand(uri, map);

assertEquals(expected, result);
```

Example 9: advanced composition of URI components with appropriate delimiters - `join()` method
```java
String s = new URIBuilder("http://www.example.com?val1=%25")
                .path().join("api", "subpath")
                .toString();
assertEquals("http://www.example.com/api/subpath?val1=%25", s);
```
The approach above works with `host()`, `path()`, `query()` and `fragment()` methods.

Example 10: advanced composition of URI components - `append()` method
```java
String s = new URIBuilder("http://www.example.com?val1=%25")
                .path().append("/api", "subpath")
                .toString();
assertEquals("http://www.example.com/apisubpath?val1=%25", s);
```

Example 10: "server" editing for cloud service discovery scenarios
```java
String serviceLink = new URIBuilder("https://myservice-01-01-01-01.somecloud.com:8765/api/v1/endpoint")
                .server(template("services.myservice"))
                .asTemplate()
                .toString();
assertEquals("{services.myservice}/api/v1/endpoint", serviceLink);

String publicLink = new URITemplate(serviceLink)
                .expand(singletonMap("services.myservice", "https://api.mydomain.com/myservice"));
assertEquals("https://api.mydomain.com/myservice/api/v1/endpoint", publicLink);
```