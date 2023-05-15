---
title: "黑暗模式(dark mode)設計與Hugo實作"
date: 2023-03-22T11:34:07+08:00
draft: false
slug: "tutorials/dark-mode-tips-hugo"
thumbnail: "img/thumbnail/024.webp" # Thumbnail image
description: ""
categories:
  - "程式語言"
tags:
  - "Hugo"
  - "Web"
  - "CSS"
  - "JavaScript"
# Theme-Defined params
lead: "" # Lead text
authorbox: true # Enable authorbox for specific page
pager: false # Enable pager navigation (prev/next) for specific page
toc: true # Enable Table of Contents for specific page
mathjax: true # Enable MathJax for specific page
---
紀錄一下網站增加黑暗模式功能踩到的一些坑、一些前端顯示的細節還有在Hugo上實作。
<!--more-->

### 前言
黑暗明亮模式轉換，應該是現在不管是網頁還是App都必備的功能，平常在使用一些手機App，若沒有能讓我選擇轉換的地方，心理不免納悶這團隊也太不人性化了吧。自從開始經營網站後，很不幸的，內建的主題沒有支援此功能，所以只好自己弄一個。原本的理解很簡單，就是多一份dark mode專屬的css，藉由轉換每個css的class進而轉換畫面使用的css達到修改成dark mode的目的。  
畫面能成功轉換成黑暗模式，但遇到的問題是在頁面跳轉時會出現短暫的白色閃爍。經歷各種的嘗試發現改為使用`prefers-color-scheme: dark`的方式就能解決此問題。  

### 原理
主要原理就是藉由CSS `prefers-color-scheme` 媒體查詢和 JavaScript 來監聽操作系統或瀏覽器的顏色模式設置，並在頁面上動態地添加或刪除 `dark-mode` 類。這意味著在頁面跳轉時，頁面不會立即切換到預設的亮色模式或黑暗模式，而是保持當前的顏色模式狀態，從而避免了短暫的白色閃爍。  

### 轉換模式按鈕
首先要先設一個可以點選轉換模式的按鈕，樣式可以自己選擇，這裡我是選擇用Unicode對應的太陽與月亮的符號來充當按鈕的樣子。  
位置可以放在`根目錄\themes\Mainroad\layouts\baseof.html`
```HTML {linenos=inline}
<span class="sun-btn">&#x2600;</span>
<span class="moon-btn">&#x263D;</span>
```
然後在style.css增加按鈕的樣式與黑暗模式的樣式。 
### CSS variable 應用
CSS的部分因為不同網站的配置都不相同還是需要自己去設置對應的黑暗模式配色，為了更好的控管CSS，可以使用`CSS variable`(CSS 自訂屬性)，是一種在 CSS 中定義和使用的值，可以在整個文件中重複使用。它們以雙減號（--）為前綴加上變數名稱來命名。

範例:  
```CSS {linenos=inline}
:root {
  --main-color: #ff0000;
}

h1 {
  color: var(--main-color);
}
```
在 :root 偽類中定義了一個名為 `--main-color` 的CSS變數，並將其設置為紅色。然後，在h1元素中，使用`var()`函數來引用這個變數，將其值應用於 color 屬性。  
CSS 變數的優點之一是它們可以在不同的元素和選擇器之間共享，並且可以通過 JavaScript 動態更改它們的值。這使得 CSS 變數成為一種非常強大和靈活的工具，可以用於實現動態和可維護的樣式表。  

