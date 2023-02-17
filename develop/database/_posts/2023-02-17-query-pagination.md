---
layout: post
title: 🪶 쿼리 Pagination 사용하기
sitemap: false
hide_last_modified: true
---

개발을 하다 보면 디비에서 데이터를 특정 인덱스부터 지정한 Row count만큼 데이터를 리턴해줘야 할 일이 있죠?<br/>
우리는 이것을 Pagination이라고 부르죠.<br/>

보통 백엔드 영역에서 쿼리를 통해 Pagination을 구현할 때 우리는 아래의 2가지의 데이터를 활용합니다.
  > 한 페이지에 보여줄 데이터의 개수(rowCount), 전체 중 현재 몇 페이지를 보여줄건지(currentPage)

이 데이터를 사용 해 과연 Pagination을 구현하기 위해 쿼리는 어떻게 작성하는지,
특히 대표적으로 사용되는 MySQL과 MSSQL에서의 작성법을 정리 해 볼게요 😇

<!-- * toc -->
<!-- {:toc} -->

## 📖 MySQL에서의 Pagination

MySQL에는 <span style="color: lightgray">~~눈부시도록 간편한~~</span> `LIMIT`라는 키워드를 활용합니다.<br/>
`LIMIT` 키워드는 선택된 레코드 중 지정한 RowCount만큼만 쿼리 결과로 리턴 해 줍니다.

대표적으로 아래와 같은 방식으로 사용할 수 있죠.

~~~sql
SELECT * FROM TestTable WHERE (1=1) ORDER BY ID ASC LIMIT 30;
~~~

Pagination을 위해 한 페이지에 보여줄 레코드는 30개로 설정했다! 라는 가정을 세워볼까요?

그러면 위 쿼리를 적용하면 상위 30개의 레코드만 가져온다고 했으니 일단 절반은 성공한 것 같아요.

그런데 우리가 적용하고자 하는 Pagination은 상위 30개만 가져올게 아니라,<br/>
1페이지를 선택 했을 땐 1번부터 30번까지, 2페이지를 선택했을 땐 31번부터 60번까지, ... 의 형태로 쿼리 결과가 리턴되어야겠죠?

다행히(?) LIMIT에는 OFFSET 옵션을 줄 수 있어요.<br/>

예를 들면 이런 식이죠.

~~~sql
SELECT * FROM TestTable WHERE (1=1) ORDER BY ID ASC LIMIT 0, 30;
~~~

위 쿼리와 방금 짠 쿼리의 차이점이 보이시나요?<br/>

`LIMIT` 뒤에 파라미터를 위와 같이(LIMIT 30) 주면 무조건 <span style="background-color: #fff5b1">정렬된 결과의 상위 30개만</span> 가져오고,<br/>
아래와 같이 (LIMIT 0, 30) 주면 <span style="background-color: #fff5b1">정렬된 결과에서 0개의 레코드를 지나친 후 30개의 레코드를 가져온다</span>는 의미에요.

슬슬 느낌이 오시죠?<br/>
이 offset을 적용하여 쿼리를 다시 작성 해 볼까요?

~~~sql
SELECT * FROM TestTable WHERE (1=1) ORDER BY ID ASC LIMIT 0, 30; -- 0번 인덱스부터 30개를 가져온다.
SELECT * FROM TestTable WHERE (1=1) ORDER BY ID ASC LIMIT 30, 30; -- 30번 인덱스부터 30개를 가져온다.
~~~


## 📖 MSSQL에서의 Pagination

MSSQL에서 MySQL LIMIT 30의 효과와 같은 쿼리는 아래와 같이 작성합니다.

~~~sql
SELECT TOP 30 * FROM TestTable WHERE (1=1) ORDER BY ID ASC;
~~~

생각보다 별거 없네~ 하고 생각했는데 페이징은 또 이야기가 좀 달라요 😇

MySQL 쿼리에서 세웠던 가정 그대로! 한 페이지에 보여줄 레코드는 30개로 할래요~ 했을 때,<br/>
첫 번째에 해당하는 30개의 레코드를 가져오는 쿼리는 아래와 같습니다.

~~~sql
SELECT * FROM TestTable WHERE (1=1) ORDER BY ID ASC OFFSET 0 ROW FETCH NEXT 30 ROWS ONLY;
~~~

<span style="color: lightgray">~~저도 처음 봤을 땐 읭? 했어요~~</span><br/>
중요하게 눈여겨 봐야 할 곳은 `OFFSET` 이하 쿼리예요.

MySQL에서 봤던 LIMIT 0, 30과 비교 해 보면 훨씬 길지만 의외로(?) 이해하는데 어렵지는 않을 것 같아요.

말 그대로 OFFSET 이하 쿼리를 문장 그대로 해석 해 보자면,
> OFFSET을 0만큼 밀고, 그 밀린 데이터에서 상위 30개만 가져올래요~

라는 뜻이에요.

아! 그러면 그 다음 페이지는 페이지에서 보여줄 레코드의 개수만큼 오프셋을 밀어줘야 하니까,<br/>
다음 페이지의 쿼리는 이런식으로 작성되면 되겠네요?

~~~sql
SELECT * FROM TestTable WHERE (1=1) ORDER BY ID ASC OFFSET 30 ROW FETCH NEXT 30 ROWS ONLY;
~~~

## 💁🏻 결론!

~~~cs
public QueryResponse QueryPagination(string DBType, int CurrentPage, int PageLimit, string additionalCondition)
{
  string whereClause = additionalCondition;   // WHERE 절 뒤에 붙을 임의의 조건절 조합
  string query = string.Empty;
  switch(DBType)
  {
    case "MySQL":
      query = string.Format(@"SELECT * FROM TestTable WHERE (1=1) {0} ORDER BY ID ASC LIMIT {1},{2}", 
        whereClause, (currentPage-1)*PageLimit, PageLimit);
      break;
    case "MSSQL":
      query = string.Format(@"SELECT * FROM TestTable WHERE (1=1) AND {0} ORDER BY ID ASC OFFSET {1} ROWS FETCH NEXT {2} ROWS ONLY", 
        whereClause, (CurrentPage-1)*PageLimit, PageLimit);
      break;
  }
  return new QueryResponse(RunQuery(query));
}
~~~

<br/>
오늘은 가볍게! 대표적으로 사용되는 MySQL과 MSSQL에서 Pagination 쿼리는 어떻게 작성하는가에 대해 정리 해 보았습니다.<br/><br/>
그럼, 다음 글에서 또 뵈어요 :) 🪶

