# CmsLess
_the Content Management System that gets out of your way_

* ultra-lightweight 
* dynamically load pages based on url hash
* keep static web sites [DRY](http://programmer.97things.oreilly.com/wiki/index.php/Don't_Repeat_Yourself)
* because WordPress is a great hammer but not every website is a nail

---

1. [Quick Start](#QuickStart)
2. [More Details](#MoreDetails)
3. [Install](#Install)

# <a name="QuickStart"></a>Quick Start
_index.html:_

    <html>
      <body>
        <!-- USING CmsLess: -->
        <h1>Site Header</h1>
        <p>And other content that belongs on every page</p>
        <div id="cms-less-destination">
        	<!-- will be populated from cms-less-content/pagename.html -->
        	<!-- 	where the pagename is the hash up to the last hyphen -->
        	<!-- 	and if there is no hash, pagename is 'index' -->
        	<!-- 	and if the page can't be loaded, the pagename is '404' -->
       </div>
       
       <!-- SETUP for CmsLess: -->
       <script src="https://ajax.googleapis.com/ajax/libs/jquery/2.1.4/jquery.min.js"></script>
       <script src="js/lib/cms-less/cms-less.js"></script>
       <script type="text/javascript">
         $(function() {
           CmsLess.Init({
             configurationOptions: 'see documentation'
           });
         });
       </script>
     </body>
   	</html>
   	
* there are two `index.html` files: 
	* `index.html` contains all the content common to all pages (header, nav, library includes)
	* `cms-less-content/index.html` contains the content to be shown for both example.com/#index and example.com/
* individual pages:
	* accessed from: `example.com/#pagename`
	* content stored in: `cms-less-content/pagename.html`
* Note: this repo is set up to be added to an existing project by cloning into `js/lib`. If you're making a new site from scratch and want boilerplate with CmsLess, See [the CmsLess boilerplate site](https://github.com/alexanderbird/cms-less-boilerplate) which can be forked as the initial commit of your new site. 
   	
# <a name="MoreDetails"></a>More Details
## What Goes Where
* `index.html`
	* loads jQuery and CmsLess
	* initializes and (optionally) configures CmsLess
	* all the **content that must appear on every page**
* `cms-less-content/` (or whatever you've chosen as your [content path](#Configuration))
	* one html file for each page
	* the contents of that file will be [load](http://api.jquery.com/load/)ed into the `#cms-less-destination` element
* `cms-less-content/foobar.html`
	* **will be shown for example.com/#foobar**
* `cms-less-content/404.html `
	* will be shown when a page cannot be found (example.com/#missing)
		* note that if [url rewriting](#UrlRewriting) has been enabled, then example.com/missing will become example.com/#missing, and so the same 404 page will be shown
		* without url rewriting enabled, example.com/missing will show the web server's 404 page

## <a name="Anchors"></a>About Anchors
**Yes**, this works with page anchors. 

Anything after the `anchorDelimiter` (default: `-`) in the url hash is ignored when loading the page content. This way, you can still use the hash for page anchors. The only nuisance is that the anchor must now be `<a href='#pagename-anchorname'>Anchor Name</a>` to link to `<div id='pagename-anchorname'>Anchored Stuff</div>`. 

To reiterate, all of the following will load `cms-less-content/foo.html`:

* `example.com/#foo`
* `example.com/#foo-`
* `example.com/#foo-bar`
* `example.com/#foo-baz`

You can test this out by [installing CmsLess](#Install) on your web site and browsing to the index page. From your web browser's JavaScript console, run `CmsLess.PageName('#somepage-totest')` to see what page name CmsLess extracts from the hash. If you don't provide an argument, it will take the hash from the `window.location.hash` property. 

## Updating UI on page change
CmsLess dispatches the `cms-less-page-change` event when the new page has loaded. 

    $(document).bind('cms-less-page-change', function(e) {
        console.log("The current page is " + e.originalEvent.detail.pageName);
    });
    
 Note: currently this event is not dispatched for the first page load if there is no url hash. It's a bug, and I really can't figure out what's causing it. I welcome pull requests or feedback :)
    
## Avoiding the footer [FOUC](https://en.wikipedia.org/wiki/Flash_of_unstyled_content) (Flash Of Unstyled Content)
Or rather, "Flash of Empty Page Before Content Loads" - if you have any footer or anything, you'll see it at the top of the screen before the page content pushes it down to the bottom. It's jarring. Avoid this by adding: 

To /index.html

    <span id='cms-less-content-placeholder'/>
 
To stylesheet

    #cms-less-content-placeholder {
      display: block;
      height: 100vh;
    }
    
Before the first page is loaded the placeholder pushes the footer off the screen. When the first page is loaded, it replaces the placeholder span. 

## Without a web server
**Sort-of**. [Some browsers protect from cross origin requests](http://stackoverflow.com/questions/20041656/xmlhttprequest-cannot-load-file-cross-origin-requests-are-only-supported-for-ht) when you're viewing the file direct from your local machine. This means that when CmsLess tries to load page content, the browser blocks the request (there's an error message shown in the developer console, and nothing shown on the web page).

### Your options:
Use Firefox, run a local web server, or muck about with the [Chrome workaround](http://stackoverflow.com/questions/18586921/how-to-launch-html-using-chrome-at-allow-file-access-from-files-mode) to get it working. 

If you're trying to get Chrome to work, you should note that me and [this guy on Stack Overflow](http://stackoverflow.com/questions/20041656/xmlhttprequest-cannot-load-file-cross-origin-requests-are-only-supported-for-ht) weren't able to get it to work

I prefer the local server option, and I like to know that if I have to check something quickly without starting up the server, I can always use Firefox. 

---

# <a name="Install"></a>Install
## 1. Clone into your project
    cd js/lib
    git submodule add https://github.com/alexanderbird/cms-less.git
    mkdir cms-less-content
    cp js/lib/cms-less/404.html cms-less-content
    
## 2. Add to index.html

    <div id="cms-less-destination"></div>
    <script src="js/lib/cms-less/cms-less.js"></script>
    <script type="text/javascript">
      $(function() {
        CmsLess.Init();
      });
    </script>

* make sure you use the correct `src`path to cms-less.js in place of `js/lib/cms-less/cms-less.js`
* if you don't want to use `cms-less-destination`, set a different `destinationSelector` in the Init() configuration - see Configuration notes

## 3. Dependancies (JQuery)
CmsLess relies on JQuery 2.x. 

    <script src="https://ajax.googleapis.com/ajax/libs/jquery/2.1.4/jquery.min.js"></script>
    

## <a name="Configuration"></a>Configuration
The following options can be configured by passing an associative array to the Init method, as shown above.

| Key | Default | Description |
|-----|---------|-------------|
|`contentPath`|`cms-less-content`|Directory containing the page content html files|
|`destinationSelector`|`#cms-less-destination`| Css selector for the element that will have its content will be set by CssLess |
|`anchorDelimiter`|`-`|Delimiter between page name and anchor tag. See [Anchors](#Anchors)|
|`notFoundPageName`|`404`|Page to be loaded if a page is not found. See [Page Not Found](#PageNotFound).|


# Other Install Details

## <a name="PageNotFound"></a>Page Not Found
If the content load fails, then the notFoundPageName content is loaded. (See also: [Usage](#Usage).) A sample 404 page is provided with cms-less: to use it, copy it to your contentPath folder. 

## <a name="UrlRewriting"></a>Redirecting `/path` to `/#path`
### Advantages
* more robust - someone who types in yourdomain.com/path is automatically redirected to yourdomain.com#path
* you can use the built-in CmsLess 404 page for `/invalid-path`

### Setup with apache
Add the following to your .htaccess file: 

    RewriteEngine on

    # Ignore everything in sub-directories - necessary so assets, etc. can still be loaded
    RewriteCond %{REQUEST_URI}    ^/([^/]+)/.*$  [NC]
    RewriteRule .*                -              [L]

    # redirect /path to /#path
    RewriteCond %{REQUEST_URI}    ^/([^/]+)      [NC]
    RewriteRule (.*)              /#%1           [R,NE,L]
    
Note: with this configured, CmsLess does't serve anything from the root directory other than index.html

    
