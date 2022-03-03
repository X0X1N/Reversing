# Reversing

##### 작성자 : xo_xin 

## topic = Exercise: Helloworld



#### Introduction


이 주제를 선택한 이유는 앞서 정적 분석과 동적 분석의 개념에 대해 공부를 했었고 이를 가지고 더 자세히 공부하고자 선택하게 되었습니다. 드림핵을 사용해 공부하는데 도움을 받았고 IDA를 사용하였습니다. 


#### Body

**정적 분석**

먼저 정적분석을 이용하여 살펴보도록 하겠습니다.
Dream hack에 올라와있는 Hello world.exe 파일을 다운 받습니다.

<img width="111" alt="image" src="https://user-images.githubusercontent.com/78540248/156380891-e2018eba-077c-4db1-b4ce-90c47088cbbc.png">



그 후에 IDA를 사용하여 다운받은 파일을 열고 아무것도 설정하지 않은 기본 상태에서 OK를 
눌러 실행합니다. 


<img width="271" alt="image" src="https://user-images.githubusercontent.com/78540248/156380743-842c6634-bf31-468d-b8a5-ae03debeb1a7.png">

main 함수를 찾아볼건데 진입점부터 main 함수까지 도달하는 경로를 정적으로 분석해보도록 하겠습니다. 


 
 <img width="261" alt="image" src="https://user-images.githubusercontent.com/78540248/156381965-fd5035c1-3a05-48ab-94bd-679945daf74d.png">






실행을 하고 Shift + F12를 눌러 바이너리에 포함괸 문자열이 열거된 Strings 창을 열어 많은 문자열 중 'Hello, world!'라는 문자열을 찾습니다. 컴파일 과정에서 다양한 문자열이 
바이너리에 추가되는데 이 문자열은 프로그래머가 넣었을 것으로 추측되고 위에서 찾은 'Hello world'라는 문자열을 더블 클릭하여 따라 들어갑니다.      



<img width="216" alt="image" src="https://user-images.githubusercontent.com/78540248/156382260-468e9d47-2ded-439d-8503-297550adcc0f.png">

많은 정적 분석 도구들은 상호 참조(Cross Reference, XRef) 라는 기능을 통해 이를 지원하는데 앞에서 찾은 "Hello world!\n"라는 문자열이 어디서 사용되는지 추적을 해보았습니다.

위 사진에 노란색으로 표시되어있는 "aHelloWorld" 를 클릭하고 Command + X (윈도우는 control + X)를 누르면 
와 같이 xrefs(cross reference) 창이 나타나면 OK를 눌러 실행합니다. 
이 창에는 해당 변수를 참조하는 모든 주소가 출력됩니다.  첫 번쨰 항목을 더블클릭하여 이를 따라가면 main 함수를 찾을 수 있습니다. 




<img width="270" alt="image" src="https://user-images.githubusercontent.com/78540248/156382507-82006b4b-3bbd-4716-bb25-aa58452e2db7.png">


















<img width="350" alt="image" src="https://user-images.githubusercontent.com/78540248/156383259-8095fc7a-d143-496f-aabd-77c0613cf947.png">









 < main 함수 >
 
 
 
 **main 함수를 찾는 일반적인 방법 **
 
main 함수는 C계열 언어에서 프로그래머가 작성한 코드 중 가장 먼저 실행되는 함수 입니다.  리버싱을 처음 공부할 때는 프로그램에서 가장 먼저 실행되는 코드가 main 함수의 코드라고 생각하기 쉽습니다. 그러나 사실 운영체제는 바이너리를 실행할 떄, 바이너리에 명시된 진입점부터 프로그램을 실행합니다.  진입점이 main 함수인제 불가능한 것은 아니지만, 일반적으로는 그렇지 않습니다. 
진입점과 main함수의 사이를 채우는 것은 컴파일러의 몫입니다. 대부분의 컴파일러는 둘 사이에 여러 함수를 삽입하여 바이너리가 실행될 환경을 먼저 구성하고, 그 뒤에 main 함수가호출되게 합니다. 
진입점에 위치한 함수를 start 함수라고 하고 main 함수를 쉽게 찾고 싶다면, 컴파일러가 작성하는 start 함수에 익숙해져야 합니다. start 함수를 여러번 분석하다 보면 나중에는 start 함수를 보고, main 함수가 어디서 호출될지를 쉽게 찾아 낼 수 있습니다.

  이제 main 함수 분석을 해보면
  
  <img width="222" alt="image" src="https://user-images.githubusercontent.com/78540248/156384170-3c12f0c8-0cd9-436e-b43e-b05cd2779ea5.png">


