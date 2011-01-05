# ajaxyDialog jQuery UI widget

A jQuery UI widget that can be applied to hyperlinks or forms, and causes the destination of the hyperlink or form to be loaded in a jQuery UI Dialog. 

This is new, there are probably some bugs, but it's working well for me. 

## Background and Summary

The [JQuery UI dialog](http://jqueryui.com/demos/dialog/) is awesome. What I ended up doing most with it was taking a hyperlink or form on the page, and unobtrusive-style making it so a click on that hyperlink (or submit on that form) would not cause a page reload, but would instead load the destination in a jQuery UI Dialog. 

This isn't 'ajax' exactly, it's more ajah (HTML rather than XML), so let's call it ajaxy. 

This isn't too hard to do, but after doing it several times, and realizing the edge and exceptional cases to be taken care of, I decided to abstract the pattern into a widget of its own. One tricky thing the widget does is, after you've loaded a dialog, all (or some) hyperlinks or forms in the loaded content can easily be similarly ajaxyDialog-ified. 

ajaxyDialog widget will:

* Magically change the hyperlinks or forms of your choice to load their destination/results in a jquery UI dialog set up with options of your choice. Forms will be sumbitted with the http method they use. 
* Change mouse pointer to load-style when loading, and back to auto when done. 
* Remove the first h1,h2,etc (or custom selector) from loaded content, and make it the Dialog title. Can be turned off. 
* Handle .ajax load errors by displaying an error in the dialog, or with a custom callback
* Gives you a callback to prevent load after examining the xhr response or returned content
* Make hyperlinks and forms in the loaded content also ajaxDialog-ified. You can turn off this behavior or supply a custom selector for forms and hyperlinks to apply ajaxDialog to. 
* Gives you methods and callbacks and options you'd want for a well-behaved flexible widget. 

*note modal restriction*: Right now, ajaxyDialog insists on creating only modal dialogs. It will re-use the same div container and dialog widget for all ajaxyDialogs on a page. This met my actual use cases (I think the times when a modeless dialog in a web page are appropriate UI are pretty rare), and it is somewhat more complicated to make things work efficiently with modeless dialogs. However, the widget could be expanded to handle modeless at a later date. 


## Examples

TODO: Actual live example/demo

Can mix and match hyperlinks and forms, no problem

    $("a.something, form.something_else").ajaxyDialog();

jQuery UI dialogs default to 300 pixels wide and don't do auto-width, so you might want to set a width. 

    $(".something").ajaxyDialog({
       width: 500;
    });

Any other JQuery UI Dialog, including callbacks on the dialog widget itself (wait, not entirely sure this will work properly actually), can be passed in too, on creation or later as options:
    
    # all these are actually Dialog options
    $("a.something").ajaxyDialog({
        width: 500,
        draggable: false,
        beforeClose: function() {
           alert("closing me");
        }
     });
     # More dialog options, after creation, no problem
     $("a#specific_one").ajaxyDialog("option", "buttons", {Ok: function() { $(this).dialog("close"); }});


Access the dialog container, and it's dialog widget, to do things to it, like calling methods or binding events. 

    $("a.blue").ajaxyDialog("dialogContainer").dialog("open");
    $("a.blue").ajaxyDialog("dialogContainer").bind( "dialogbeforeclose", function(event, ui) {
          ...
    });

A couple other useful methods on the ajaxyDialog itself:

    # triggers a load of content in a dialog that will be opened, with options already setup
    $("a.red").ajaxyDialog("open");  
    # close the associated dialog if it's open....
    $("a.red").ajaxyDialog("close");

A couple callbacks of our own

    $("a").ajaxyDialog({
       error: function(event, ui) {
           alert("Problem loading " + ui.url);
           alert(ui.xhr.status);
           #return false to prevent default error display. We
           #can't be sure if the dialog is already open or not,
           #so let's close it. 
           $(this).ajaxyDialog("close");
           return false;
       },
       beforeDisplay: function(event, htmlString) {
           # htmlString is the html returned and about to be loaded
           # in the dialog. Still in string form, sorry. Return false
           # if you don't want to load it. 
       }
    });

## Getting 'live' behavior

Sometimes you want to have ajaxyDialog behavior on links or forms that may not be present on the page at load time. How do you use "live" or "delegate" type behavior to apply a widget?  The technique below works. (Might try to extract this into a built in method, not sure exactly how that'd work). 

      $("a.red").live("click", function() {
          if( ! $(this).data("ajaxyDialog")) {            
            $(this).ajaxyDialog().ajaxyDialog("open");
            return false;
      });

## Options
  * extractTitleSelector
    
    default: "h1, h2, h3, h4, h5"

    First item in loaded content matching the selector will be REMOVED from dom, and it's text will be set as dialog    title. Set to 'false' to turn off this behavior altogether. 

  * chainAjaxySelector
    
    default: "a:not([target]), form:not([target])"

    Elements matching this selector in content loaded into the dialog will also have ajaxyDialog applied to them, with the same options as the 'parent'.  The default is any hyperlinks or forms without targets set.  Set a different selector that selects only particular hyperlinks or forms, or turn off by setting to 'false'. 

  * closeDialogSelector
 
    default: "a.dialog-close"

    Anything in loaded content matching this selector will close the dialog it's in. Set to 'false' to disable, or use the selector of your choice. 

  * ALL JQuery UI Dialog options.  

    ANY other option will be passed to the associated Dialog widget when it's opened. 

## Events

 * beforeDisplay(null-event, html_string)

   called BEFORE the loaded content is inserted into the dialog container. Return false to stop it from being inserted. You can't be sure if the dialog is currently opened or closed, so if you call false best to close it if you want it closed:   `$(this).ajaxyDialog("close");`

   This event and how it works is a bit sketchy, subject to change for real use cases. 

 * error(event, ui)
  
   If the .ajax call made to load external content resulted in an http error, by default ajaxyDialog will display an error message in a Dialog.  This method is called before that happens, and by returning false can prevent that from happening.  The custom data is an object including keys uri, xhr, statusMessage. 

   You can't be sure if the host dialog was already open or not when the error occured loading more content, so best to close it if you want it closed on custom error:  `$(this).ajaxyDialog("close");`

## Methods

  * .ajaxyDialog("open")

    Load the content and open a dialog to display it in. If dialog was already open, will still trigger a reload of content. 

  * .ajaxyDialog("close")

    Close the associated dialog. 

  * .ajaxyDialog("dialogContainer")

    Return the <div> that has .dialog() applied to it for displaying content. Current always-modal behavior means this is always the same div, but that may change.  

    Can be useful for calling methods on the dialog, binding events on the dialog, or custom loading content in it with .html(). 

         .ajaxyDialog("dialogContainer").dialog("isOpen");

         .ajaxyDialog("dialogContainer").html("my own content");

          $("a.blue").ajaxyDialog("dialogContainer").bind( "dialogbeforeclose", function(event, ui) {
                ...
          });

