Tapestry가 고안된 이유는 감당하기 어려울 정도로 많은 메일들을 받기 때문임. 많은 양의 메일을 다루는 방법에는 여러가지가 있음. 1) 유저가 구독한 메일만 받는 방법. 그치만 깔끔하게 받아지기는 어려운 방법. 2) 유저가 구체적인 필터를 만드는 방법. 이미 몇몇 메일 시스템은 이 방법을 적용중임. Tapestry의 목적은 이러한 필터링 방법을 더 효과적으로 하는데 있음.

컨텐츠 기반 필터링뿐만 아니라 Tapestry는 협업 필터링도 지원함. 협업 필터링은 유저간의 상호작용을 비교하는 방법. 보통 annotations라고 불리는 유저가 재미있다 또는 재미없다 라고한 반응들은 다른 필터에서 이용할 수 있음.

Tapestry는 내가 모든 문서를 볼 시간도 없고 검색은 하고 싶지만 뭐라고 검색해야 할 지 모를 때 Smith, Jones, O'Brien이 응답한 문서라고 추천해줄 수 있음.

협업 필터링의 경우 두 개 이상의 텍스트들(메세지와 이에 대한 답장 또는 문서와 피드백)이 관련있다는 점에서 새롭다. 현재의 필터링 시스템과 달리 Tapestry는 단순히 문서가 도착했을 때 검사하는 게 아니라 반복적으로 전체적인 문서들을 검사함. 언젠가 스미스가 어떤 문서를 읽고 재미있다 평가하면 그 때  나도 

Tapestry는 메일 시스템 그 이상의 역할이 가능함. 또, 단순히 필터링 뿐만 아니라 저장소 역할도 함. 유저는 원하는 문서를 찾으려고 할 때는 키워드로 검색을 할 것임. 이 때, 매우 많은 문서가 검색되므로 키워드를 하나 더 설정하거나 최소 3명이 평가한 문서를 찾는 것이 더 효과적임을 알것임.

### Architecture
Tapestry는 Indexer, Documnet Store, Annotation store, Filterer, Little box, Remailer, Appraiser, Reader/Browser로 구성됨.

* Indexer : 메일이나 뉴스기사같은 외부문서를 가져와 쿼리를 가지고 참고할 수 있도록 인덱싱 하는 역할
* Document Store : 저장소 역할을 하는데, 쿼리가 효율적으로 수행될 수 있도록 함. 데이터 추가만 가능
* Annotation store : anootation 저장소 역할. 데이터 추가만 가능.
* Filterer : 유저의 쿼리랑 매칭되는 문서들을 배치만큼씩 반복적으로 찾음
* Little box : 유저가 관심있는 문서들을 담아둠. 각각의 유저는 이 박스가 있음
* Remailer : 주기적으로 little box에 문서들을 유저한테 보내줌
* Appraiser : 유저의 문서들을 개인화해서 분류해줌. 자동으로 우선순위를 결정하고 분류함
* Reader/Browser : 인터페이스 제공. 필터를 추가, 삭제 수정하거나, 새 문서를 받거나, 보거나 모으거나, 반응을 달 수 있음

대부분의 Tapestry 구조는 협업 필터링을 제공하는 데 목적이 있음. documnets끼리 비교하려면 documnet store가 있어야 하고 유저의 평점과 비교하려면 annotations가 있어야 함.


### Document and Annotation Stores
이상적으로는 Tapestry는 documents를 영구 보관할 것임. annotations은 분리되서 보관되지만 관련 문서와 연결은 되어있음. 한곳에 두면 좋겠지만 그러지 못하는 이유가 있음.
* doc이 저장되고 이후에 anno가 저장되는데 (문서는 append-only만 가능하므로) doc 뒤에 anno를 추가하면 doc의 불변성을 해침
* 복잡한 anno와 간단한 anno가 있는데 복잡한 anno의 경우 붙여놓으면 query로 불러오기 어려운 점이 있음. Annotations 부분에서 또 설명하겠음

### Appraisers
Tapestry 유저는 단순히 관심있음 없음을 필터링하는 것보다 우선순위에 맞게 필터링 하는 것을 원함. 이를 위해서 Appraisers가 있음. 직관적인 작동 방식은 아님. 매 유저마다 appreaise를 작동시키는게 제일 간단한 방법 같지만, 백명의 유저가 많은 쿼리를 많은 doc에다가 매분마다 수행하는 것은 계산적으로 결함이 있는 방법임. 그래서 Tapestry는 다음과 같은 두 단계로 진행함.
1) 일단, 관심있는지 없는지부터 구분을 함. 관심있는 문서는 little box로 이동됨
2) 그래서, little box에 있는 문서만 필터링 함. 그래서 복잡한 appraiser를 지원할 수 있음

