---
title: "[Spring 기초] Object Mapper"
categories:
  - Backend
  - Spring
tags:
  - spring
  - object_mapper
  - jackson_object_mapper
date: 2022-01-23 18:11:00+0900
last_modified_at: 2022-01-23 22:20:00+0900
---
## 개요

Java Object와 JSON의 형태를 자유롭게 변경하기 위해 사용되는 것이 Object Mapper이다. 

1. Serialize: Java Object → JSON
2. Deserialize: JSON → Java Object

형태를 바꿔주는 기능은 위의 두 가지 방식으로 나누어져 있으며 Object Mapper의 가장 중요한 기능이다.

## Jackson ObjectMapper

가장 유명한 Object Mapper는 Jackson ObjectMapper이며 프로젝트 종속성에 추가시켜준 후 사용할 수 있다. Gradle 기준으로 설정하는 방법은 아래와 같다.

```java
dependencies {
  ...
  implementation 'com.fasterxml.jackson.core:jackson-databind:2.13.0'
}
```
{: file="build.gradle"}

![Untitled](https://user-images.githubusercontent.com/80937237/150671482-4dc8af77-0560-48ee-9ab6-b196cbdf8d9d.png)

`build.gradle`의 dependencies에 다음과 같이 추가해주고 gradle을 업데이트 시켜주면 정상적으로 사용이 가능하다.

> 참고: [여기](https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind)에서 jackson databind의 버전을 확인하고 원하는 버전을 implement 할 수 있다. 본 글에서는 게시 기준 가장 사용량이 많은 2.13.0 버전을 사용하도록 한다.
> 

## 기본 사용 예제

```java
public class Member {

  private int id;
  private String name;

  public Member(int id, String name) {
    this.id = id;
    this.name = name;
  }

  public int getId() {
    return id;
  }

  public void setId(int id) {
    this.id = id;
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

}
```

위와 같은 Java class가 있다고 가정하자. 이 Member class를 이용하여 어떻게 serialization, deserialization이 가능한 것인지 알아볼 것이다.

### Serialization (Java Object → JSON)

JSON의 형태로 serialize 시킬 때 objectMapper의 `writeValue()` 와 `writeValueAsString()` 메소드를 사용한다. 이 두 메소드는 각각 파일로 변환, 문자열로 변환의 기능을 갖고 있다.

```java
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import java.io.File;
import java.io.IOException;

public class ObjectMapperPractice {
  public static void main(String[] args) {
    ObjectMapper objectMapper = new ObjectMapper();
    Member member = new Member(1, "Sangyup");

    // Java Object -> JSON
    // 파일 출력: member.json
    try {
      objectMapper.writeValue(new File("원하는 경로/member.json"), member);
    } catch (IOException e) {
      e.printStackTrace();
    }
    
    // 문자열 출력
    try {
      String memberAsJsonString = objectMapper.writeValueAsString(member);
      System.out.println(memberAsJsonString);
    } catch (JsonProcessingException e) {
      e.printStackTrace();
    }
    // 출력값: {"id":1,"name":"Sangyup"}
  }
}
```

<u>이 때, serialize 하려고 하는 class에 Getter가 없다면 안된다.</u>

Jackson 라이브러리는 serialize할 때, Getter의 prefix를 제거하고 소문자로 만들어서 식별하기 때문에 가져올 Getter가 없다면 오류를 내보내게 된다.

```json
{"id":1,"name":"Sangyup"}
```
{: file="member.json"}

파일 출력의 결과로 위와 같이 `member.json` 이 생성되어 있음을 확인할 수 있다. 

### Deserialization (JSON → Java Object)

JSON을 Java Object로 deserialize시킬 때는 objectMapper의 `readValue()` 메소드를 이용한다.

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.core.JsonProcessingException;

public class ObjectMapperPractice {
  public static void main(String[] args) {
    ObjectMapper objectMapper = new ObjectMapper();

    // JSON -> Java Object
    String json = "{\"id\":2, \"name\":\"Sangyup Lee\"}";
    try {
      Member newMember = objectMapper.readValue(json, Member.class);
      System.out.println(newMember);
      System.out.println(newMember.getId() + " " + newMember.getName());
    } catch (JsonProcessingException e) {
      e.printStackTrace();
    }
  }
}
```

이 때 아래와 같은 오류가 발생한다.

```
Cannot construct instance of `ObjectMapperPractice.Member` (no Creators, like default constructor, exist): cannot deserialize from Object value (no delegate- or property-based Creator)
 at [Source: (String)"{"id":2, "name":"Sangyup Lee"}"; line: 1, column: 2]
```

이러한 오류가 생기는 이유는 Member class에 기본 생성자가 없어서 그렇다.

```java
public class Member {
  public Member() {
  }
  ...
}
```

따라서 위와 같이 기본 생성자를 추가해주고 실행한다면 정상적으로 출력결과를 통해 Java Object가 생성되었음을 알 수 있다.

> 기본 생성자 이외에도 어노테이션 등을 사용해서 하는 방법이 있다. 예제에선 단순히 기본 생성자를 추가해서 사용하였지만 상황에 맞게 방법을 바꿔야 할 것이다. 다른 방법은 [여기](https://www.baeldung.com/jackson-annotations)에서 확인할 수 있다.
> 

## 심화 사용 예제

### Configure를 통한 옵션 설정

```java
import com.fasterxml.jackson.databind.DeserializationFeature;
```

- JSON에는 존재하지만 대상 class에 field 값이 존재하지 않을 때
    
```java
String json = "{\"id\":2, \"name\":\"Sangyup Lee\", \"age\":30}";
objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
try {
    Member newMember = objectMapper.readValue(json, Member.class);
    System.out.println(newMember);
    System.out.println(newMember.getId() + " " + newMember.getName());
} catch (JsonProcessingException e) {
    e.printStackTrace();
}
```
    
`FAIL_ON_UNKNOWN_PROPERTIES` 를 false로 작성하게 되면 class에 존재하지 않는 field는 무시된다.
    
- JSON에 null 값이 존재하고 이를 primitive type에 넘겨주어야 할 때
    
```java
objectMapper.configure(DeserializationFeature.FAIL_ON_NULL_FOR_PRIMITIVES, false);
```

`FAIL_ON_NULL_FOR_PRIMITIVES` 를 false로 작성하게 되면 null값이 무시되고 해당 primitive type의 기본값이 저장된다.
    

### Custom Serializer와 Deserializer

JSON의 구조와 Java class의 구조가 다른 상황에서 serialize 혹은 deserialize 해야할 때 유용하다.

- **Custom Serializer**

```java
public class CustomCarSerializer extends StdSerializer<Car> {
  public CustomMemberSerializer() {
    this(null);
  }

  public CustomMemberSerializer(Class<Member> t) {
    super(t);
  }

  @Override
  public void serialize(
      Member member, JsonGenerator jsonGenerator, SerializerProvider serializer) {
    try {
      jsonGenerator.writeStartObject();
      jsonGenerator.writeStringField("your_name_is", member.getName());
      jsonGenerator.writeEndObject();
    } catch (IOException e) {
      e.printStackTrace();
    }
  }
}
```

```java
SimpleModule module =
    new SimpleModule("CustomMemberSerializer", new Version(1, 0, 0, null, null, null));
module.addSerializer(Member.class, new CustomMemberSerializer());
objectMapper.registerModule(module);
Member member = new Member(3, "Smith");
try {
  String memberAsJsonString = objectMapper.writeValueAsString(member);
  System.out.println(memberAsJsonString);
  // 출력결과: {"your_name_is":"Smith"}
} catch (JsonProcessingException e) {
  e.printStackTrace();
}
```

- **Custom Deserializer**

```java
class CustomMemberDeserializer extends StdDeserializer<Member> {

  public CustomMemberDeserializer() {
    this(null);
  }

  public CustomMemberDeserializer(Class<?> vc) {
    super(vc);
  }

  @Override
  public Member deserialize(JsonParser parser, DeserializationContext deserializer) {
    Member member = new Member();
    ObjectCodec codec = parser.getCodec();
    JsonNode node = null;
    try {
      node = codec.readTree(parser);
    } catch (IOException e) {
      e.printStackTrace();
    }

    // 이름만을 빼내서 Object로 deserialize한다.
    JsonNode nameNode = node.get("name");
    try {
      // node.get의 return 값이 null일 수 있다.
      String name = nameNode.asText();
      member.setName(name);
    } catch (NullPointerException e) {
      e.printStackTrace();
    }
    return member;
  }
}
```

```java
String json = "{ \"id\" : 4, \"name\" : \"James\" }";
SimpleModule module =
    new SimpleModule("CustomMemberDeserializer", new Version(1, 0, 0, null, null, null));
module.addDeserializer(Member.class, new CustomMemberDeserializer());
objectMapper.registerModule(module);

try {
  Member member = objectMapper.readValue(json, Member.class);
  System.out.println(member.getName());
} catch (JsonProcessingException e) {
  e.printStackTrace();
}
```

위의 두 가지 예제처럼 구조가 달라져도 custom하게 설정을 해줄 수 있다.

### Date Format 사용

date format을 사용하여 JSON으로 serialize 해줄 수 있다.

```java
class MemberStatus {
  private int id;
  private Date registeredAt;
  ...
}
```

```java
MemberStatus memberStatus = new MemberStatus();
memberStatus.setId(1);
memberStatus.setRegisteredAt(Date.from(Instant.now()));

ObjectMapper objectMapper = new ObjectMapper();
DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm a z");
objectMapper.setDateFormat(df);
try {
  String memberAsJsonString = objectMapper.writeValueAsString(memberStatus);
  // 출력결과: {"id":1,"registeredAt":"2022-01-23 17:00 PM KST"}
  System.out.println(memberAsJsonString);
} catch (JsonProcessingException e) {
  e.printStackTrace();
}
```

### Handling Collections

JSON에 2개 이상의 데이터가 존재한다면 Collection의 형태로 deserialize 할 수 있다.

- As an Array

```java
String json =
    "[{\"id\":5, \"name\":\"Tony\"}, {\"id\":6, \"name\":\"John\"}]";
objectMapper.configure(DeserializationFeature.USE_JAVA_ARRAY_FOR_JSON_ARRAY, true);
try {
  Member[] members = objectMapper.readValue(json, Member[].class);
} catch (JsonProcessingException e) {
  e.printStackTrace();
}
```

- As a List

```java
String json =
    "[{\"id\":5, \"name\":\"Tony\"}, {\"id\":6, \"name\":\"John\"}]";
objectMapper.configure(DeserializationFeature.USE_JAVA_ARRAY_FOR_JSON_ARRAY, true);
try {
  List<Member> memberList = objectMapper.readValue(json, new TypeReference<List<Member>>(){});
} catch (JsonProcessingException e) {
  e.printStackTrace();
}
```

## 참고자료

[https://www.baeldung.com/jackson-object-mapper-tutorial](https://www.baeldung.com/jackson-object-mapper-tutorial)