以下是我的主題使用的CSS，為了方便管理，與增加黑暗模式的樣式，先新增:root 偽類中定義各個CSS variable，然後把整個css.style裡有用到的都先替換成這些CSS變數，這樣就可以透過在dark mode類直接修改變數值達到統一轉換的效果。  
```CSS {linenos=inline}
:root {
	--bodyBG: #f7f7f7;
	--wrapperBG:#ffffff;
	--wrapperBG2:#f5f5f5;
	--text-color: #000000;
	--codeBG:#f6f8fa;
  }  
```
然後對明暗轉換的按鈕客製化一下樣式
```CSS {linenos=inline}
/* Light Mode */
.sun-btn {
	display: block;
	font-size: 2rem;
	color: #ffc533;
  }
  
.moon-btn {
	display: none;
  }
/* Dark Mode */
 .dark-mode .sun-btn{
	display: none;
 }
 
 .dark-mode .moon-btn {
	 display: block;
	 font-size: 2rem;
	 color: #fff9be;
   }
 
```

以下則是黑暗模式下的CSS，可以看到實際要改的其實不多，因為都用CSS variable取代了。  
```CSS {linenos=inline}
  /*dark-mode styles*/
.dark-mode {
	--bodyBG:#2b2b2b;
	--wrapperBG: #212121;
	--wrapperBG2: #212121;
	--text-color: #C5C5C5;
	--codeBG:#424242;
	box-shadow: none;
  } 
.dark-mode span.nav-indicator{
	background-color: #ACD6FF !important;
  } 
.dark-mode li{
	color: var(--text-color);
  }
```
### 降低白底亮度
如果網頁上會放一些影音或是圖片，若是白底的可能會在黑暗模式下異常刺眼，可以使用此屬性降低在黑暗模式下的亮度。
```CSS {linenos=inline}
.dark-mode iframe, 
.dark-mode img,
.dark-mode video {
  filter: brightness(0.9);
}
```
### 按鈕監聽、localStorage存放用戶偏好模式
接下來就是使用JS實現以下功能。  
位置放在`根目錄\themes\Mainroad\layouts\baseof.html`
1. 當頁面加載時，檢查 localStorage 中是否存在 "dark-mode" 設置。如果存在，則根據設置狀態啟用或禁用深色模式。
2. 監聽系統設置是否切換到深色模式（通過 window.matchMedia 創建 MediaQueryList 對象來實現）。
3. 當用戶點擊「太陽」或「月亮」按鈕時，切換深色模式的狀態，並同時更新 localStorage 的值。

```JS {linenos=inline}
let darkModeState = false;

const sun = document.querySelector('.sun-btn');
const moon = document.querySelector('.moon-btn');

// MediaQueryList object
const useDark = window.matchMedia("(prefers-color-scheme: dark)");

// Toggles the "dark-mode" class
function toggleDarkMode(state) {
  document.documentElement.classList.toggle("dark-mode", state);
  darkModeState = state;
}

// Sets localStorage state
function setDarkModeLocalStorage(state) {
  localStorage.setItem("dark-mode", state);
}

// Initial setting
toggleDarkMode(localStorage.getItem("dark-mode") == "true");

// Listen for changes in the OS settings.
// useDark.addListener((evt) => toggleDarkMode(evt.matches));
  useDark.addEventListener('change', (evt) => {
    toggleDarkMode(evt.matches);
  });


// Toggles the "dark-mode" class on click and sets localStorage state
sun.addEventListener("click", () => {
  darkModeState = !darkModeState;

  toggleDarkMode(darkModeState);
  setDarkModeLocalStorage(darkModeState);
});
moon.addEventListener("click", () => {
  darkModeState = !darkModeState;

  toggleDarkMode(darkModeState);
  setDarkModeLocalStorage(darkModeState);
});
```

### 心得
其實我認為此方法，跟我原本也是使用JS在頁面上動態地添加或刪除 `dark-mode` 類的方法大同小異，實在是不清楚為何就能解決頁面跳轉時會出現短暫的白色閃爍問題，希望有前端專業的人可以補充。  


{{< br 8  1>}}
> Reference

- [Dark Mode - The prefers-color-scheme Website Tutorial](https://www.ditdot.hr/en/dark-mode-website-tutorial "Dark Mode - The prefers-color-scheme Website Tutorial")

