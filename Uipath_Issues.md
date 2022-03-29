# 특이사항
알고 싶지 않았으나 알게된 정보를 정리합니다.   

#### UiPath 라이브러리 올릴 떄 참고
```yaml
주석관련 : 
  - Activity에 커서올리면 바로 볼 수 있는 주석 설정하기 :
    - 방법 : Project패널 > 해당 xaml 우클릭 > Properties > tooltip 설정 > Save
    - 비고 : 최상위 activity에 주석 넣어놓으면, tooltip 설정할 때 자동으로 주석 내용을 넣어준다.
             tooltip 을 적용하면 xaml에 <sap2010:Annotation.AnnotationText> 항목이 생긴다. 
에러관련 : 
  - 배포 후 NameSpace 컴파일 에러 : 
    - Invoke Code에서 liblaray 함수/객체 명을 strict하게 쓰지 않았을 때 주로 생김 (특힉 workbook)
    - 다른 라이브러리와 객체/함수 명이 같아 생기는 오류. 함수/객체 호출 시 System부터 쭉 경로 다 써줘야 예방 가능
  - 배포 후 잘못된 형식 관련 에러 : 
    - Library 안에서 State Machine 사용했을 떄, 트랜젝션 Triger에 Element Exist 등 넣어두면 해당 오류 생김.
    - 정확한 원인은 모르겠으나. 트렌젝션에서 처리하는 내용을 entry나 exist 마지막 부분으로 이동시키면 해결 가능.
```

#### Kill Process 액티비티
```yaml
Description : 
  UiPath.Core.System.Activity 버전이 18.4 -> 19.4 로 변경될 때 kill process의 대상이
  이미 꺼져있는 경우(process 목록에서 찾지 못한 경우) 에러를 발생하도록 변경되었다.
ToDo:
  이전과제에 적용된 Kill Process를 모두 찾아 ContinueOnError=True를 적용하여 해결
After : 
  kill process 를 Uipath 액티비티로 사용하기 보다는 Custom Activity나 library를 만들어 사용하는 게 좋을 것
```

#### IE 브라우저 Edge로 셀렉터 변환 
```yaml
Description : 
  텍스트 에디터로 xaml에서 바로 Replace 시켜버릴 것
Before:
  - "&lt;html title
  - "&lt;html html
  - app='iexplore.exe'
  - BrowserType="IE"
After:
  - "&lt;html app='msedge.exe' title
  - "&lt;html app='msedge.exe' html
  - app='msedge.exe'
  - BrowserType="Edge"
Target:
  - app이 명시되지 않은 UI 셀렉터
  - app이 명시되지 않은 UI 셀렉터
  - app이 IE로 명시된 UI셀렉터
  - Open, Attach Browser에 콤보박스 변경
```


## 1. [시작] 메뉴 패널 안열릴 떄 window app 사용하기
시작버튼, window 키를 눌러도 패널이 열리지 않을 때 Window 기본 app 사용 방법  
윈도우 탐색기가 열린다면 시스템 경로 이동하여 수동으로 app 실행이 가능합니다.  

| 이름 | 경로,이름 | 
|:---:|---|
경로이동 | C:\Windows\System32
원격접속 | mstsc.exe
그림판 | mspaint.exe

## 2. System32 경로 내 존재하지 않는 앱이 System32 경로에서 실행되는 경우
| 폴더명 | 경로 위치 | 
|:---:|---|
System32 | C:\Windows\System32
SysWOW64 | C:\Windows\SysWOW64

###### 출처: [몽키의 IT개발 노트](https://pung77.tistory.com/23) 
```txt
32bit 프로세스는 SysWOW64 시스템 폴더의 내용을 참조하여 실행된다.
64bit 프로세스는 System 32 시스템 폴더의 내용을 참조하여 실행된다.

Windows는 File System Redirector라는 기능을 지원하여,   
Sytem32 폴더에 접근하여 사용하려 하여도 SysWOW64 폴더로 리다이렉트 시켜 자동으로 SysWOW64 폴더의 내용을 참조한다.   
https://msdn.microsoft.com/ko-kr/library/windows/desktop/aa384187(v=vs.85).aspx   

예를들어 64bit os에서 
32bit 프로세스가  LoadLibrary(C:\windows\System32\Kernel32.dll) 을 호출하여 Kernel32.dll을 로딩하려 하여도 
실제로는 리다이렉트되어 C:\windows\SysWOW64\Kernel32.dll 경로의 Kernel32.dll을 참조한다.

32bit 프로세스가 System32 폴더에 접근하고 싶다면 Wow64EnableWow64FsRedirection API를 사용해 리다이렉트 기능을 끄고 강제로 접근하면된다.  
https://msdn.microsoft.com/ko-kr/library/windows/desktop/aa365744(v=vs.85).aspx
```


## 3. 동일한 문자열이 고장날 떄
눈에 보이지 않는 공백문자가 포함되어 있을 수 있다.  
글자의 가로 길이가 0인 특수문자가 포함되어 있는 문자다.  

```vb
A = "C:\Users\UserName\Desktop\tmp\test.xlsx"
B = "‪C:\Users\UserName\Desktop\tmp\test.xlsx"

System.IO.File.Exists(A) | True | window 탐색기 주소창에서 복사
System.IO.File.Exists(B) | False| 해당 파일 속성>보안>개체이름 에서 왼쪽에서 오른쪽으로 드레그하여 복사

A.Length | 39
B.Length | 40 | 글자의 폭이 0 인 문자가 끼어있음

VScode에 복사 붙여넣기 하면 명확한데,
A | "C:\Users\UserName\Desktop\tmp\test.xlsx"
B | "[U+202A]C:\Users\UserName\Desktop\tmp\test.xlsx"
로 표시된다. 

다음과 같은 Linq로 보완로직을 만들면 해당 이슈를 예방할 수 있다. 아스키 63번은  아스키 코드에 등록되지 않은 ? 문자이다. 
B = join(B.where(function(x) asc(x)<>63).Select(function(x) x.ToString).ToArray,"")
```


## 4. Edge에서 간헐적으로 팝업을 못잡을 떄
최상의 셀렉터를 설정할 떄 ``` idx='*' ``` 를 명시해야 여러 Wndow를 동시에 확인한다.   
셀렉터가 모호할 때, 가능한 최대 경우의 수에 대해 자동으로 탐색해주지 않는다고 생각하는 게 이롭다.   

```xml
<!-- 가장 앞서 포커스된 Edge에 대해 동작함 -->
<wnd app='msedge.exe'/>

<!-- Process에 열려있는 모든 Edge Window를 살펴보나봄 -->
<wnd app='msedge.exe' idx='*' />
```
## 5. cmd로 Uipath 실행 관련
[스케줄러 참고](https://deje0ng.tistory.com/78)
[UiPath 문서 참고](https://docs.uipath.com/robot/docs/arguments-description)
```cmd 
# 해당 방법은 Uipath 2019.10 버전 이하에서 작동했던 방법입니다.
# 2020 이후 버전부터는 Uipath Assistant를 경유해서 동작하도록 업데이트 되었습니다.
# Ubot 라이센스를 구매하지 않고 MS스케줄러를 사용하는 편법을 막기 위한 조치로 보입니다. 

# KS출근.bat
cd "<C:\Program Files (x86)\UiPath\Studio\>"
.\UiRobot.exe execute  -process "KS출근" -input "{ 'str_workTime' : '8-17' , 'str_ID' : 'wbpark' }"
```

##### Invoke Code 사용할 경우 MethodName 의 경우 대소문자를 구분 필수
"Add"로 써야할 것을 "add"로 쓸 경우 에러가 발생한다.