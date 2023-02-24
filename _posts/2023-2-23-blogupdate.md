---
layout: post
title: "blog update"
---

i switched to jekyll because its a lot easier to use than completely hand written html + directories. 
for anyone (morbidly) interested, heres the code i wrote for my method of generating blog pages from markdown files on the client side

first, heres what that `data.json` file looked like:
```
{
  "posts": [
    {
      "file": "test.md",
      "name": "Test",
      "date": "feb 20, 2023",
      "update": "",
      "description": "a test of the blog"
    }
  ]
}
```

this ran in the "blog index" page:
```
;(function(){
	var target = document.getElementById('posts');
	fetch("blog/data.json").then(function(e){
		return e.json();
	}).then(function(e){
		for (var i = 0; i < e.posts.length; i++) {
			var element = document.createElement("div");
			// styling
			element.classList.add('box');
			element.style.marginTop = '1em';

			element.innerHTML = 
				"<a href=\"./blogpost?i=" + 
				e.posts[i].file + "\"><h2>" + 
				e.posts[i].name + 
				"</h2></a> <hr> <br>" + 
				e.posts[i].description;
			target.appendChild(element);
		}
	});
})();
```

pretty self explanitory I hope.

this ran in the "blog post" page:
```
// note: showdown.min.js is loading like <script src="./showdown.min.js"></script> before this.
// additionally, the url has a parameter named "i" storing the name of the target markdown file, like ".../?i=test.md"
;(function(){
	var target = document.getElementById('content');
	target.innerHTML = "loading...";

	var converter = new showdown.Converter({
	strikethrough: true,
	customizedHeaderId: true,
	omitExtraWLInCodeBlocks: true,
	tasklists: true,
	});
	
	fetch("blog/data.json").then(function(e){
		return e.json();
	}).then(function(e){
		var pos = (new URLSearchParams(window.location.search)).get("i");
		var filedata = 'error';
		for (var i = 0; i < e.posts.length; i++) {
				if (e.posts[i].file == pos) {
				filedata = e.posts[i];
				break;
			}
		}

		fetch("blog/" + filedata.file).then(function(e){
			return e.text();
		}).then(function(e){
			var prestr = "posted on " + filedata.date;
			if (filedata.update && filedata.update != "") prestr += ", updated on " + filedata.update;
			target.innerHTML = "<span style=\"font-size:0.7em\">" + prestr + "</span>" + converter.makeHtml(e);
		});
	
	});
})();
```

also self-explanitory. 

that's all I have for now, I felt that it was a silly method worth sharing. <br/>
have a nice day :3
