---
layout: post
title: 🪶 Null 전파의 사용
sitemap: false
hide_last_modified: true
---
<!---->
## 📖 Intro
아래의 코드를 한 번 살펴 볼까요?

먼저, 아래와 같이 데이터베이스 서버에서 쿼리 결과를 가져오는 임의의 함수가 있다고 가정해 봅시다.

~~~csharp
public List<T> Query(string query)
{
    bool success = false;
    var queryResponse = null;

    // ... do something to get queryResponse ...

    if (!success)       // 쿼리에 실패하였다면
        return null;    // null을 리턴하고
    else                // 쿼리에 성공하였다면
        return BuildModelListResult(queryResponse);     // 쿼리 결과를 List로 응답
}
~~~

그리고 위 함수를 사용하여 또 다른 임의의 API 바디를 작성했다고 가정해 봅시다.<br/>
이 API는 DB에 있는 `Employee`라는 테이블의 전체 정보를 쿼리 해 온 다음,<br/>
Level 필드가 3보다 큰 데이터가 1개 이상 존재한다면 API 응답으로 포함된 bool 필드를 다르게 리턴해주는 API입니다.<br/>
아래와 같이 말이죠.

~~~csharp
public ResponseApi Foo(RequestApi request)
{
    ResponseApi response = new ResponseApi();
    string query = "SELECT * FROM Employeeee";
    response.Data = Query(query);

    if (response.Data.Where(x=>x.Level >= 3).Count() > 0)
        response.IsLevel3 = true;
    else
        response.IsLevel3 = false;

    return response;
}
~~~

자, 그런데 문제가 있군요.<br/>
우리가 쿼리해오려는 타겟 테이블의 이름은 <b>Employee</b>인데, API 바디 안에 작성된 query 변수에 작성된 쿼리는 <b>Employeeee</b>라고 잘못 작성되었네요.<br/>
이 상태로 API 테스트를 했을 때 예상되는 결과는 무엇일까요?<br/>

DB서버에는 <b>Employeeee</b>라는 테이블은 존재하지 않습니다.
그런데 우리가 사용한 함수 Query를 다시 보면, 쿼리에 실패 했다면 null을 리턴하도록 작성되어 있군요.
즉, 서버에 존재하지 않는 테이블에 대해 쿼리를 시도하였으니 response.Data 필드에는 null이 할당 되겠군요.
문제는 그 다음입니다.
이 상태에서 response.Data 필드에 대해 Linq를 시도하면 어떻게 될까요?
null object에 대해 Linq를 시도하려 했기 때문에 NullReferenceException이 발생하게 됩니다.

## 📖 Concepts
그러면 저 API 바디를 조금만 더 고도화 해 볼까요?
일단 아래처럼 null이 응답으로 들어왔을 경우를 고려한 예외처리를 해볼 수 있겠네요.

~~~csharp
public ResponseApi Foo(RequestApi request)
{
    ResponseApi response = new ResponseApi();
    string query = "SELECT * FROM Employeeee";
    response.Data = Query(query);

    if (response.Data != null)
    {
        if (response.Data.Where(x=>x.Level >= 3).Count() > 0)
            response.IsLevel3 = true;
        else
            response.IsLevel3 = false;

        response.Success = true;
    }
    else
        response.Success = false;

    return response;
}
~~~

애초에 Linq에 진입하기 전에 Query 함수 결과가 null이라면 응답의 Success 필드를 false로 지정함으로써 API 동작에 실패하였다는 상태값으로 리턴하도록 해 줬습니다.
그런데 코드가 제법 길어져서 영 보기 좋지가 않습니다. 좀 더 깔끔하게 정리가 되면 좋겠다는 생각이 듭니다.

그럼 이런 방법은 어떨까요?

~~~csharp
public ResponseApi Foo(RequestApi request)
{
    ResponseApi response = new ResponseApi();
    string query = "SELECT * FROM Employeeee";
    response.Data = Query(query);

    try
    {
        if (response.Data?.Where(x=>x.Level >= 3).Count() > 0)
            response.IsLevel3 = true;
        else
            response.IsLevel3 = false;

        response.Success = true;
    }
    catch (Exception ex)
    {
        response.Success = false;
    }

    return response;
}
~~~

이렇게 try-catch를 사용하여 NullReferenceException 이외에 우리가 예상하지 못한 예외가 발생하더라도 후처리를 할 수 있는 장치를 마련해 줬습니다.
그리고, List Linq를 사용하는 부분을 주목해 보면, 갑자기 `?` 기호가 붙었음을 혹시 보셨나요?
이 기호가 우리가 오늘 짚고 넘어갈 Null 전파입니다.

## 📖 Null 전파란?
기본적으로 3항 연산에 대한 계산식을 간단하게 줄여주는 문법입니다.
코드를 간략하게 작성하는데 유용하게 사용할 수 있죠.

위의 API 예제보다 더 간편하게 줄인 아래의 코드를 보면,
~~~csharp
TestClass A = null;
TestClass B = new TestClass();

TestClass C;
if (A == null)
    C = B;
else
    C = A;

C = A ?? B;
~~~

TestClass C에 새 값을 할당할건데, A가 null이면 B를 할당하고 A가 null이 아니면 A를 할당해라-
라는 의미의 코드입니다.<br/>
굳이 if-else 구문을 사용하지 않더라도 `C = A ?? B` 한 줄로 축약하여 사용할 수 있습니다.

3항 연산 뿐 아니라 접근하고자 하는 데이터가 null인 경우에 대비하기 위한 처리도 할 수 있습니다.<br/>
`if (response.Data?.Where(x=>x.Level >= 3).Count() > 0)` 와 같은 처리를 다시 보면,
response.Data가 null이 아닌 경우에는 Where Linq가 동작하여 조건절을 실행합니다.<br/>
반대로 response.Data가 null이라면 하위 Linq까지 동작하지 않습니다.<br/>
아예 하위 카운트 값을 0으로 반환시켜 버리죠.<br/>

즉, 조건에 맞는 데이터가 하나도 없다는 취급을 하기 때문에 NullReferenceException을 반환하지 않습니다.<br/>

## 📖 List 타입에만 사용 가능한가요?

아니요! null을 가질 수 있는 일반 변수에 대해서도 사용 가능한 문법입니다.<br/>
아래의 예시로 간단하게 정리 해 볼까요?

~~~csharp
string str1 = null;
int result = -1;

if (str1 != null && str1.Length > 0)        // str1이 null이 아니면서 길이가 0보다 큰가?
    result = str1.Length;
else
    result = 0;

// 위의 if문은 아래의 식으로 축약 사용 가능하다.
result = str1?.Length > 0 ? str1.Length : 0;
~~~

## 💁🏻 결론!

물론 예상 가능한 오류에 대해 개발자들이 미리 처리를 해 두는 것이 가장 좋은 방법이겠으나,<br/>
막상 코드를 짜다 보면 의외로 생각지도 못한 부분에서 오류가 발생하는 경우가 꽤나 많잖아요? 😇<br/>

오늘의 키포인트는 null일 가능성이 있는 데이터에 대한 처리였습니다.<br/>
코드를 짜다보면 심심찮게 의도치 않은 null을 리턴받는 경우가 더러 있더라고요.<br/>
nullable 타입에 대한 처리에 대해 좀 더 신경써서 오류 가능성도 줄이고 짧아진 코드로 가독성도 챙기면 좋을 것 같아 null 전파에 대해 정리 해 보았습니다.<br/>

그럼, 다음 글에서 또 뵈어요 :) 🪶