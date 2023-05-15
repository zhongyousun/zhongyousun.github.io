---
title: "Hugo 實現推薦文章功能與輪播顯示"
date: 2023-02-22T11:02:15+08:00
draft: false
slug: "tutorials/hugo-related-posts-supports-carousel-display"
thumbnail: "img/thumbnail/021.png" # Thumbnail image
description: "簡單實現以輪播方式顯示推薦文章列表。"
categories:
  - "程式語言"
tags:
  - "Hugo"
# Theme-Defined params
lead: "" # Lead text
authorbox: true # Enable authorbox for specific page
pager: false # Enable pager navigation (prev/next) for specific page
toc: true # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page
---
簡單實現以輪播方式顯示推薦文章列表。
<!--more-->

### 前言
其實好奇推薦文章的實作方式很久了，也看過一些論點對於網站是否有推薦文章功能並不是哪麼的必需!!?但對於自己本身，如果在瀏覽網站時，對於查閱推薦文章若發現有興趣的文章再點進去的機率實在是非常高。所以來記錄一下實做的細節，並且用輪播的方式呈現。  
輪播也是一個大坑，畢竟弄不好，若在手機平板閱讀時，反而會導致操作不順，此篇使用的輪播外掛完全沒有這個問題👍。  

{{< br 3 1 >}}
### 文章關聯性判斷
推薦文章的內容可以直接藉由Hugo本身文章的tags參數去取得關聯性，邏輯是會先在所有文章中搜尋是否有跟當下文章有相同tags的文章，若有的話就顯示推薦文章列表。    

為了避免修改到Hugo原本的主題，可以對`根目錄\layouts\_default\single.html`修改。  
(若沒有此檔案可以從`根目錄\themes\主題\layouts\_default\single.html`複製過去)  
推薦文章列表通常會放在文章底部，可以依自己喜好選擇放在哪裡。我是選擇放在留言版的上方，所以程式碼加在`single.html`裡`{{ partial "comments.html" . }}`的上方。 

```HTML {linenos=inline}
{{- range first 1 (where (where .Site.Pages ".Params.tags" "intersect" .Params.tags) "Permalink" "!=" .Permalink) -}}
    {{- $.Scratch.Set "has_related" true -}}
{{- end -}}

{{ if $.Scratch.Get "has_related" }}
    <div class="related-content">
        <h3>See Also</h3>
        <ul>
            {{- $num_to_show := .Site.Params.related_content_limit | default 3 -}}
            {{ range first $num_to_show (where (where .Site.Pages ".Params.tags" "intersect" .Params.tags) "Permalink" "!=" .Permalink) }}
            <li>
                <a href="{{ .RelPermalink }}">{{ .Title }}</a>
                &ndash; 
                <time datetime="{{ .Date.UTC.Format "2006-01-02T15:04:05-0700" }}">
                    {{ .Date.Format "Jan 2, 2006" }}
                </time>
                <br> 
                <small>{{ .Summary | plainify | htmlUnescape }}</small>
            </li>
          {{ end }}
        </ul>
    </div>
{{ end }}
```
`{{ .Title }}`為相關文章標題  
`{{ .RelPermalink }}`為相關文章連結  
`{{ .Summary | plainify | htmlUnescape }}`為相關文章的摘要   
{{< br 3 12 >}}

{{< figure src="pics/RelatedPosts.png" width="80%" alt="RelatedPosts" title="相關文章列表，預設顯示三篇">}}  

{{< br 3 13 >}}

