+++
author = "Soeun"
title = "MVC 패턴과 MVC 패턴을 지키기 위한 규칙들"
date = "2023-10-20"
summary = "유지보수가 편해지는 MVC 패턴"
categories = [
    "CS"
]
tags = [
    "MVC"
]
+++

## MVC 개념 

MVC 패턴은, 기본적으로 **유지보수가 편해지는 코드 구성 방식**이다. 개발을 진행하다가 프로젝트 규모가 커지니, MVC 패턴으로 코드를 정리하니 유지보수가 편해지는 것을 느낀 개발자들이 MVC 패턴을 만든 것이다 ! 처음에는 대체 MVC 가 뭘까.. 했는데 역시 처음 작은 개인 프로젝트에 MVC 를 적용하는 것은 힘들다. 자연스럽게 프로젝트가 커지면서 고민해야 하는 부분이다. 하지만 MVC 패턴을 배웠으니 처음부터 적용하도록 연습해보자 ! 절대 정해진 정답이 있는 것은 아니니, 마음 편히 가지자. 

우선 MVC개념을 비유적으로 쉽게 설명한 [웹사이트](https://realpython.com/the-model-view-controller-mvc-paradigm-summarized-with-legos/)가 있다. 

레고 더미가 내 앞에 있다고 하자. 근데 친구가 오더니 레고를 가지고 커다란 우주선을 만들어달라고 요청(request) 했다고 하자. 그러면 나는 우주선을 만들 수 있는 큰 레고, 작은 레고(building blocks)를 고를 것이다. 고른 레고들을 가지고 우주선을 만든다. 

이 과정을 **MVC 패턴**으로 바꿔보자. 일단 시작은 **request** 이다. 친구가 와서 우주선을 만들어달라고 요청했다. 웹페이지에서는 유저가 URL 을 입력해서 해당 페이지를 보려는 요청을 보내는 것과 같다. Request 가 **Controller** 에 도달한다. 위의 예제에서는 내가 바로 controller 이다.  내가 우주선을 만들기 위한 레고 블럭들을 찾아야 한다. Controller 도 요청받은 사항을 수행하기 위한 building block 을 찾아야 한다. 이 Building Block 이 바로 **model** 이다. 웹에서는 model 이 controller 를 도와서 데이터베이스에서 필요한 정보를 찾도록 도와준다. 최종 우주선이 바로 **view** 이다. 요청 한 사람이 최종으로 보게 되는 화면이 view 이다. 

Web App 에서 다시 설명하자면,
1. 사용자가 URL 을 입력하여 페이지 보기를 요청한다.
2. Controller 가 사용자의 요청을 받는다.
3. Model 을 사용하여 필요한 데이터를 찾고, 정렬하고, 최종적으로 View 로 보낸다.
4. 사용자는 자신이 요청한 페이지인 View 를 브라우저에서 볼 수 있다.

<img width="550" alt="스크린샷 2023-10-21 오후 6 14 44" src="https://github.com/ddoddii/ddoddii.github.io/assets/95014836/bc9ea3af-0b7b-4bf2-94e1-72e6cc2f23ce">

Model 은 데이터와 관련된 일을 한다. Controller 는 Model, View 의 중개자 역할을 한다. View 는 사용자에게 보여지는 부분이다. 이 패턴의 가장 큰 장점은, 사용자가 보는 View 와 비즈니스 로직을 처리하는 부분을 분리했다는 것이다. 그래서 확장에 더욱 용이하다 ! 

## MVC 패턴을 적용시키기 위한 규칙

**1. Model 은 Controller 과 View에 의존하면 안된다.**

Model 내에 Controller, View 에 관련된 코드가 있으면 안 된다. 

**2. View 는 Model 에만 의존해야 하고, Controller 에는 의존하면 안된다.**

   View 내부에 Model의 코드만 있을 수 있고, Controller 의 코드가 있으면 안된다. 
   
**3. View 가 Model 로부터 데이터를 받을 때는, 사용자마다 다르게 보여주어야 하는 데이터에 대해서만 받아야 한다.**

   모든 사용자에 대해서 같은 정보는 View 가 자체적으로 가지고 있어야 한다. 
   
**4. Controller 는 Model 과 View 에 의존해도 된다.**

   왜냐하면 Controller 는 Model, View 를 중개하면서 전체 로직을 구성하기 때문이다. 
   
**5. View 가 Model 로부터 데이터를 받을 때, 반드시 controller 에서 받아야 한다.** 




## 좀 더 기술적인 접근(Flask)
내가 URL 을 브라우저에 입력하면, 나는 웹 어플리케이션에게 특정한 페이지를 보기 위한 요청을 보내는 것이다. 그렇다면 웹 어플리케이션은 어떤 페이지를 보여주고 렌더링할지 어떻게 알 수 있을까?

웹 어플리케이션을 만들 때, routes 라는 것을 정의한다. Routes 는 각각 다른 페이지와 연관된 URL pattern 이다. 그래서 어떤 사람이 URL 을 입력하면 뒤에서는 웹 어플리케이션이 이 URL 과 사전 정의된 routes 를 매칭하려고 노력한다. 

그래서 사실 4가지 component 가 있다 : routes, models, views, controllers

### Routes
각 route 는 controller 와 연관되어 있다 - 더 자세히 서술하자면, controller 내의 function, 즉 controller action 과 관련 있다. 그래서 URL 을 입력했을 때 매칭되는 route 를 찾으면 그 route 과 연관된 controller action 을 call 한다. 

Flask 에서의 예시는,
```python
@app.route('/')
def main_page():
	pass
```

여기서 우리는 / route 를 main_page() view function 과 연관시켰다. 

### Models and Controllers
Controller action 에서는 2가지 일들이 일어난다 - models 는 필요한 데이터를 데이터베이스에서 가져온다 . 그 데이터는 view 로 전해지고, 요청된 페이지를 렌더링한다. model이 가져온 데이터는 list 나 dictionary 와 같은 자료구조에 더해지고, 그 구조가 view 로 보내진다. 

```python
@app.route('/')
def main_page():
    """Searches the database for entries, then displays them."""
    db = get_db()
    cur = db.execute('select * from entries order by id desc')
    entries = cur.fetchall()
    return render_template('index.html', entries=entries)
```

이 view function 에서는 db 에서 데이터를 가져온다. 이것은 List 를 반환하고, entries 변수에 할당한다. 그리고 index.html 은 이 entries 정보를 사용할 수 있다. 

### Views
View 에서는 이 자료구조와 HTML 를 렌더링 하기 위한 정보가 전해진다. 아래에서처럼 entries 를 loop 을 통해 각각 보여줄 수 있다.  

```html
{% for entry in entries %}
  <li>
    <h2>{{ entry.title }}</h2>
    <div>{{ entry.text|safe }}</div>
  </li>
{% else %}
  <li><em>No entries yet. Add some!</em></li>
{% endfor %}
```


## MVC , MVCS ?

조금 더 찾아보다 보면, Service 라는 개념도 등장한다. MVCS 는 MVC 에 Service 개념까지 합친 패턴이다. 

**Controller** 는 사용자 요청의 Entrypoint 이다. 요청을 적절하게 가공하여 service 레이어에 전달하고, 그 결과를 다시 가공해서 사용자에게 전달한다. 

**Service** 는 Controller 과 Model 사이에 존재한다. 여기서 비즈니스 로직이 처리된다. 각 서비스는 서로를 직접적으로 호출하지 않고, controller 를 통해서 소통한다. I/O 나 DB 요청, 데이터 가공은 service 계층에서 처리한다. 


결국 느낀점은.. 정답은 없다 ! 단지 개발을 하면서 복잡해지는 코드를 정리하는 방법 중 하나일 뿐이다. 그러니 확장성을 처음부터 고려해서 초기 단계에서부터 MVC 를 도입하는 것은 어렵다. 미래를 보지 않고 현재를 설계하는 것은 어려운 법이다. 하지만 이 패턴이 적용되게끔 도와주는 규칙들을 최대한 적용해보려고 하자. 그럼 미래에는 저절로 MVC 패턴대로 프로그램이 짜여 있을 것이다. 

## Reference
- https://www.youtube.com/watch?v=ogaXW6KPc8I
- https://realpython.com/the-model-view-controller-mvc-paradigm-summarized-with-legos/
- https://www.linkedin.com/pulse/mvc-mvcs-software-engineering-asher-toqeer/