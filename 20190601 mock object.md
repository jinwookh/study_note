# mock object 블로그 글 번역

- 원문 : https://techblog.gmo-ap.jp/2017/03/13/mock%E3%81%A7%E3%83%A6%E3%83%8B%E3%83%83%E3%83%88%E3%83%86%E3%82%B9%E3%83%88%E3%82%92%E7%B0%A1%E5%8D%98%E3%81%AB%E3%81%97%E3%82%88%E3%81%86%EF%BC%81/
- 파파고를 많이(거의) 참조했습니다.
- 초급 일본어 실력으로 번역한 글입니다. 의역 및 오역이 있을 수 있습니다.
- 본문에서 번역하기 어려운 부분은 내용의 흐름을 해치지 않는 선에서 뺐습니다.

## 내용
## 시작하며 
안녕하세요, 모두들 테스트는 하고 계신가요?   
최근 개발방법을 보면, 거의 확실히 테스트가 고려되고 있기에 테스트 하기가 싫어지기도 합니다.   
테스트는 사실 꽤 어렵습니다.   
product code 에 따라서 테스트 코드 짜실 때 꽤 고생하실 수도 있어요.
    
그래서 이번에는 테스트에 초점을 맞춰, 테스트 코드를 쉽게 쓸 수 있는 mock을 이용하는 방법을 소개해 보려고 해요.    
저는 GMO MARS DMP에서 개발 및 운용을 담당하고 있고, 오늘 소개하는 내용은 보통 업무 때 실천하는 내용입니다.    
언어는 JAVA로, 프레임워크는 JUnit을 사용하겠습니다.

## unit test를 씁시다
우선 unit test를 쓰는 의의를 다시 생각해 봅시다. 본론으로 들어가기 전, 조금만 기다려 주세요.    
unit test를 씀으로써 가장 기대되는 점은, 역시 품질의 향상이겠죠.    
잘못된 부분을 빠르게 발견할 수 있고, 기존 기능을 변경할 때도 오류를 방지할 수 있어요.   
또한, product code를 refactoring 할 때도, unit test는 실수를 방지하는 방파제 역할을 하기에, 기능면 뿐만 아니라 코드 품질도 향상시킬 수 있어요.

그 외에도, 테스트 코드를 씀으로써 product code의 사용법을 명시할 수 있어요.    
즉, test 대상 class의 사용법 샘플로써 test code를 이용할 수 있습니다.   
test code는 실제 동작 가능하기에, 사용법 문서보다도 신뢰성이 높다고 할 수 있을 것입니다.   
(test code가 오래된 경우 test가 fail할 가능성이 높기 때문)
      
한층 더 나아가, unit test는 설계의 품질 향상에도 기여합니다. unit test의 용이함으로 설계의 품질 정도를 알 수 있기 때문이죠.    
이것에 대해서는, 본고에서 실감할 수 있으리라 생각합니다.

## unit test에서 자주 발생하는 문제
그런데, unit test가 중요함을 재확인했지만, 실제로 테스트 코드를 쓰는 일은 의외로 어렵습니다.   
테스트 대상 클래스 외부 리소스를 access 하는 처리 또는 실행하는 환경에 의존하는 처리가 있어,    
자기의 local 환경에서 test할 수 없는 경우가 다반사입니다.    
    
예를 들어, 이하와 같이 id로부터 유저 이름을 추출하는 간단한 테스트를 생각해봅시다.    
유자의 정보는 db에 넣어져 있기에, db에 접속해 데이터를 취득하는 처리가 필요합니다.     
```java
public class SomeService {
    // 지정된 user id list로부터 user 이름 list를 취득한다.
    public List<String> getActiveUsers(List<Integer> ids) {
        final List<String> userNames = new ArrayList<>();
        for (int id : ids) {
            userNames.add(getUserNameById(id));
        }
        return userNames;
    }
    private String getUserNameById(int id) {
        String name = "";
        //
        // db로부터 유저 정보를 취득하는 긴 처리
        //
        return name;
    }
}
```



