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
유저의 정보는 db에 넣어져 있기에, db에 접속해 데이터를 취득하는 처리가 필요합니다.     
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
이 클래스의 unit test를 하려면, 자신의 locl 환경에 db를 만들거나 db접속을 설정해야한다는 사실을 알 수 있습니다.   
또한, 실제 unit test를 실행시키면, db와의 접속이나 test data 투입에 꽤 많은 시간이 걸립니다.    

## Mock을 씁시다
이런 경우에, Mock을 쓰면 능히 해결할 수 있습니다.   
Mock은, 단순히 말해서 클래스의 동작을 simulate하기 위한 object입니다.   
test 대상 클래스가 호출하고 있는(의존하고 있는) 클래스를 Mock으로 바꿔, Mock의 동작 내용을 정의함으로써,     
원하는 테스트의 조건을 용이하게 만들 수 있습니다.    
    
Mock을 다루는 library는 각 언어마다 존재하고 있습니다만, Java언어에서는 Mockito library가 유명합니다.     
Mockito는 1 버전과 2 버전이 존재하지만, 여기에서는 1 버전을 다루겠습니다.
     
실제 아래와 같은 느낌으로 Mockito를 사용합니다.    
     
Product code는 다음과 같아요.    
```java
public class MockSample {
    private final SampleService sampleService;
 
    public MockSample(SampleService sampleService) {
        this.sampleService = sampleService;
    }
 
    public int validate(int id) {
        if (sampleService.isSomething(id)) return -1;
        return id;
    }
 
}
```
test code는 다음과 같아요.
```java
import org.junit.Test;
import static org.hamcrest.CoreMatchers.*;
import static org.mockito.Mockito.*;
// ～～생략
    @Test
    public void testSample() throws Exception {
        /* set up */
        SampleService sampleService = mock(SampleService.class);  // 8행
        MockSample suv = new MockSample(sampleService);
        // Mock의 동작을 정의
        when(sampleService.isSomething(100)).thenReturn(true);    // 11행
        
        /* execute */
        int actual = suv.validate(100);
 
        /* verify */
        assertThat(actual, is(-1));         //    17행
 
    }
```
test code의 8행에서 Mock object를 생성하고 있고, 11행에서 Mock의 동작을 정의하고 있어요.    
11행에 보면 'isSomething() method와 인수 100이 불려졌을 때 true를 리턴한다'는 동작이 정의되어 있어요.    
그리고 17행에서, isSomething(100)이 true를 리턴한다는 가정 하에 test 대상 method가 정확히 -1을 리턴하는지 테스트합니다.   
   
이와 같이, Mock를 사용하면 test 대상 클래스가 의존하는 class의 동작을 simulate하여,    
test case의 사전 조건을 용이하게 정돈할 수 있습니다.    

## Mock를 실제로 사용해 보면
그럼 아까 말한 SomeService class의 test code를 Mock을 이용해 작성해 보죠.    
당장 의존하고 있는 class를 Mock으로 바꿔 줍시다!....하지만 의존하는 클래스가 없군요.    
그렇다면 mock으로 바꿀 수가 없습니다.    
    
이런 경우는 특히 legacy code에서 자주 발생합니다.   
이하의 순서에 따라 product code를 수정하여, Mock으로 바꿀 수 있는 설계를 해 보아요.   

## 책임 외의 처리는 다른 클래스로 위임합시다
우선 DB에 access하고 있는 부분은 SomeService의 책임이 아니므로, 별도의 클래스로 위임해 버립시다.   
예로, 새롭게 위임한 class를 UserMapper class라고 합시다.
   
실제로 수정하면 이하와 같이 됩니다.    
```java
public class SomeService {
 
    // 지정한 id list로부터 user의 이름 list를 취득
    public List<String> getActiveUsers(List<Integer> ids) {
        final UserMapper userMapper = new UserMapper(); // DB access하는 부분을 별도의 class로 위임
        final List<String> userNames = new ArrayList<>();
        for (int id : ids) {
            userNames.add(userMapper.getUserNameById(id));
        }
        return userNames;
    }
}
```
이것으로 귀찮은 DB access하는 부분은 별도의 class에 있게 됬으므로, 간단히 테스트를 쓸 수 있....을 수 없군요.    
이렇게 UserMapper를 생성하는 부분이 클래스 안에 있게 되면, UserMapper 인스턴스화시 결국 DB로 접근하게 되어 Mock으로 UserMapper를 대체할 수 없습니다.   

## 의존하는 object는 외부로부터 주입되도록 합시다
그렇다면 UserMapper object를 인스턴스화하는 부분을 클래스 외부로 빼 버립시다.    
즉, SomeService를 사용하는 측에서 UserMapper를 인스턴스화하여, 그것을 SomeService에 주입해 주기로 해요. 생성자를 사용하여 주입해도 좋아요.    
```java
public class SomeService {
    private final UserMapper userMapper;

    public SomeService(UserMapper userMapper) {
        this.userMapper = userMapper; // 외부로부터 이미 생성된 인스턴스를 Someservice 생성자를 통해 주입
    }

    // 지정된 id list로부터 이름 list를 취득
    public List<String> getActiveUsers(List<Integer> ids) {
        final List<String> userNames = new ArrayList<>();
        for (int id : ids) {
            userNames.add(userMapper.getUserNameById(id));
        }
        return userNames;
    }
}
```
겨우 할 수 있었네요. 이제 test하기 곤란한 부분(db로 접근)을 Mock으로 대체할 수 있습니다.   

## Mock을 이용해 unit test를 간단히 써요







