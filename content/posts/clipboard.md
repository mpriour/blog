---
title: "Using the Clipboard APIs"
date: 2017-09-11
---

If you've ever wanted a way for a user to copy information in a different format or control what exactly gets copied from areas of your page, the the [Cliboard API](https://developer.mozilla.org/en-US/docs/Web/API/ClipboardEvent) & the [execCommand API](https://developer.mozilla.org/en-US/docs/Web/API/Document/execCommand) are very useful. However reading specs and [limitations](http://caniuse.com/#feat=clipboard) between different browsers makes it seem more complicated and limiting to use than it is.

Let say you have contact information that you want to display in a nice card layout on the screen. However, that formatted display doesn't copy well into other systems or uses. We could provide users with a link to copy the contact data in a CSV format. That way it could be pasted into a text file or a spreadsheet or imported in Outlook.

### Our contact data

```json
{"contacts":[
    {
        "fname": "Aly",
        "lname": "Money",
        "email": "a.money@gmail.com",
        "phone": "555-333-1234"
    },
...
    {
        "fname": "Dot",
        "lname": "Matrix",
        "email": "dmatrix@yahoo.com",
        "phone": "555-333-4123"
    }
]}
```

Rendered into the following markup for each card

```html
<div data-cid="0" class="card">
    <p class="name">Aly Money</p>
    <p class="email"><a href="mailto:a.money@gmail.com">a.money@gmail.com</a></p>
    <p class="phone">555-333-1234</p>
</div>
<p><a href="#" class="copy-link" data-cid="0">Copy</a></p>
```

To make this work properly we are going to need an input field to actually do the copying from. Copying from a selected, focused input field is the only cross-browser way to make this work. So we also add

```html
<div id="form-container">
    <textarea id="copy-area" tabindex="-1"></textarea>
</div>
```

But we don't actually want to see this field or have users interact with it. For the copy function to work, the field also has to be visible, so we can't use `display: none` to hide this field. We have to push it offscreen.

```css
#form-container{
    position: absolute;
    left: -5000px;
} 
```

Here is the jQuery based copy function we are using in this example. You can see a vanilla js one in [this post](https://www.sitepoint.com/javascript-copy-to-clipboard/) by Craig Buckler of SitePoint.

```js
function copyValues(id, val){
    //set value in hidden text area and select it.
    var $area = $("#"+id).val(val).select();
    try {
        // copy text
        document.execCommand('copy');
        $area.blur();
    } catch (err) {
        //show in an alert just in case execCommand doesn't work
        alert($area.val());
    }
}
```

Hook up the handlers for the "Copy" buttons

```js
//Hook up copy button handlers
$(".copy-link").on("click", function(evt){
    var contact = data.contacts[evt.target.dataset.cid];
    var csv = "First Name, Last Name, E-mail, Work Phone\n" +
        [contact.fname,contact.lname,contact.email,contact.phone].join(",");
    copyValues("copy-area", csv);
    console.log("Copied\n" + csv);
});

$(".copy-all-link").on("click", function(evt){
    var csv = ["First Name, Last Name, E-mail, Work Phone"];
    var records = data.contacts.map(function(c){
        return [c.fname,c.lname,c.email,c.phone].join(",");
    });
    var vals = csv.concat(records).join("\n")
    copyValues("copy-area", vals);
    console.log("Copied\n" + vals);
})
```

Now what if want to provide specialized copy behavior for keyboard (Ctrl+C / Cmd+C) or context menu copy commands? Then we need to capture the "copy" event. One thing we could do is simply hookup an event listner for the `copy` event on the "card" class. However, the user would have to have their mouse over the card when they hit Ctrl+C / Cmd+C. Unless we provided instructions or feedback it would be difficult to know that this interaction was even possible. Also, they would not get a "copy" option in the context menu either. 

This example was already fairly contrived to begin with but it is going to get even more so as we add some "features" to the address cards.  We want the user to be able to tab to focus on the cards and we want a copy option in the context menu. The simplest way to achieve this is to make the cards `contenteditable`. To keep the cards from actually being editable we have to trap keypresses and prevent them unless they have a "meta" key or tab. This function for making the content-editable cards remain read-only is contained in the pen below.

To handle the `copy` event, we simply set a listener for `copy` on the "card" class. We prevent the default event, which wouldn't copy anything anyway if we didn't have some text highlighted. Then attempt to `setData` on the event's `clipboardData`, if we can't do that then we just use the same `execCommand` copy method as before.

```js
//Hook up copy context, etc.. handler
$(".card").on("copy", function(evt){
    evt.preventDefault();
    evt = evt.originalEvent;
    var contact = data.contacts[evt.currentTarget.dataset.cid];
    var csv = "First Name, Last Name, E-mail, Work Phone\n" +
        [contact.fname,contact.lname,contact.email,contact.phone].join(",");
    if(evt.clipboardData && evt.clipboardData.setData){
        evt.clipboardData.setData('text/plain', csv);
    } else {
        copyValues("copy-area", csv);
    }
    console.log("Copied\n" + csv);
});
```

Unfortunately, IE11 is still a problem and this trick doesn't work on it. The `copy` event is never fired. Only the copy links work in IE. Chrome, Firefox, and Safari all work in both modes. You can see the full code in the pen below.

<p data-height="400" data-theme-id="1" data-slug-hash="RZXBme" data-default-tab="js,result" data-user="mpriour" data-embed-version="2" data-pen-title="Using both Copy API's" class="codepen">See the Pen <a href="https://codepen.io/mpriour/pen/RZXBme/">Using both Copy API's</a> by Matt Priour (<a href="https://codepen.io/mpriour">@mpriour</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