### 輪播顯示
只顯示文章標題、日期與摘要感覺有點單調，所以網路上找了一下輪播的實作。最後選擇了使用[Owl Carousel](https://owlcarousel2.github.io/OwlCarousel2/  "Owl Carousel")，覺得功能十分強大包含了RWD、參數設定靈活，還可以藉由滑鼠拖拉來使圖片輪播。

引入JS與CSS
```HTML {linenos=inline}
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/OwlCarousel2/2.3.4/assets/owl.carousel.min.css"></link>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/OwlCarousel2/2.3.4/assets/owl.theme.default.min.css"></link>
<script src="https://code.jquery.com/jquery-3.5.1.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/OwlCarousel2/2.3.4/owl.carousel.min.js"></script>

```

可以自行選擇畫面要呈現多少個輪播區塊
```JAVASCRIPT {linenos=inline}
$(".owl-carousel").owlCarousel({
  loop: true, // Enable loop playback
  margin: 10, // Set margin between items to 10px
  nav: true, // Enable navigation dots
  responsive: {
    0: {
      items: 2 // Display 2 items on screens from 0 to 600
    },
    600: {
      items: 4 // Display 4 items on screens from 600 to 1000
    },
    1000: {
      items: 4 // Display 4 items on screens larger than 1000
    }
  }
});

```


想要在輪播區塊顯示文章封面，在滑鼠經過時有浮出的效果，針對自己想要的樣子客製化一下。
```CSS {linenos=inline}
.owl-theme .item {
	height: 15rem;
	background-color: rgba(245, 245, 245, 0.3);
	padding: 0.5rem;
	box-shadow: 0 0 20px 0 rgba(0, 0, 0, 0.1);
	transform: scale(0.9, 0.9);
	transition: all 0.3s ease-out;
}
.owl-theme .item:hover {
	transform: scale(1.0, 1.0);
}
.owl-carousel .item h4 {
	color: #000000;
	font-weight: 400;
	font-size: 1rem;
	margin-top: 0rem;
}
.owl-carousel .item img {
	width: 100%;
	height: 7rem;
	border-radius: 4px;
}

```

因為我的文章有些有封面，有些沒有，所以再調整一下Html，若沒封面則顯示文章標題與摘要，若有封面則顯示文章封面與標題。

```HTML {linenos=inline}
{{- range first 1 (where (where .Site.Pages ".Params.tags" "intersect" .Params.tags) "Permalink" "!=" .Permalink) -}}
	{{- $.Scratch.Set "has_related" true -}}
{{- end -}}

{{ if $.Scratch.Get "has_related" }}
<div class="related-content">
<h3>See Also</h3>
{{ end }}
<div class="owl-carousel owl-theme">
	{{- $num_to_show := .Site.Params.related_content_limit | default 5 -}}
	{{ range first $num_to_show (where (where .Site.Pages ".Params.tags" "intersect" .Params.tags) "Permalink" "!=" .Permalink) }}
	<div class="item">
		{{ if .Params.Thumbnail }}
			<div class="related-post__image">
				<a href="{{ .RelPermalink }}">
					<img src="{{ absURL .Params.Thumbnail }}" alt="{{ .Title }}"/>
				</a>
			<h4><a href="{{ .RelPermalink }}">{{ .Title }}
				&ndash; 
				<time datetime="{{ .Date.UTC.Format "2006-01-02T15:04:05-0700" }}">
					{{ .Date.Format "Jan 2, 2006" }}
				</time></a></h4></div>

		{{ else }}
			<h4><a href="{{ .RelPermalink }}">{{ .Title }}
				&ndash; 
				<time datetime="{{ .Date.UTC.Format "2006-01-02T15:04:05-0700" }}">
					{{ .Date.Format "Jan 2, 2006" }}
				</time></a></h4>
				<hr>
				<small>{{ .Summary | plainify | htmlUnescape }}</small>

		{{ end }}	
	</div>
  {{ end }}

```
{{< br 4 14>}} 

### 成品展示
{{< br 2 2>}}
{{< figure src="pics/carousel.gif" width="80%" alt="carousel" title="推薦文章輪播">}} 
{{< br 2 15>}} 


{{< br 10 3>}}
> 參考
- [Show Related Posts in Hugo](https://makewithhugo.com/show-related-posts/  "Show Related Posts in Hugo")

- [簡單好上手的圖片輪播 jQuery - Owl Carousel](https://ithelp.ithome.com.tw/articles/10247358  "簡單好上手的圖片輪播 jQuery - Owl Carousel")