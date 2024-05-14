+++
author = "Soeun"
title = "블로그에서 바로 실행가능한 라이브 코드 에디터 만들기"
date = "2024-05-14"
summary = "Ace 와 Piston을 사용해서 만든 웹 코드 에디터"
categories = [
    "Project"
]
tags = [
    "live code editor"
]
+++

커스텀 블로그를 하면서 가장 만들고 싶었던 기능이 있었다. 블로그에서 바로 코드를 입력하고 실행할 수 있는 **라이브 코드 에디터**를 만드는 것이다 ! 마침 휴일 전날이어서 시간이 조금 났고 미뤄놨던 계획을 실행했다.

{{< codeeditor >}}
def make_code_editor():
    print("1. Plan")
    print("2. Code")
    print("3. Fix Bug")
    print("4. Post about it")

def play_music():
    print("now playing..🎶")
    print("aespa - supernova")
    print("tripleS - girls never die")

make_code_editor()
play_music()
{{< /codeeditor >}}

## Plan

생각한 기능은 아래와 같다. 

1. Python3, c++, Java, Javascript 실행할 수 있게 하기
2. 내가 블로그에 default 코드를 입력하면 그대로 보여주고, 포스팅을 보는 사용자가 수정해서 실행할 수 있게 하기 
3. 실행 속도가 빠를 것 !