F5를 눌러 이를 디컴파일 합니다.  디컴파일된 코드를 통해 함수의 주요 정보를 살펴보면 IDA는 argc, argv, envp 로 3개의 인자를 받는다고 해석이 되고 

동작은

**1. Sleep 함수를 호출하여 1초 대기합니다.**

**2. qword_14001DBE0에 "Hello world!\n" 문자열의 주소를 넣고**

**3. sub_140001060에 " Hello world!\n" 를 인자로 전달하여 호출합니다.**

**4. 0을 반환합니다.  반환 값 - 0을 반환합니다.**

여기서 qword_14001DBE0은 data 에 위치하고, "Hello, world!\n"은 rodata 에 위치하고, 
sub_140001060은 printf 함수라고 합니다. 

그 다음에 지금까지 정적분석으로 살펴본걸 바탕으로 이어서 동적 분석으로 살펴보도록 하겠습니다. 

**정적 분석**

대부분의 디버거에는 우리가 원하는 지점까지 프로그램을 실행시킬 수 있는 중단점과 실행이라는 기능이 있습니다. 

중단점을 특정 주소에 설정하고, 실행 명령을 내리면 프로그램은 중단점까지 멈추지 않고 실행됩니다. 우리는 main 함수의 동작이 궁금하므로, 이 기능을 이용하여 main 함수로 진입하겠습니다.

MAC에서는 디버깅이 되지않아 윈도우로 넘어가 디버깅을 해보았습니다. 



<img width="189" alt="image" src="https://user-images.githubusercontent.com/78540248/156585271-f1b1ff74-7136-40dd-9783-0e4e71d25ecb.png">




<img width="172" alt="image" src="https://user-images.githubusercontent.com/78540248/156585619-ea006906-8077-41a4-81c2-b0020ec14e8a.png">


먼저 main 함수의 중단점을 설정하겠습니다. 중단점을 설정하는 단축키는 F2입니다. 중단점을 설정하면 저렇게 빨간색으로 표시가 됩니다. 

그다음 디버깅을 시작하여 main 함수까지 실행합니다. 디버깅 시작 단축키는 F9입니다.  앞서 살펴본 중단점과 실행이 필요한 실행 과정을 생략하는 기능이라면, Step Over(F8)은 관심이 있는 부분의 코드를 정밀하게 분석하기 위해 사용하는 기능입니다. 

지금부터는 F8을 누르며 프로그램의 동작을 분석해보도록 하겠습니다.

**1. sub rsp, 38을 통해 main 함수가 사용할 스택 영역을 확보합니다.**

**2. rsp+0x20에 4 바이트 값인 0x000003e8을 저장합니다.**

**3. rsp+0x20에 저장된 값을 ecx에 옮깁니다. 이는 함수의 첫 번째 인자를 설정하는 것입니다.**

**4. Sleep 함수를 호출합니다. ecx가 0x3e8이므로, Sleep(1000)이 실행되어 1초간 실행이 멈춥니다.**

**5. “Hello, world!\n” 문자열의 주소를 rax에 옮깁니다.**

**6. 아래의 메모리 덤프 창을 이용하여 0x14001a140의 데이터를 보면 실제로 해당 문자열이 저장되어 있음을 확인할 수 있습니다.**

**7. rax의 값을 data세그먼트의 주소인 0x14001dbe0에 저장합니다.**

**8. 0x14001dbe0에 저장된 값을 rcx에 옮깁니다. 이는 다음 호출할 함수의 첫번째 인자로 사용될 것입니다.**

