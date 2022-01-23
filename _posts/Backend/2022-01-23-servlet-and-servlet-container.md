---
title: "[Spring 기초] 서블릿(servlet) 과 서블릿 컨테이너(servlet container)"
categories:
  - Backend
  - Spring
tags:
  - Servlet
  - JSP
  - Spring
date: 2022-01-23 14:23:00+0900
---

## 서블릿

Dynmaic Web Page(동적 웹페이지)를 사용하기 위해 활용되는 기술로 클라이언트의 요청(request)을 처리하고 그 결과(response)를 반환하는 Servlet 클래스의 구현 규칙을 지킨 자바 웹 프로그래밍 기술이다. 서블릿이 없다면 개발자는 request와 response의 규약을 지키기 위해 상당히 많은 시간을 할애해야 할 것이며 코드 또한 굉장히 복잡해진다. 이 때, 서블릿이 이러한 request와 response를 간단한 메소드 호출만으로 가능하게 해주어, 개발자가 더욱 로직에만 집중할 수 있게끔 도와준다. 

- 특징
    - Java의 쓰레드를 이용하여 동작
    - MVC 패턴의 컨트롤러로써 동작한다.
    - http 프로토콜을 지원하는 `javax.servlet.http.HttpServlet` 클래스를 상속한다.
    - html을 사용하여 응답하며 html 변경 시 servlet을 재컴파일 해야한다.
    - `init()`, `service()`, `destroy()` 등의 메소드가 있다.

## 서블릿 컨테이너