우선 코드를 실행하려면 **코드 에디터**와 **코드를 실행시킬 수 있는 무언가(?)** 가 필요하다. 구글링 해본 결과 웹 상에서 사용할 수 있는 **코드 에디터**로는 [Ace](https://ace.c9.io/#nav=embedding) 가 가장 많이 사용되는 것 같았다. [How to Guide](https://ace.c9.io/#nav=howto) 도 굉장히 간편해서 Ace를 사용하기로 결정했다.

이제 어려운 부분인 **코드를 실행시킬 수 있는 무언가(?)** 를 결정해야 하는데.. 처음에는 따로 도커로 실행환경을 구축해야 하나? 생각했지만 아주 좋은 [Piston](https://github.com/engineer-man/piston) 이라는 HTTP 요청을 통해 코드를 보내고 받아올 수 있는 API 가 있었다. 

{{< github repo="engineer-man/piston" >}}


```json
{
  "language": "python",
  "version": "3.10.0",
  "files": [
    {
      "name": "code.py",
      "content": "print('Hello, world!')"
    }
  ]
}

```

대략 이렇게 요청을 보내면,


```json
{
    "language": "python",
    "version": "3.10.0",
    "run": {
        "stdout": "Hello, world!\n",
        "stderr": "",
        "code": 0,
        "signal": null,
        "output": "Hello, world!\n"
    }
}
```

이렇게 응답이 온다. 

필요한 재료들을 다 얻었으니 구현을 해보자 !

## Implementation

나는 hugo 프레임워크를 사용하므로, `layouts/shortcodes`에 `codeeditor.html` 을 만들었다. 코드 자체는 너무 간단하다.

```html
<div class="code-editor-container">
    <div class="code-editor-header">
      <label for="language" style="font-size: 14px;">Select Language: </label>
      <select id="language" class="language-selector" onchange="changeLanguage()">
        <option value="python">Python3</option>
        <option value="cpp">C++</option>
        <option value="java">Java</option>
        <option value="javascript">JavaScript</option>
      </select>
    </div>
    <div id="editor"></div>
    <button class="run-button" onclick="runCode()">Run Code</button>
    <pre id="output"></pre>
  </div>
  <script>
    var editor = ace.edit("editor");
    editor.setTheme("ace/theme/textmate");
    editor.session.setMode("ace/mode/python"); // Default to Python
    var initialContent = `{{ .Inner | htmlUnescape | safeJS }}`;
    editor.setValue(initialContent.trim()); 
  
    function changeLanguage() {
      var language = document.getElementById("language").value;
      var mode = "ace/mode/";
      if (language === "python") {
        mode += "python";
      } else if (language === "cpp") {
        mode += "c_cpp";
      } else if (language === "javascript") {
        mode += "javascript";
      } else if (language == "java") {
        mode += "java";
      }
      editor.session.setMode(mode);
    }
  
    function runCode() {
      var code = editor.getValue();
      var language = document.getElementById("language").value;
  
      var pistonLanguage;
      var version;
      if (language === "python") {
        pistonLanguage = "python";
        version = "3.10.0";
      } else if (language === "cpp") {
        pistonLanguage = "cpp";
        version = "10.2.0";
      } else if (language === "javascript") {
        pistonLanguage = "javascript";
        version = "18.15.0"
      } else if (language == "java") {
        pistonLanguage = "java";
        version = "15.0.2"
      }
  
      fetch('https://emkc.org/api/v2/piston/execute', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          language: pistonLanguage,
          version: version,
          files: [{
            name: 'code.' + pistonLanguage,
            content: code
          }],
          stdin: "",
          args: [],
          compile_timeout: 10000,
          run_timeout: 3000,
          compile_memory_limit: -1,
          run_memory_limit: -1
        })
      })
      .then(response => response.json())
      .then(data => {
        document.getElementById('output').innerText = data.run.output;
      })
      .catch(error => {
        document.getElementById('output').innerText = 'Error: ' + error.message;
      });
    }
  </script>
  
```
그리고 `layout/_default/baseof.html`에서 Ace 에디터를 불러오도록 아래와 같이 추가해줬다. 

```html
<!-- Include Ace Editor -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/ace/1.4.12/ace.js"></script>
<!-- css for Ace Editor -->
<link rel="stylesheet" href="{{ "css/custom.css" | relURL }}">
```

기본 html 형태가 마음에 들지 않아서 gpt의 도움을 받아 css를 만져주었다. 이 css 파일은 assets/css/custom.css 에 있다. 

```css
/* Live code editor setting */
.code-editor-container {
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
  font-family: "Meslo LG M", sans-serif;
}
.code-editor-header {
display: flex;
justify-content: space-between;
align-items: center;
margin-bottom: 10px;
}
.language-selector {
padding: 5px;
font-size: 14px;
border: 1px solid #ccc;
border-radius: 5px;
background-color: #f5f5f5;
cursor: pointer;
transition: border-color 0.3s ease;
}
.language-selector:hover {
border-color: #888;
}

#editor {
width: 100%;
height: 350px;
border: 1px solid #ddd;
border-radius: 5px;
box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
.run-button {
padding: 10px 15px;
font-size: 14px;
color: #fff;
background-color: #007bff;
border: none;
border-radius: 5px;
cursor: pointer;
transition: background-color 0.3s ease;
margin-top: 10px; 
}
.run-button:hover {
background-color: #0056b3;
}

#output {
margin-top: 15px;
padding: 10px;
border: 1px solid #ddd;
border-radius: 5px;
background-color: #f8f9fa;
white-space: pre-wrap;
word-wrap: break-word;
}
```


## Fix Bug

### 초기코드 설정하기

여기서 가장 삽질했던 부분이 아래 부분이다. 코드를 단순히 실행하는 것을 넘어서 블로그에 들어갈 코드 에디터이므로 포스팅을 작성할 때 내가 기본 코드를 입력하고 그것을 보여주고 싶었다. 그래서 아래와 같이 구현했는데, 깃헙 액션을 이용해서 배포할 때 오류가 발생했다.


```javascript
var initialContent = `{{ .Inner | htmlUnescape | safeJS }}`;
editor.setValue(initialContent.trim()); 
```

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/7642d991-159c-476d-98c0-c4cfb0694f75)

```text
"/home/runner/work/ddoddii.github.io/ddoddii.github.io/content/post/CS/java/garbage-collector/index.md:182:1": failed to render shortcode "codeeditor": failed to process shortcode: execute of template failed: html/template:shortcodes/codeeditor.html:19:24: {{.Inner | safeJS}} appears in a JS template literal
```

음.. 🤔 뭐가 문제일까? hugo 포럼에서 [비슷한 질문](https://discourse.gohugo.io/t/site-baseurl-appears-in-a-js-template-literal/46692)을 찾았다. 

달린 답변이, Go가 어떤 보호 메카니즘이 있어서 사이트 이러한 보호를 통과해서 JS Template literal 을 실행시키려면 아래와 같이 config 파일에 추가하면 된다고 했다. 

![image](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/79d183e0-bdaa-4ab4-bb50-98dbc8c7182e)

따라서 config 파일에 추가하고 다시 배포한 결과 ! 에러가 나지 않았다. 😃

### cpp thread 라이브러리 관련 이슈

 다만 c++ 에서 thread 라이브러리를 포함한 코드를 실행했는데, 오류가 났다. 아마 gcc의 버전때문인 것 같은데.. 아니면 -pthread 를 포함해서 오브젝트 파일을 실행해야 하는데 이 파라미터를 어디에 추가하는지 모르겠다.. 🥲 여기 [execution endpoint](https://github.com/engineer-man/piston?tab=readme-ov-file#execute-endpoint) 에는 컴파일 커맨드를 추가하는 파라미터가 없다.   아무리 실행해도 오류가 나서 [Piston 레포에 이슈](https://github.com/engineer-man/piston/issues/665) 를 남겼다. 해결되는 대로 업데이트 하겠다.. 

## Summary

이렇게 간단하게 블로그에서 코드를 바로 실행해 볼 수 있는 코드 에디터를 만들어 보았다. python, c++, javascript, java 모두 잘 돌아간다. 생각보다 쉽게 구현 가능해서 놀랐고 blowfish 에 내가 구현한 것 아이디어를 남겼는데 PR 머지가 되었으면 좋겠다 !! 이렇게 연휴 전날 끝 ... 