---
title: What are Vue talking about
author: deonfourie
---

A progressive framework focussed on the view layer. Easy to pick up and integrate with existing projects.

I do not want to simply summarize Vue’s own documentation, but instead present my personal findings. You can read Vue’s documentation [here](https://vuejs.org/v2/guide/#What-is-Vue-js), as well as a great [comparison article](https://vuejs.org/v2/guide/comparison.html). The guide is extensive.
<!--more-->

####I like Vue
There are always different ways to do different things. Why use Vue is you can use React? Why use React if you can use Angular. Angular also have components. React can do everything Vue does. This is where personal choice enters the fray.

Angular was my first proper JavaScript Framework, and so formed a lot of my thinking when it comes to how JavaScript Frameworks should work. It is wonderful having a Framework that has most everything you need. I then continued on to React. That took a while to get orientated and to start [thinking in React](https://reactjs.org/docs/thinking-in-react.html). It was great. I found it much more free experience than Angular. I could approach problems more in my own way and it felt like I could divide up the code to a greater extent. Really get into the fine grain.

With that level of separation I found that a lot of opinions came about as well. Everyone’s way was the best way. Whereas multiple Angular projects have a greater tendency to look similar in the code, React was different every time. Not necessarily bad though, as it would expose me to many different ways of doing the same thing. But it certainly slows things down initially and constantly made me wonder if there was a better way, than the way I was handling things. As much as I loved the increased freedom, I was looking for ways contain or order the way I do things. 

I think this is why I like Vue.js. It has the freedom of React with some of the order of Angular. Now obviously I am being a bit over dramatic in my approach, but I am trying to put across my experience with all three JavaScript players. I will try to go into more detail on some of my findings.

####HTML and JSX
Maybe it’s because I’m older, but I enjoy coding in my familiar HTML a little bit more than React’s JSX. Although I am fine with generated HTML, I always get a bit uncomfortable when I cannot control the output of HTML. There always seems to be too many divs. I like my HTML clean and simple. That way I avoid having to do weird things in my CSS after I find out that 2 new divs have inserted themselves into specificity path.

####Separating Concerns
Coming from a not so coding background, I found the way the export statement was handled to be particularly useful. Separating it out as they do makes a lot of sense to me and helped me visualize and handle my code design much better. I firmly believe that using Vue.js is a great way to train up new Front-end developers. Just a quick example from a project I created,  my export statement looked something like this:


```
export default {
        name: "MainPage",

// import any component I want to use
        components: {
            ComponentOne,
            ComponentTwo
        },

// all data properties go to the reactivity system. when the values change, the view will “react”, updating to match the new values
        data: function() {
            return {
                isLoading: false,
                items: [],
            };
        },

// lifecycle hook, called after the instance has been mounted, one of a few
        mounted: function() {
            this.isLoading = true;
            ItemService.getItems()
                .then(response => (this.items = response.data))
                .then(() => (this.isLoading = false));
        },

// logic cached based on dependencies
        computed: {
            newItems: function() {}
       },

// regular methods not chached on dependancies
       methods: {
           standardMethod: function() {}
       }
};
```

To be continued...