![Untitled](https://user-images.githubusercontent.com/80937237/150666133-2400e716-046c-4767-ad5e-8f12864a09e8.png)

서블릿을 등록시켰다고 해서 서블릿이 자동으로 동작하는 것은 아니다. 서블릿을 관리하는 주체가 필요한데 그 역할을 해주는 것이 서블릿 컨테이너이다. 서블릿 컨테이너는 웹 서버와 소켓으로 통신하며 클라이언트의 요청을 받고 응답을 해준다. 서블릿 컨테이너의 대표적인 예가 **톰캣(Tomcat)**이다.

- 서블릿의 lifecycle을 관리
    
    서블릿 컨테이너가 동작하면 서블릿 클래스를 인스턴스화하고 `init()` 함수를 호출한다. 이후 요청에 따라 적절한 메소드를 찾아서 호출하고 서블릿이 사라질 때에는 garbage collection을 통해 메모리에서 제거한다.
    
- 웹 서버와의 통신 지원
    
    클라이언트의 Request를 받고 Response를 보내주기 위해 웹 서버와 소켓을 만들어 통신한다. listen, accept 등등을 API로 구현하여 개발자로 하여금 로직에만 신경 쓸 수 있게 해준다.
    
- 멀티 쓰레딩 관리
    
    요청이 올 때마다 자바 쓰레드를 하나 생성하게 되는데 service 메소드 실행 후 자연스럽게 쓰레드를 삭제한다. 멀티 쓰레드의 관리를 해주기 때문에 쓰레드의 안정성을 신경쓰지 않아도 된다.
    
- 선언적인 보안 관리
    
    보안 관리를 xml를 통해 진행하기 때문에 보안 쪽 설정이 변경되었다하더라도 소스코드를 고치고 재컴파일 할 필요가 없다.
    

서블릿 컨테이너에 의해 서블릿이 호출되고 사라지는데 클라이언트로부터 요청을 받으면 현재 실행할 서블릿이 그 전에 호출된 적이 있는지 판단하고 없다면 서블릿을 생성한다. 이 과정은 서블릿 별로 단 한 번만 동작하며 서블릿 컨테이너가 내려갈 때 해당 컨테이너가 관리하고 있던 서블릿은 사라지게 된다. 이를 **싱글톤 형태**라고 부른다. 한 번 생성된 서블릿은 이후 같은 요청에 대해 재사용된다.

## 서블릿 사용방법

### 서블릿 등록

요청에 따라 적합한 서블릿을 찾을 수 있도록 서블릿을 등록해주어야 한다. 이것을 등록해주는 방법이 크게 2가지가 있는데 첫 번째는 `web.xml` 을 이용하는 방법, 두 번째가 `@WebServlet` 어노테이션을 이용하는 방법이다. 이전에는 `web.xml` 을 통해서만 서블릿을 관리할 수 있었지만 서블릿 3.0부터 어노테이션을 활용하여 서블릿을 매핑해줄 수 있게 되었다.

1. `web.xml` 을 이용하는 방법

```xml
<servlet>
	<servlet-name>hello</servler-name>
    <servlet-class>Hello</servlet-class>
</servlet>
<Servlet-mapping>
	<servlet-name>hello</servlet-name>
    <url-pattern>/hello</url-pattern>
</Servlet-mapping>
```
{: file="web.xml"}

```java
import javax.servlet.*;
import javax.servlet.http.*;
import java.io.*;

public class Hello extends HttpServlet{
	public void hello(HttpServletRequest request, HttpServletResponse response) throws IOException, ServletException {
		...
	}
}
```
{: file="Hello.java"}

2. `@WebServlet` 어노테이션을 이용하는 방법

```java
import javax.servlet.*;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.*;
import java.io.*;

@WebServlet("/hello")
public class Hello extends HttpServlet {
  ...
}
```
{: file="Hello.java"}

### HttpServletRequest

- http 프로토콜의 request 정보를 서블릿에게 전달하기 위해 사용되는 클래스
- Header, Parameter, Cookie, URI, URL 등의 정보를 읽어오는 메소드를 가짐
- Body의 Stream을 읽어들이는 메소드를 가짐

### HttpServletResponse

- 서블릿은 HttpServletResponse 객체에 Content type, 응답코드, 응답 메세지 등을 담아 전송

### 서블릿 컨테이너 동작 방식

![Untitled 1](https://user-images.githubusercontent.com/80937237/150666128-10856c5c-c1d5-4c6e-ac10-02a23e8a0662.png)

1. 사용자가 http request를 보내면 이것이 서블릿 컨테이너로 전송된다. 
2. 해당 서블릿 컨테이너는 `HttpServletRequest`, `HttpServletResponse` 객체를 생성한다.
3. `web.xml` 을 이용하여 해당 request가 어떤 서블릿에 대한 요청인지 확인
4. 해당 서블릿에서 service 메소드를 호출하고 종류에 따라 `doGet()` , `doPost()` 등을 호출한다.
5. 메소드는 동적 페이지 생성 후 `HttpServletResponse` 객체에 응답을 보낸다.
6. 5번까지의 과정이 모두 끝나면 `HttpServletRequest`, `HttpServletResponse` 를 소멸시킨다.

> 참고: `HttpServletRequest` , `HttpServletResponse` 객체는 생성 후 소멸되지만 서블릿은 생성 후 소멸되지 않는 이유가 기본적으로 서블릿은 싱글톤으로 관리되기 때문이다.
> 

## JSP(Java Server Pages)

JSP는 Java 코드를 포함하고 있는 html 코드이다. 

> 주의: 서블릿은 Java 코드 속에 html 코드를 포함하고 있다. (정반대)
> 

Java 코드 속 html 코드를 작성하기 굉장히 불편하기 때문에 이를 보완하기 위해 나온 ‘서버 스크립트 기술’이다. 스크립트 기술이란 실행 시점에 약속에 따라 매핑된 키워드로 변환되어 코드가 실행되는 것을 말한다.  

JSP는 <% %> 태그 안에 Java 코드를 작성한다. 화면(View)의 구현이 서블릿에 비해 굉장히 쉽기 때문에 MVC 패턴에서 View 부분의 코드를 JSP, Controller 부분을 서블릿으로 작성하여 JSP와 서블릿을 동시에 사용하는 형태를 차용하였다.

![Untitled 2](https://user-images.githubusercontent.com/80937237/150666350-2d78d44c-35c0-46aa-b6a5-5e16375400cb.png)


<div align="center">MVC 구조에 대해서는 추후 포스팅에서 더 자세히 다루도록 하겠다.</div>
{: .small}
<br />

JSP 파일은 서블릿에 기반하고 있기 때문에 최종적으로는 서블릿으로 변환되어 실행된다. 이 과정은 한 번만 실행되기 때문에 요청이 많아져도 크게 성능이 떨어지지 않으며 JSP 파일에 변화가 있을 경우 그 때 다시 변환 과정을 거친다.

<u>종합하자면 JSP의 특징으로는 다음과 같다.</u>

- 개발 속도가 빠르다.
- 배우기 쉽다.
- View 관련 작성이 용이하다.
- Java 코드를 <% %> 태그 안에 처리한다.
