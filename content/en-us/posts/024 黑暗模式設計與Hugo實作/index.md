---
title: "Designing and Implementing Dark Mode in Hugo"
date: 2023-03-22T11:34:07+08:00
draft: false
slug: "tutorials/dark-mode-tips-hugo"
thumbnail: "img/thumbnail/024en.webp" # Thumbnail image
description: ""
categories:
  - "Program"
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
Recording the hurdles encountered while adding dark mode functionality to a website, as well as frontend display tips and implementation on Hugo.
<!--more-->

### Foreword
The switch between dark and light modes is now a must-have feature for both webpages and apps. When using certain mobile apps without the option to switch, it's difficult for me not to wonder why the team didn't consider user-friendliness. Unfortunately, the built-in theme of my website didn't support this feature, so I had to create one myself. Initially, I thought it was simple, just add a dark mode-specific CSS and use class switching to modify the CSS used for the interface to achieve the goal of modifying it to a dark mode.  
Although the interface successfully switched to dark mode, I encountered a problem where there was a brief white flash when navigating to other pages within the same website. After trying various solutions, I found that using `prefers-color-scheme: dark` resolved the issue.

### Principle
The main principle is to use CSS `prefers-color-scheme` media query and JavaScript to monitor the color mode settings of the operating system or browser and dynamically add or remove the `dark-mode` class on the page. This means that when navigating to other pages, the page will not immediately switch to the default light or dark mode, but instead will maintain the current color mode state, thus avoiding the brief white flash.

### Switch Mode Button
First, need to create a button that can be clicked to switch between modes. You can choose your own style for the button, and here I chose to use Unicode symbols for the sun and moon to serve as the appearance of the button.
The location of the code:`Root directory\themes\Mainroad\layouts\baseof.html`
```HTML {linenos=inline}
<span class="sun-btn">&#x2600;</span>
<span class="moon-btn">&#x263D;</span>
```
Next, you can add the style for the button and the dark mode in style.css.   
### CSS variable 
Since the configuration of different websites varies, you will still need to set up the corresponding dark mode color scheme in CSS. To have better control over CSS, you can use `CSS variables`, which are values defined and used in CSS that can be repeated throughout the document. They are named by prefixing a variable name with a double hyphen (--).  

Example:  
```CSS {linenos=inline}
:root {
  --main-color: #ff0000;
}

h1 {
  color: var(--main-color);
}
```
A CSS variable named `--main-color` is defined in the :root pseudo-class and set to red. Then, the var() function is used to reference this variable in the color property applied to the h1 element.   

One of the benefits of CSS variables is that they can be shared across different elements and selectors, and their values can be dynamically changed through JavaScript. This makes CSS variables a very powerful and flexible tool that can be used to achieve dynamic and maintainable style sheets. 

Here is the CSS used in my theme. To facilitate management and add styles for dark mode, I first define various CSS variables in the :root pseudo-class. Then, I replace all the CSS styles used in the css.style with these CSS variables, so that the effect of a unified transition can be achieved by simply modifying the variable values in the dark mode class.    
```CSS {linenos=inline}
:root {
	--bodyBG: #f7f7f7;
	--wrapperBG:#ffffff;
	--wrapperBG2:#f5f5f5;
	--text-color: #000000;
	--codeBG:#f6f8fa;
  }  
```
Then customize the style of the light/dark mode toggle button.  
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

Here is the CSS for the dark mode, and you can see that there are not many actual changes because most of them have been replaced with CSS variables.   
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
### Reduce the brightness of the white background
If there are some media or images on the webpage, it may be too bright in dark mode if they have a white background. In this case, you can use the brightness CSS property to reduce the brightness in dark mode.
```CSS {linenos=inline}
.dark-mode iframe, 
.dark-mode img,
.dark-mode video {
  filter: brightness(0.9);
}
```
### Button monitoring, localStorage store user preference mode
Next, we will use JavaScript to implement the following functionalities.    
The location of the code:`Root directory\themes\Mainroad\layouts\baseof.html`
1. When the page is loaded, check if the "dark-mode" setting exists in the localStorage. If it does, enable or disable the dark mode based on the setting.
2. Listen for changes in the system settings for switching to dark mode (using window.matchMedia to create a MediaQueryList object).
3. When the user clicks the "sun" or "moon" button, toggle the state of the dark mode and update the localStorage value accordingly.

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

### Conclusion
I actually think that this method is similar to the approach I was originally using, which was dynamically adding or removing the `dark-mode` class on the page using JavaScript. I'm not sure why this method seems to solve the issue of a brief white flash on page transitions, and I would appreciate any input from frontend experts. 


{{< br 8  1>}}
> Reference

- [Dark Mode - The prefers-color-scheme Website Tutorial](https://www.ditdot.hr/en/dark-mode-website-tutorial "Dark Mode - The prefers-color-scheme Website Tutorial")

