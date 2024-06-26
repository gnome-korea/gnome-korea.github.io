---
title: "그놈(GNOME) 플랫폼 아키텍쳐 개관"
draft: false
---

## 그놈(GNOME) 플랫폼 아키텍쳐 개관
이 문서는 그놈 플랫폼에서 사용하는 구조/기술들에 대한 내용을 다루게 됩니다. 아래의 내용을 바탕으로, 전체적인 그놈 데스크탑 환경의 배경이 되는 아키텍쳐를 이해하여 사용자/어플리케이션 개발자들이 어떠한 방향으로 접근해나가야 할 것인지에 대한 사항을 다루게 됩니다.

## GTK+
[GTK+](http://gkt.org) 는 그놈이 사용하는 기본 위젯 툴킷(base widget toolkit, 주: 버튼, 스크롤바, 입력창등이 위젯에 해당합니다) 라이브러리 입니다. 원래 [GIMP](http://gimp.org) 프로젝트로부터 파생되었고, 대부분의 코드는 C로 작성되었지만 [수많은 언어 바인딩](https://www.gtk.org/docs/language-bindings/)을 지원, C++, Python, JavaScript, Rust 등에서도 사용할 수 있습니다.

## Glib
GLib 라이브러리는, C 언어에서 보다 더 편리한 기능적 요소를 제공해주고 있습니다. GLib은 GTK+ 를 포함, 그놈 프로젝트의 많은 프로그램 구석구석에서 사용됩니다. 그놈에서는 GLib을 사용하는 것으로 다음의 4가지 기능을 얻을 수 있었습니다.

첫째, 이식성(Portability)입니다. 예를 들어 보자면, 몇몇 C 라이브러리에서는 제공하지만, 또 다른 몇몇 C 라이브러리에서 제공하지 않는 함수들을 제공합니다. 예를 들어볼까요? GLib에는 g_ascii_strcasecmp() 와 g_memmove() 같은 함수가 있습니다. 같은 기능을 하는 함수가 glibc에도 있습니다. strcasecmp() 와 memmove() 가 똑같은 기능을 제공합니다. 만약 기본 C 라이브러리로 glibc를 사용하는 플랫폼이라면 g_ascii_strcasecmp() 와 g_memmove()를 호출시 glibc로 구현되어 있는 함수의 wrapper 역할을 하게 될테지만, 저런 함수를 제공하지 않는 C 라이브러리를 사용하는 플랫폼이라면, 앞의 두 함수를 호출 시 GLib에서 이식성있게 새로 구현해놓은 코드를 사용하게 합니다. 이런 라이브러리를 사용하는 것으로, 프로그래머가 여러가지 플랫폼을 위해 신경쓰는 일을 줄여줄 수  있습니다.

둘째, GLib은 몇몇 특정 함수를 통해 C를 더욱 편리하게 쓸 수 있게 해줍니다. 예를 들어볼까요? GLib의 날짜 관련 함수 중에, [g_date_set_parse()](https://developer.gnome.org/glib/stable/glib-Date-and-Time-Functions.html) 라는게 있습니다. 흔히 일반적이라고 생각하는 날짜 기록방식대로 문자열을 만들어 던져주면, 로케일 등을 따져서 날짜를 내부에서 사용하기 편한 자료 구조로 만들어줍니다. 그 밖에도, 문자열을 다루거나, 로그 나 디버그 메시지 출력 / 관리에 대해 유연하게 처리할 수 있게 해줍니다.

세번째로는 [제너릭한 자료 구조](https://developer.gnome.org/glib/stable/glib-data-types.html)를 GLib에서 제공받을 수 있습니다. Linked List 라던지, 해시 테이블, Balanced 2진 트리, 가변 배열과 같이 비교적 정형화된 자료 구조를 사용하기 위해 바닥부터 코드를 만들 필요가 없다는 장점이 생깁니다. GLib에서 제공해주는 함수만으로도, 해당 자료 구조를 자유롭게 수정, 추가, 삭제, 탐색할 수 있습니다. 조건에 따라 변화할 수 있는 해시 테이블 같은 자료 구조는, 자기가 원하는 행동을 담은 콜백 함수를 연결할 수 있는 정형화된 인터페이스를 제공해줍니다.

마지막으로는 [GLib main loop](https://developer.gnome.org/glib/stable/glib-The-Main-Event-Loop.html)를 네번째로 들었는데, 이 추상적인 단어는 확장성이 충분히 고려된 이벤트 loop을 정형화된 형태로 제공한다는 것을 의미한다고 설명드릴 수 있습니다. 타이머라던지, 입출력 콜백, 그리고 idle 상태 감지와 같이 일반적으로 사용되는 이벤트 발생원을 포함해서, X Window System의 이벤트와 같이 외부의 이벤트 발생원으로부터 나오는 이벤트들을 일관적인 형태로 처리할 수 있습니다. 이것을 통해 GTK+ 가 이벤트 기반의 위젯 컨트롤이라던지, 각종 이벤트 / 시그널 처리를 할 수 있는 기반을 마련하게 됩니다. 예를 들어볼까요? 눈에 보이는 것을 들어보자면, 마우스를 움직여 스크롤 바를 내린다던지, 마우스 오른쪽 버튼을 팝업 메뉴를 띄우는 것도, X 로부터 발생하는 이벤트를 넘겨받아 프로그래머가 의도한 대로 처리하게 하는 것입니다. 이해를 돕기위해 눈에 보이는 것을 예로 들었지만, GLib에서는 눈에 보이건, 보이지 않건 상관없이 동일한 이벤트 loop을 사용해서 이벤트 기반의 프로그램을 만들 수 있습니다.

GLib에 대해서 추가적으로 알고 싶으시다면, [GLib API](https://developer.gnome.org/glib/stable/) 매뉴얼을 참고하시길 권장합니다.

## GTK+ 객체 시스템(Object System)

앞에서 언급했듯이, GTK+는 객체 지향 프로그램을 위한 특별한 지원이 없는 C로 작성되었지만, GTK+ 의 디자인은 객체 지향을 중심으로 이루어져있습니다.  소위 객체지향의 전통적인 기능들인 상속(Inheritance), 다형성(Polymorphism), 레퍼런스 카운팅(Reference counting) 은 물론이고, 위젯 툴킷에 적합한 기능- 즉, 알림(Notification)을 위한 시그널 처리 시스템을 포함해서, 객체 속성(Attribute) 시스템과 같은 것으로 구성되어있습니다. 이러한 기능을 바탕으로, GTK+ 객체 시스템이라는 하나의 계층(layer)로 자리잡고 있습니다.

GTK+ 객체 시스템에 포함된 상속 기능은 서로간의 [Nesting Structure](http://crasseux.com/books/ctutorial/Nested-structures.html) (주: 구조체 내에서 구조체를 멤버 변수로 사용함)를 통해 만들어졌습니다. 예를 들어 설명해볼까요? GtkButton 클래스는 GtkWidget 클래스로부터 상속을 받습니다. 이렇게 될 경우, GtkButton 구조체의 첫번째 부분은, 바로 GtkWidget 구조체가 되는 것입니다. 이 말은 GtkButton을 가르키는 포인터가 GtkWidget을 가르키는 포인터로 타입 캐스팅될 수 있다는 것이 됩니다.  각각의 클래스는 "각자의 클래스 함수 구현을 가르키는 함수 포인터가 모인 테이블"을 포함하는 방식을 통해, 부모 클래스로부터 오버라이딩을 할 수 있게 합니다. (예를 들자면, GtkWidgetClall 구조체가 draw() 함수를 가르키는 포인터를 가지고 있게 되면, GtkButtonClass는 자신을 그리는 구현체(Implementation)만 제공하는 것으로 버튼을 그릴수 있다는 겁니다.)

각 GTK+ 객체 클래스는 관련된 시그널의 집합체를 가지고 있습니다. 각각의 시그널은 특정한 형태의 이벤트나, 어플리케이션에서 호출(Callback)을 할 수 있는- 소위 "발생"의 의미를 가집니다. 예를 들어볼까요? GtkButton 클래스는 "clicked" 라는 시그널을 제공하는데, 이 시그널은 사용자가 그 버튼 위젯을 눌렀을 때 발생(주: 영문으로는 Emission, Emitted 라는 표현을 사용합니다)하게 됩니다. 하나의 시그널에는 여러개의 Callbacks가 연결될 수 있고, 시그널이 방출될 경우에는 연결되어 있던 모든 Callback을 향해 보내지게 되어, 이들을 순서대로 호출하게 될 것입니다. 시그널은 또한, 위젯의 행동을 변화시키는 데에도 사용될 수 있습니다. 예를 들자면, [GtkEntry](https://developer.gnome.org/gtk3/stable/GtkEntry.html) 위젯의 "insert_text" 시그널에 콜백 함수를 연결하게 될 경우, 숫자만 입력되게 한다던지, 또는 특정 문자만을 입력하게 한다던지 등의 필터링을 할 수 있는것 처럼 말이죠.

각각의 클래스는 자신과 관련된 인자(arguments)의 집합을 가지고 있습니다. 각각의 인자는 그 위젯이 가진 몇몇 특성을 표현할 수 있습니다. 예를 들자면, [GtkLabel](https://developer.gnome.org/gtk3/stable/GtkLabel.html) 위젯은 "label" 이라는 인자를 제공하여, 그 위젯에 들어갈 임의의 텍스트를 설정하게 하거나, "justify" 인자를 제공하여 레이블의 자리맞춤을 하게 할 수도 있는것 처럼 말입니다. 이러한 인자 시스템은 실행 시간대(run-time)에 동적으로 적용될 수 있다는데 그 장점이 있습니다. 쉽게 말하자면, 앞으로 어떤일이 벌어지는지 알 필요 없이, 몇몇 인자를 설정하는 것으로 상황에 맞는 인터페이스를 제공할 수 있게 된다는 것입니다.

* [KLDP](http://kldp.org/node/31629)에서, GTK+ Object System 에 관련된 토론기록을 보실 수 있습니다.
* [comp.os.linux.advocacy](https://groups.google.com/u/2/g/comp.os.linux.advocacy) 메일링 리스트에서 벌어진 GTK+ Object System 관련 토론기록을 보실 수 있습니다.

## GDK
[GDK](https://developer.gnome.org/gdk3/stable/) 라이브러리는 GTK+ 위젯(어플리케이션)과 윈도우 시스템을 이어주는 추상적 계층을 제공해줍니다. 다르게 말하자면, 무언가 화면에 그려줘야 할 것이 있다던지, 혹은 이벤트를 핸들링하고자 할 때 어플리케이션이 X 윈도우 시스템에 직접 호출을 하는 것이 아니라, GDK를 호출하여 처리하게 합니다.

윈도우 시스템과 어플리케이션 사이의 추상적인 계층은 여러가지 이점을 가져옵니다. 첫번째, 이식성(Portability)를 높일 수 있습니다. GTK+ 를 X 윈도우 시스템이 아닌 다른 윈도우기반 시스템 (Win32, Mac OSX, DirectFB 등등)에 포팅하는데는 GDK 계층을 옮기는 것만으로 해결 할 수 있습니다. 게다가 GTK+ 프로그램이 존재할지 안할지 모르는 X 확장(extensions)에 투명성있게 접근할 수 있는 기반을 제공해줍니다. 마지막으로, GDK 호출은 X 호출에 비에 응답하기 간편합니다. 평소에 거의 사용되지 않는 파라미터 값을 주지 않아도, 다른 파라미터 값에 맞는 적절한 값을 자동적으로 결정해주기도 합니다.

[GDK API 레퍼런스 문서](https://developer.gnome.org/gdk3/stable/)를 통해 GDK에 대한 더욱 상세한 내용을 보실 수 있습니다.


## Drag and Drop(DND)
드래그 앤 드롭(Drag and Drop, aka DND)을 사용해서 정보를 옮기는 것은, 대부분의 현대적인 사용자 인터페이스에서 가능한 일입니다. 사용자가 마우스를 가지고 원본(source)을 클릭하여, 옮길 곳(destination)으로 끌고 갈(drag)수 있습니다. 아이콘은 사용자에게 행동에 대한 피드백을 위해 보여지는 것입니다. GTK+ 는 사용하기 쉽고, 프로그래머가 자신이 충분이 원하는 대로 바꿀 수 있는 드래그 앤 드롭 기능을 일련의 인터페이스를 통해 제공합니다. 이 인터페이스들을 사용하는 것으로 어플리케이션은 모티프(Motif, 주: POSIX - Compliant 시스템에서 동작하는 그래피컬 위젯 툴킷.)나 Xdnd 드래그 앤 드롭 프로토콜을 지원하는 프로그램들과 상호 연계될 수 있습니다.

GTK+ 인터페이스는 원본(source)과 목표지점(destination) 관점이라는 2개의 부분으로 나뉘어져있습니다. 그 두개의 관점에서도 드래그 앤 드롭 행동의 세부적인 커스토마이징이 가능한 저수준(low-level) 인터페이스와, 일반적인 형태의 드래그 앤 드롭을 비교적 간단한 코드를 통해 구현하는 고수준(high-level) 인터페이스로 구분을 할 수 있습니다.

내부적으로, GTK+는 전통적인 Motif 의 드래그 앤 드롭 프로토콜과, 새로운 Xdnd 드래그 앤 드롭 프로토콜을 지원합니다. 어플리케이션을 만드는 프로그래머의 입장에서는 특별한 노력없이, Xdnd 프로토콜(일례로, Qt와 Star Office가 Xdnd 프로토콜을 지원합니다)과 이미 많은 수를 차지하는 Motif 프로토콜을 사용하는 어플리케이션과 상호작용이 가능합니다. (다른말로 이야기 하자면, 투명성을 가지고 있다는 뜻입니다)

Xdnd 프로토콜은 전송되는 데이터의 타입을 MIME 타입으로 취급합니다. 이 관습은 그놈 프로젝트 전체에 적용되어 있습니다.

* [Drag-and-Drop in GTK+ and GNOME](https://developer.gnome.org/gtk-tutorial/stable/c1899.html). GTK+2.0문서지만, 드래그 앤 드롭 개관을 살펴 볼 수 있습니다.
* [GTK+ API 레퍼런스 - Drag and Drop](https://developer.gnome.org/gtk3/stable/gtk3-Drag-and-Drop.html) 페이지에서, 고수준 인터페이스의  상세를 알 수 있습니다.
* [GDK API 레퍼런스 - Drag and Drop](https://developer.gnome.org/gdk3/stable/gdk3-Drag-and-Drop.html) 페이지에서, 드래그 앤 드롭의 저수준 인터페이스 상세를 알 수 있습니다.


## 테마(Themes)
GTK+는 테마를 통해 사용자 인터페이스의 사용자화를 지원합니다. GTK+나 어플리케이션을 재 컴파일 할 필요없이, 사용자는 단순히 새로운 테마를 설치하는 것 만으로도 어플리케이션의 새로운 모습을 선택할 수 있습니다. 테마는 단순히 색상의 집합뿐만 아니라, 현존하는 드로잉 코드에서 사용되는 그림(pixmap)만 추가/변경하거나, 위젯을 그리는 기능 자체를 완전히 바꿀 수 도 있습니다.
GTK+ 에서 테마가 어떻게 동작하는지를 이해하기 위해서는 몇가지의 개념이 필요합니다. 첫째로, 스타일(style)은 각각의 위젯을 어떻게 그리느냐에 대한 정보의 집합을 의미합니다. 색상에 대한 정보, 배경에 들어갈 그림, 위젯에 들어갈 글꼴 등이 이들에 해당합니다. 스타일은 또한, 위젯의 기본 구성품을 어떻게 그리는지에 대한 코드가 들어있는 공유라이브러리인 "테마 엔진(theme engine)"을 가르키는 포인터를 포함합니다. (그림자 들어간 박스, 프레임, 화살표, 체크 버튼 표시부 등등이 위젯의 기본 구성품에 포함됩니다) 배포판마다 그 이름은 대개 다르지만, gtk-engines-* 라고 들어가는 패키지들이 이들 테마 엔진입니다.

어플리케이션의 색상, 글꼴, 그리고 위젯을 위해 사용되는 테마 엔진등은 리소스 파일(resource file)로 설정됩니다. 리소스 파일에는 특정 테마 엔진에 관련된 데이터를 포함할 수도 있으며, 최종적으로 테마는 리소스 파일과 그림 파일과 같이 필요한 파일들의 조합으로 이루집니다.

그놈(GNOME) 에서는 일관된 테마의 배포를 위해 표준 파일 형식을 정의하고 있습니다. 이들 기준을 간략하게 살펴보면, 테마파일은 테마의 이름으로 된 디렉토리가 들어있는 타르볼(주: .tar, .tar.gz 와 같이 tar로 묶여있는 파일)이어야 하며, 이 디렉토리의 내부에는 README.html 파일과, PNG 형식의 아이콘인 ICON.png 파일, 그리고 각각의 테마 정보가 들어있는 하위디렉토리가 존재해야 합니다. GTK+ 테마 정보는 반드시 gtk 라는 이름의 하위디렉토리에, 리소스 파일은 gtkrc 라는 이름으로 들어가 있어야 합니다.

* [GTK+ 리소스 파일 레퍼런스](https://developer.gnome.org/gtk3/stable/gtk3-Resource-Files.html) : Resource Files 페이지에서 리소스 파일에 대한 레퍼런스를 보실 수 있습니다.
* [GTK+ CSS](https://developer.gnome.org/gtk3/stable/chap-css-overview.html) : CSS를 이용해서 GTK+에 스타일을 적용할 수 있습니다.
* [Themeable Stock Images](https://developer.gnome.org/gtk3/stable/gtk3-Themeable-Stock-Images.html) : Themeable Stock Icons 페이지에서 스톡 아이콘에 대한 레퍼런스 매뉴얼을 보실 수 있습니다.

참고로, 이 문서는 그놈 홈페이지의 [Architecture Overview](https://web.archive.org/web/20070312022147/http://developer.gnome.org/arch/)를 바탕으로 작성되었으며, 예전 [페이지](https://web.archive.org/web/20070529152631/http://www.gnome.or.kr/c/portal/layout?p_l_id=PUB.1.43)
를 현재 상황에 맞게 고친 것입니다.

