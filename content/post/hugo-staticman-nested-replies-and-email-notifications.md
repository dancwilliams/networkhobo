---
title: "Hugo + Staticman: Nested Replies and E-mail Notifications"
date: 2017-12-30T20:47:00-06:00
draft: false
tags: [ "go", "hugo", "staticman", "comments" ]
categories: [ "hugo", "staticman" ]
---

In this post I want to cover the steps I went through to get Staticman nested comments and e-mail notifications working in Hugo.

**Disclaimer: I am new to Hugo, Go Templating, JavaScript, and all the bits and pieces used in this write-up.  I am sure there are more efficient methods to achieve these results.  Please provide any feedback in the comments or feel free to issue a pull request.  Thanks!**

<!--more-->

_Below is a list of the technology I use for this blog:_

* [_Hugo_](https://gohugo.io) _- Version 0.31.1_
* [_Beautiful Hugo Theme_](https://github.com/halogenica/beautifulhugo)
* [_Staticman_](https://staticman.net)
* [_Netlify_](https://www.netlify.com)
* [_Network Hobo GitHub Repo_](https://github.com/dancwilliams/networkhobo/)

_I do not go through all of the steps for basic Staticman setup and API registration.  This is covered in the great [Staticman Documentation](https://staticman.net/docs/)._

### In the beginning...

After years of hosting this blog on Wordpress I decided I wanted to make a move to a more flexible option.  I started looking at static site generation and all of the various options.  I finally decided to take a crack at Hugo.

I went in search of a clean template for my new Hugo site.  After much searching I decided to go with [Beautiful Hugo](https://github.com/halogenica/beautifulhugo).  At the time, I started using Beautiful Hugo it only supported [Disqus](https://disqus.com) for commenting.  I knew I wanted to take this opportunity to gain control of all of my data, so I started adapting the Beautiful Hugo theme to use [Staticman](https://staticman.net) for commenting.

* _Note: As of November 21, 2017, the Beautiful Hugo theme now natively supports Staticman comments ([PR#99](https://github.com/halogenica/beautifulhugo/pull/99)).  This is basic commenting without replies or e-mail notification.  I plan to work these features into the theme.  Just wanted to get my logic down here before I lose it :-)._

I started my work with Staticman by using the [Hugo example site from Staticman themselves](https://hugo.staticman.net).  [Eduardo Bouças](https://github.com/eduardoboucas) (creator of Staticman) did a great job putting this small example site together. I used the [post-comments.html](https://github.com/eduardoboucas/hugo-plus-staticman/blob/master/themes/hugo-type-theme/layouts/partials/post-comments.html) partial from this site as the base to begin exploring Staticman.  I broke this out into two partials, one for parsing the comments to show and one for the comment form itself.  I did this so that later I could add a feature to lock comments on a post if needed.

Now I need to investigate how to handle e-mail notifications.  That is when I stumbled upon [this gold mine of information](https://mademistakes.com/articles/improving-jekyll-static-comments/) from [Michael Rose](https://github.com/mmistakes).  This article and Michael's [GitHub repo](https://github.com/mmistakes/made-mistakes-jekyll) really helped me walk through the logic of e-mail replies as well as how he handled nesting replies.  What a **GREAT** source of information!

### Putting it all together:

The first thing I wanted to be sure of was that I did not statically configure anything.  I wanted Staticman to act just like another available comment module within the template.  Therefor I added the following pieces of information to the ```Params``` section of my [```config.toml```](https://github.com/dancwilliams/networkhobo/blob/master/config.toml) file:

```yaml
staticman_api = "https://api.staticman.net/v2/entry/dancwilliams/networkhobo/master/comments" #Add staticman API URL to enable staticman comments
```

Then I added some logic to the [```layouts/_default/single.html```](https://github.com/dancwilliams/networkhobo/blob/master/layouts/_default/single.html) file to look for the staticman_api Site.Param, and if existed to add the ```post-comments.html``` partial:

```html
{{ if (.Params.comments) | or (and (or (not (isset .Params "comments")) (eq .Params.comments nil)) (.Site.Params.comments)) }}
  {{ if .Site.DisqusShortname }}
    <div class="disqus-comments">
      {{ template "_internal/disqus.html" . }}
    </div>
  {{ end }}
  {{ if (.Site.Params.staticman_api) }}
    {{ partial "post-comments" . }}
  {{ end }}
{{ end }}
```

I then moved on to the creation of the [```layouts/partials/post-comments.html```](https://github.com/dancwilliams/networkhobo/blob/master/layouts/partials/post-comments.html) partial:

```html
<section class="post-comments">
  <h3>Comments</h3>

  {{ $.Scratch.Add "hasComments" 0 }}
  {{ $entryId := .File.BaseFileName }}

  {{ range $index, $comments := (index $.Site.Data.comments $entryId ) }}
    {{ $.Scratch.Add "hasComments" 1 }}
    {{ if not .reply_to }}
      <div class="post-comment">
        <div class="post-comment-header">
          <img class="post-comment-avatar" src="https://www.gravatar.com/avatar/{{ .email }}?s=100">
          <p class="post-comment-info"><strong>{{ .name }}</strong><br>{{ dateFormat "Monday, Jan 2, 2006" .date }}</p>
        </div>
        {{ .body | markdownify }}
      </div>
      <div class="comment__reply">
        <a id="{{ ._id }}" class="btn-info" href="#comment-form" onclick="changeValue('fields[reply_to]', '{{ ._id }}')">Reply to {{ .name }}</a>
          </div>
      {{ partial "comment-replies" (dict "entryId_parent" $entryId "SiteDataComments_parent" $.Site.Data.comments "parentId" ._id "parentName" .name "context" .) }}
    {{ end }}
  {{ end }}       


  {{ if eq ($.Scratch.Get "hasComments") 0 }}
    <p>Nothing yet.</p>
  {{ end }}

  {{ partial "comment_form" . }}

</section>
```

You can see that most of this is pulled from the Staticman Hugo example.  I did remove some of the looping and dataset reading that I found to be excessive.  You will also see that I added some logic to only present comments that were not replies and to add a button for replies to those comments.  

I had to add a piece of JavaScript to the reply button to set the value of the ```fields[reply_to]``` hidden input.  This field is set to the ```._id``` value of the current parent comment.  This allows for the fields to be properly populated by Staticman:

_Located at the bottom of [```main.js```](https://github.com/dancwilliams/networkhobo/blob/master/static/js/main.js)._

```javascript
// Added function to change value onclick
function changeValue(elementName, newValue){
  document.getElementsByName(elementName)[0].value=newValue;
};
```

Here are examples of the two types of comment YAML files that are created:

#### Parent Comment:

_In the parent comments you will see that the ```reply_to``` field is blank._

```yaml
_id: 74ea2730-ed18-11e7-96e3-b9aaffd0f2aa
_parent: >-
  2013-12-23-cisco-unified-communications-manager-unity-connection-sftp-emergency-backup-to-mac-os-x-over-the-internet
reply_to: ''
name: Dan
email: 9162d0c5aca33e7e4c8ec6fc3d44f541
body: Test comment 1
date: '2017-12-30T04:18:15.955Z'
```

#### Child Comment:

_In the reply comments you will see that the ```reply_to``` field is populated with the ```._id``` value from the parent comment._

```yaml
_id: f7be83c0-ed1a-11e7-96e3-b9aaffd0f2aa
_parent: >-
  2013-12-23-cisco-unified-communications-manager-unity-connection-sftp-emergency-backup-to-mac-os-x-over-the-internet
reply_to: 74ea2730-ed18-11e7-96e3-b9aaffd0f2aa
name: Reply Tester
email: 53d8e4904144b75f9ada3862b6ebafae
body: Testing a reply to Dan’s comment
date: '2017-12-30T04:36:14.421Z'
```

This ```reply_to``` field is what is used to differentiate parent and child comments.

After a "parent" comment is printed the [```layouts/partials/comment-replies.html```](https://github.com/dancwilliams/networkhobo/blob/master/layouts/partials/comment-replies.html) partial is called.  This partial looks much like the ```post-comments``` partial:

```html
{{ range $index, $comments := (index $.SiteDataComments_parent $.entryId_parent ) }}
  {{ if eq .reply_to $.parentId }}
    <div class="post-comment-reply">
      <div class="post-comment-header">
        <img class="post-comment-avatar" src="https://www.gravatar.com/avatar/{{ .email }}?s=100">
        <p class="post-comment-info"><strong>{{ .name }}</strong><br><i><small>In reply to {{ $.parentName }}</i></small><br>{{ dateFormat "Monday, Jan 2, 2006" .date }}</p>
      </div>
      {{ .body | markdownify }}
    </div>
  {{ end }}
{{ end }}
```

When this partial is called a dictionary of variables are passed:

```html
{{ partial "comment-replies" (dict "entryId_parent" $entryId "SiteDataComments_parent" $.Site.Data.comments "parentId" ._id "parentName" .name "context" .) }}
```

These variables allow the replies partial to match the ```reply_to``` field against the parent comment ```._id``` field (variable ```parentId```) within this partial.  Once a match it hit the same process is used for presenting the comment, with the slight addition of adding a text blurb to say this is a reply to the name of the parent comment author.

After all of the comments and replies for a particular post have been processed, the [```layouts/partials/comment_form.html```](https://github.com/dancwilliams/networkhobo/blob/master/layouts/partials/comment_form.html) partial is called:

```html
<section class="comment_form">
  <a id="comment-form"></a>
  <h3>Say something</h3>

  <form class="post-new-comment" method="POST" action="{{ .Site.Params.staticman_api }}">
    <input type="hidden" name="options[redirect]" value="{{ .Permalink }}#post-submitted">
    <input type="hidden" name="options[redirectError]" value="{{ .Permalink }}#post-error">
    <input type="hidden" name="options[entryId]" value="{{ .File.BaseFileName }}">
    <input type="hidden" name="options[slug]" value="{{ .Permalink }}">
    <input type="hidden" name="options[origin]" value="{{ .Permalink }}">
    <input type="hidden" name="options[parent]" value="{{ .File.BaseFileName }}">
    <input type="hidden" name="fields[reply_to]" value="">
    <input type="hidden" name="options[reCaptcha][siteKey]" value="{{ .Site.Params.recaptcha_siteKey }}">
    <input type="hidden" name="options[reCaptcha][secret]" value="{{ .Site.Params.recaptcha_secret }}">

    <fieldset>
      <input name="fields[name]" type="text" class="post-comment-field" placeholder="Your name">
    </fieldset>

    <fieldset>
      <input name="fields[email]" type="email" class="post-comment-field" placeholder="Your email address">
    </fieldset>

    <fieldset>
      <textarea name="fields[body]" class="post-comment-field" placeholder="Your message. Feel free to use Markdown." rows="10"></textarea>
    </fieldset>
    
    <fieldset>
      <div class="notify-me">
        <input type="checkbox" id="comment-form-reply" name="options[subscribe]" value="email">
        Send me an email when someone comments on this post.
      </div>
    </fieldset>

    <fieldset>
      <div class="g-recaptcha" data-sitekey="{{ .Site.Params.recaptcha_siteKey }}" data-callback="enableBtn"></div>

      <input type="submit" class="post-comment-field btn" value="Submit" id="submit_button">
    </fieldset>

  </form>

  <script async src='https://www.google.com/recaptcha/api.js' ></script>


  <script type="text/javascript">
    document.getElementById("submit_button").disabled = true;
  </script>
  
  <script type="text/javascript">
    function enableBtn(){
       document.getElementById("submit_button").disabled = false;
      }
  </script>

  <div id="post-submitted" class="dialog">
    <h3>Thank you</h3>
    <p>Your post has been submitted and will be published once it has been approved.</p>
    {{ if (.Site.Params.githubPullURL) }}
      <p><a href="{{ .Site.Params.githubPullURL }}">Click here</a> to see the pull request you generated.</p>
    {{ end }}
    <p><a href="#" class="btn">OK</a></p>
  </div>

  <div id="post-error" class="dialog">
    <h3>OOPS!</h3>
    <p>Your post has not been submitted.  Please return to the page and try again.  Thank You!</p>
    <p><a href="#" class="btn">OK</a></p>
  </div> 

</section>
```

This form was also taken from the Staticman Hugo example site and modified in a few ways.  Multiple hidden input fields were added to support replies as well as subscribing to comment e-mail notifications.

To support e-mail replies we had to add a checkbox to the comment form:

```html
<fieldset>
  <div class="notify-me">
    <input type="checkbox" id="comment-form-reply" name="options[subscribe]" value="email">
        Send me an email when someone comments on this post.
  </div>
</fieldset>
```

When this checkbox is selected it sets the value of the input ```options[subscribe]``` to the given e-mail address.  This option is used by Staticman for creating and/or populating the mailing lists in [Mailgun](https://www.mailgun.com).

There is also logic to support [reCaptcha](https://www.google.com/recaptcha/).  The reCaptcha authorizationis used in two ways:
* It is used by the backend Staticman server (setting in [```staticman.yml```](https://github.com/dancwilliams/networkhobo/blob/master/staticman.yml))
* It is used by the submit button on the comment form.  The submit button remains disabled until the reCaptcha test is passed.

Once the form is filled out and the comment is submitted one of two things could happen.  Within the ```comment_form``` partial there is a dialog for submission success and one for failure.

#### Success Dialog:

```html
<div id="post-submitted" class="dialog">
  <h3>Thank you</h3>
  <p>Your post has been submitted and will be published once it has been approved.</p>
  {{ if (.Site.Params.githubPullURL) }}
    <p><a href="{{ .Site.Params.githubPullURL }}">Click here</a> to see the pull request you generated.</p>
  {{ end }}
<p><a href="#" class="btn">OK</a></p>
  </div>
```

If the Staticman API call comes back as successful, this dialog is presented.  If the configuration parameter ```githubPullURL``` is set, a link will be presented to view the pull request created by Staticman.  The OK button will take the user back to the beginning of the post.

#### Failure Dialog:

```html
<div id="post-error" class="dialog">
  <h3>OOPS!</h3>
  <p>Your post has not been submitted.  Please return to the page and try again.  Thank You!</p>
  <p><a href="#" class="btn">OK</a></p>
</div> 
```

If the Staticman API call reports a failure, this dialog is presented.  The OK button will take the user back to the beginning of the post.

### Summary

This has been a fun project that I have been working on for some time.  I have learned a lot about all of the components of Hugo and hope that some people will show me better/cleaner/more efficient ways of handling this.

In this post I tried to cover everything from my notes, but if I forgot something I will be sure to update.

Thanks for taking the time to read this and I hope you find it helpful.  Feel free to comment here and/or reach out to me on Twitter ([@dancwilliams](https://twitter.com/dancwilliams)) if you would like to discuss any of this.

**_Thanks again!_**

### Lagniappe

When I was migrating from Wordpress to Hugo + Staticman I wanted to be sure to migrate all of my existing comments.  There was a lot of good stuff in there and I didn't want to lose it.  Since Wordpress provides a full export of your site in an XML document, I wrote a little Python script to extract the comments and create the YAML files.  It even preserves the nesting. [Here is a link to the script in GitHub](https://github.com/dancwilliams/wordpress_to_staticman_comments).