### Browsers
Tapestry의 browser는 mail reader랑 기존의 doc browser를 합친 구조임. mail reader가 하던, 새 메일을 little box로 보내고 유저가 이를 관리할 수 있게함. 또, 주기적으로 appraiser를 작동시킴. 기존의 메일 시스템은 메일을 복사해서 가지고 있기 마련인데, Tapestry는 pointer만 가지고 있어서 불변성을 유지함. 유저가 메세지를 지워도 사실 지워진 것은 아니며 언제든 복구할 수 있음. 또, 유저가 문서를 읽었는지 안읽었는지 또는 어떤 폴더에 저장되어 있는지 같은 정보를 저장하는 private filed가 있음. 이는 filter query로는 수정이 불가하고 시스템 쿼리(=ad hoc)로 수정 가능

### TQL, Tapestry Query lANGUAGE
Tapestry는 문서를 구체적인 쿼리로 필터링함. 따라서 이러한 쿼리는 중요한 요소 중 하나임. 제일 괜찮은 방법은 흔히 사용되는 SQL언어를 선택하는 것임. 이를 이용하면 구현에 있어서도 간단해지기 때문에 장점이 있음. 그치만 그렇게 하지 않은 두 가지 이유가 있음
1) 관계형 모델(=SQL)과 Tapestry 모델에는 차이가 있음. Tapestry는 문서의 field set들이 확장성있는 반면 SQL은 고정적임. 그리고, SQL은 sets을 지원하지 않는데 doc files의 경우는 거의 set-value임
2) 유저가 쉽게 ad hoc 쿼리를 사용하게 하고 싶은데 SQL 상용구 쿼리는 어려움
그래서 쿼리를 위해 독자적인 TQL 언어가 고안됨

### Basic Examples
TQL 쿼리는 1차 술어 논리랑 비슷하지만 sets을 지원한다는 점에서 다름. 가장 간단한 쿼리는 =나 <와 같은 기본 연산을 사용하는 것. m.to = {'Joe', LIKE '%Bi11%'} 와 같은 쿼리(=m.to 라는 field에 Joe가 들어가거나 Bill이 포함되는 문서)가 가능하며 EXISTS (ml: ml.sender = 'Joe' AND ml.in-reply-to = {m}) 와 같은 협업 쿼리(=Joe가 답변한 모든 문서 m)가 가능. 또, 다른 유저의 필터도 사용할 수 있음. m IN Terry.Baseball AND re.words = {'Dodgers'}. 테리의 베이스볼 쿼리에 포함되면서 m.words가 Dodgers인 문서

### Annotations
지금까지 설명한 TQL은 한번에 결정되는 것이 아니고 전자 문서의 형태를 가진 쿼리 언어를 통해 결정. 그래서 anno를 다루는 것이 직관적이지 않을 수 있음.  이전 섹션에서 말한 것처럼 anno는 doc 데이터와 같이 저장되지 않음. 그렇지만 이것이 별개의 필드로 저장되는 느낌은 아님. 오히려 우선순위 같은 anno를 매우 자연스럽게 표현하는 구조. 예를 들어서 'm.a.priority' 라는 건 doc의 우선순위에 접근하기 위함이며 이 때 a는 annotation의 또다른 네이밍임. 비슷하게 폴더에 접근할 때도 'm.a.folders'로 접근할 수 있음.

협업 필터링에 사용되는 복잡한 anno에 대해서는 깔끔하게 작성하기는 어려움. 만약에 필드를 추가해서 voting 시스템을 구현한다고 해보면 vote라는 anno가 있을건데, 이러한 anno는 자기만의 구성이 있음. voter는 누구고, 뭘 vote 했는지에 관한 부분 등. weiser가 vote한 message를 찾아달란 쿼리가 있다고 하면 m.a.vote는 v.owner=weiser 인 v라는 멤버 변수를 가져야 하고 이러한 방식에서 집합 표기방식을 확장할 필요가 있음.

이 쿼리를 TQL로 표현하면 다음과 같음
```
a.type = 'vote'
AND a.owner = 'weiser'
AND a.msg = m
```

anno에는 doc이랑 연결되기 위한 msg 필드가 항상 있기 때문에 협업 필터링을 지원하는 이러한 쿼리들은 좀 더 간단하게 표현할 수 있음. 이전에 언급한 EXISTS를 이용하면 됨. 이전 쿼리는 EXISTS가 암묵적으로 생략된 것이고 다음과 같이도 쓸 수 있음
```
EXISTS (a: a.type = 'vote'
AND a.owner = 'weiser'
AND a.msg = m) 
```

이러한 표기 방식은 우선순위가 10인 문서 라는 간단한 문장도 다소 복잡하게 표현하게 된다.
```
a.type = 'priority' AND a.value = 10
AND a.msg = m
```

그렇지만 Tapestry가 협업 필터링을 지원하는 것이 목적이므로 이러한 분리형 annotation 방식이 더 적합하다고 판단했음.

### Filter Queries