**9. 0x140001060함수를 호출합니다. 우리는 정적 분석을 통해 이 함수를 printf 함수라고 추측했습니다.**

**10. 프로그램을 확인하면, Hello, world!가 출력되어 있습니다. 정적 분석을 할 때는 함수의 기능을 추측하기 어려웠지만, 동적 분석으로는 문자열을 출력하는 함수라는 사실을 쉽게 알 수 있습니다.**
 
**11. 시작할 때 확장한 스택 영역을 add rsp, 38을 통해 다시 축소하고, ret 으로 원래 실행 흐름으로 돌아갑니다.**


<img width="377" alt="image" src="https://user-images.githubusercontent.com/78540248/156586479-63332230-8992-48e5-9c5a-1baf01aa463b.png">


앞의 Step Over를 자세히 관찰해보니 Step Over가 함수의 내부로 진입하지 않는다는 것을 발견할 수 있었습니다. 
이번에는 printf( )에 중단점을 설정하고, 해당 함수의 내부로 진입해 보았습니다. 
Ctrl + F2를 눌러 디버깅을 중단하고, printf를 호출하는 0x14000110b에 중단점을 설정하고 디버깅을 다시 시작하고, Continue(F9)를 클릭하여, printf 함수에 도달합니다.
(Step into) F7 단축키를 통해 함수 내부로 들어갑니다. 함수 내부로 RIP가 이동한 것을 확인할 수 있습니다.  


![image](https://user-images.githubusercontent.com/78540248/156587324-5fb29b50-6f5d-49f6-8ea5-7474c51f1665.png)
<img width="202" alt="image" src="https://user-images.githubusercontent.com/78540248/156586950-43e41411-6cd7-44c3-b072-4ba96c941d83.png">




IDA를 이용하면 실행중인 프로세스의 메모리를 조작할 수 있습니다.

기존 코드에서는 Sleep(delay=1000)을 호출하여 1초 동안 프로세스를 정지시켰습니다. 이번에는 delay의 값을 1000000으로 조작하여 1000초 동안 프로세스를 정지시켜보겠습니다.


**1. delay를 Sleep함수의 인자로 전달하는 부분에 중단점을 설정하고, 프로세스를 재시작합니다.**

**2. 스택을 보면 rsp+0x20에 delay의 값인 0x3e8이 저장되어 있습니다.**

**3. 해당 값을 클릭하고, F2를 누른 뒤 0xf4240(=1000000)을 입력합니다. 그리고 다시 F2를 눌러서 값을 저장합니다. delay의 값이 변경된 것을 확인할 수 있습니다.**

**4. 이제 F9를 눌러서 Sleep함수를 호출합니다. 아까와 달리 한참을 기다려도 프로세스가 재개되지 않습니다. 1000초는 대략 20분이므로, 20분 정도를 대기해야 프로세스가 재개됩니다.**


<img width="234" alt="image" src="https://user-images.githubusercontent.com/78540248/156588114-108f0448-e1f7-4fee-9007-67e1a5b8ec98.png">



<img width="278" alt="image" src="https://user-images.githubusercontent.com/78540248/156588272-ecaf4e49-d1bc-48fe-a2bf-a34c9fa251c6.png">


<img width="299" alt="image" src="https://user-images.githubusercontent.com/78540248/156588376-b26262c7-7a03-4fe2-b9c4-13e8896aea5b.png">


마무리를 지으며 IDA를 사용하여 Hello world.exe 를 IDA로 분석을 해보았습니다. 
이번에 썼던 단축키를 정리하며 마치겠습니다.


#### Conclusion

* BreakPoint(F2): 중단점을 설정하고 프로그램이 해당 지점에 도달하는 순간 정지합니다.

* Restart(Ctrl + F2): 디버깅을 중단합니다.

* Run(F9): 프로그램을 계속 실행, 또는 디버깅을 시작합니다.

* Step Into(F7): 어셈블리 코드를 한 줄 실행하지만 함수의 호출이라면, 함수 내부로 들어갑니다.

* Step Over(F8): 어셈블리 코드를 한 줄 실행하지만 함수 내부로는 들어가지 않습니다.
