---
layout: post
title:  "JSON response for remotipart (1.3.0)"
date: 2016-10-10 19:00:00
---

We are using the gem [remotipart](https://github.com/JangoSteve/remotipart) on [PaperSpider](https://www.paperspider.io/) to allow document upload using AJAX. This allow us to upload document in the background, without full page reload.

For now, the "NewDocument" form contains a file upload form as a starting point, and we listen to the ajax success event to initialize the "React form".


By default the format is `:format => 'js'`. We got a bit of trouble to find how to get a JSON response from remotipart. You need use the option `data: { type: :json }` when creating your form.


```erb
<%= form_for @document, html: {:class => "form", :multipart => true }, data: { type: :json }, :remote => true do |f| %>
```

Then in JavaScript, you can get the JSON payload from the response.

```javascript
$('form').on("ajax:remotipartComplete", function(e, data) {
    var jsonDoc = JSON.parse(data.responseText);
});
```
