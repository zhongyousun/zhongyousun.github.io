---
title: "Implementing Recommended Posts and Carousel Display in Hugo"
date: 2023-02-22T11:02:15+08:00
draft: false
slug: "tutorials/hugo-related-posts-supports-carousel-display"
thumbnail: "/img/thumbnail/021en.png" # Thumbnail image
description: "Simple implementation of recommended posts list displayed in a carousel format."
categories:
  - "Program"
tags:
  - "Hugo"
# Theme-Defined params
lead: "" # Lead text
authorbox: true # Enable authorbox for specific page
pager: false # Enable pager navigation (prev/next) for specific page
toc: true # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page

---
Simple implementation of recommended posts list displayed in a carousel format.
<!--more-->

### Foreword
I have been curious about the implementation of recommended posts for a long time, and I have come across some arguments that having a recommended posts feature on a website is not necessarily essential. However, for myself, when browsing a website, I am much more likely to click into an article that catches my interest in the recommended posts section. So, I wanted to record the implementation details and present recommended posts in a carousel format.  

Implementing a carousel can be a tricky process, as it can negatively impact user experience on mobile or tablet devices if not done properly. However, the carousel plugin used in this article does not have this issue.👍。    

{{< br 3 1 >}}
### Posts Relatedness 
The content of recommended posts can be obtained directly through the 'tags' parameter of Hugo's own posts. The logic is to first search for posts with the same tags as the current post among all posts. If there are any, then the recommended post list will be displayed.    

To avoid modifying the original theme of Hugo, the `single.html` file in the `root directory\layouts\_default` can be modified. If this file does not exist, it can be copied from   
`root directory\themes\theme\layouts\_default\single.html`.  
Recommended post lists are usually placed at the bottom of posts, but can be placed wherever preferred. I chose to place it above the comment section, so I added the code above `{{ partial "comments.html" . }}` in `single.html`.

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
`{{ .Title }}`refers to the title of the recommended post.  
`{{ .RelPermalink }}`refers to the permalink of the recommended post.  
`{{ .Summary | plainify | htmlUnescape }}`refers to the summary of the recommended post.  
{{< br 3 12 >}}

{{< figure src="pics/RelatedPosts.png" width="80%" alt="RelatedPosts" title="Recommended post list, showing three posts by default.">}}  

{{< br 3 13 >}}

### Carousel display
I felt that displaying only the post title, date, and summary was a bit monotonous, so I searched the internet for a carousel implementation. In the end, I chose to use [Owl Carousel](https://owlcarousel2.github.io/OwlCarousel2/  "Owl Carousel"), I found its features to be powerful, including RWD, flexible parameter settings, and the ability to drag and slide images in the carousel.

Importing JS and CSS.
```HTML {linenos=inline}
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/OwlCarousel2/2.3.4/assets/owl.carousel.min.css"></link>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/OwlCarousel2/2.3.4/assets/owl.theme.default.min.css"></link>
<script src="https://code.jquery.com/jquery-3.5.1.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/OwlCarousel2/2.3.4/owl.carousel.min.js"></script>

```

You can choose how many carousel items to display on the screen.  
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


I wanted to display the post thumbnail in the carousel with a hover effect, so I customized it.
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

As some of my posts have thumbnail and some do not, I made some adjustments to the HTML to display the post title and summary if there is no thumbnail, and to display the thumbnail and title if there is one.  

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

### Finished product
{{< br 2 2>}}
{{< figure src="pics/carousel.gif" width="80%" alt="carousel" title="Recommended posts carousel">}}  
{{< br 2 15>}} 


{{< br 10 3>}}
> Reference
- [Show Related Posts in Hugo](https://makewithhugo.com/show-related-posts/  "Show Related Posts in Hugo")

- [簡單好上手的圖片輪播 jQuery - Owl Carousel](https://ithelp.ithome.com.tw/articles/10247358  "簡單好上手的圖片輪播 jQuery - Owl Carousel")