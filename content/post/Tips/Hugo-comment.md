+++
author = "Soeun"
title = "Hugo Blog 에서 Comment 사용하기"
date = "2023-02-05"
summary = "utterances 이용해서 코멘트 만들기"
categories = [
    "tips"
]
tags = [
    "Hugo Blog"
]
image = ""
+++

### Utterances 

내 개인 blog 를 만들고 comment 를 만들고 싶었는데, disqus, commento, utterances 중에서 고민하다가 ==utterances== 를 골랐다. 
이유는 아래와 같다. 
- 익숙한 github issues 를 이용한 코멘트 방식
- 무료 (최대 이유)
- 설치하기 정말 쉽다 


### 설치 방법

1. **[utterances](https://utteranc.es/?installation_id=40366359&setup_action=install)로 가서 아래 사진 속 네모 친 utterances app 을 클릭한다.** 

   ![img](https://github.com/ddoddii/ddoddii.github.io/assets/95014836/cd69a2c0-9b48-4687-b2e8-0d4ad43cccb3)

   utterances app 을 내 hugo blog 가 deploy 되는 레포지토리에 설치한다. 

2. **Public 저장소 생성** 

    My-comments 라고 했다.

3. **Blog Post <-> Issue Mapping, Theme 선택** 

   나는 첫번째꺼, Theme 은 github-light 를 이용했다. 
   다 설정 완료하면 아래 코드창이 뜬다. 이것을 복사하자.

4. **Enable Utterances** 

   이 부분이 가장 헷갈렸다. 내 사이트 중 페이지의 뼈대 되는 html 페이지를 찾아서 그 밑에 위에 복사한 소스코드를 복사해야 한다.
   나는 themes/<내 테마 이름>/layouts/_default/single.html 이 내 싱글 페이지의 뼈대가 되는 소스코드였다. 
   나는 related-post 아래 comment 창을 두고 싶어서 아래와 같이 붙여넣었다. 
   

    ```go
    {{ define "body-class" }}
        article-page
        {{/* 
            Enable the right sidebar if
                - Widget different from 'TOC' is enabled
                - TOC is enabled and not empty
        */}}
        {{- $HasWidgetNotTOC := false -}}
        {{- $TOCWidgetEnabled := false -}}
        {{- range .Site.Params.widgets.page -}}
            {{- if ne .type "toc" -}}
                {{ $HasWidgetNotTOC = true -}}
            {{- else -}}
                {{ $TOCWidgetEnabled = true -}}
            {{- end -}}
        {{- end -}}

        {{- $TOCManuallyDisabled := eq .Params.toc false -}}
        {{- $TOCEnabled := and (not $TOCManuallyDisabled) $TOCWidgetEnabled -}}
        {{- $hasTOC := ge (len .TableOfContents) 100 -}}
        {{- .Scratch.Set "TOCEnabled" (and $TOCEnabled $hasTOC) -}}
        
        {{- .Scratch.Set "hasWidget" (or $HasWidgetNotTOC (and $TOCEnabled $hasTOC)) -}}
    {{ end }}

    {{ define "main" }}
        {{ partial "article/article.html" . }}

        {{ if .Params.links }}
            {{ partial "article/components/links" . }}
        {{ end }}
        
        {{ partial "article/components/related-content" . }}
        
        {{ if not (eq .Params.comments false) }}
            {{ partial "comments/include" . }}
        {{ end }}
        <script src="https://utteranc.es/client.js"
        repo="ddoddii/ddoddii.github.io"
        issue-term="pathname"
        label="Comment"
        theme="github-light"
        crossorigin="anonymous"
        async>
        </script>
        {{ partialCached "footer/footer" . }}

        {{ partialCached "article/components/photoswipe" . }}
    {{ end }}

    {{ define "right-sidebar" }}
        {{ if .Scratch.Get "hasWidget" }}{{ partial "sidebar/right.html" (dict "Context" . "Scope" "page") }}{{ end}}
    {{ end }}
    ```