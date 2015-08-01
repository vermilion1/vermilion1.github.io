---
layout:     post
date:       2015-08-01 16:22:00
title:      "React: data sharing between server and client"
comments:   true
categories: react
---

Let's talk about sharing data in your Isomorphic (or Universal) JavaScript application.  
You must know what does this term mean, so two lines below are just for others: 
  
    Isomorphic application can be run on both the server and in the browser.
    Good for page load speed. Good for SEO.
    
Assume that we have to render application with a list of fresh news on page load and use 
pagination to dynamically load older. The server side implementation is pretty trivial.


{% highlight javascript %}
// server.js
// ---------

// Read news from database
let news = [{/**/}, {/**/}, {/**/}];

// Pass them to `Application` component
let markup = React.renderToString(React.createFactory(ApplicationComponent)({
  news: news
}));

// Return html to the browser
response.send('<!doctype html>' + markup);
{% endhighlight %}


{% highlight javascript %}
// ApplicationComponent.jsx
// ------------------------

React.createClass({
  render() {
    return (
      <html>
        <body>
          <RouteHandler {...this.props} />
        </body>
      </html>
    );
  }
});
{% endhighlight %}

Where `RouteHandler` is your router or some other component that owns
some `News` component that will take care about news and render them.

Let's modify the code to make these news available on the client side.

{% highlight javascript %}
// ApplicationComponent.jsx
// ------------------------

React.createClass({
  // Above we injected news property to the application component during the server side rendering.
  // To have it on the client side we will have to parse our secret DOM node
  getDefaultProps() {
    if (isBrowser()) {
      return JSON.parse(document.getElementById(PROPS_STORE_ID).innerHTML);
    } else {
      return {};
    }
  },
  
  // Here propStore is an element that contains serialized application props 
  getInitialState() {
    let data = {__html: JSON.stringify(this.props)};
    let propStore = <script type="application/json"
                            id={PROPS_STORE_ID}
                            dangerouslySetInnerHTML={data}></script>;
    return {
      propStore: propStore
    };
  },
  
  render() {
    return (
      <html>
        <body>
          <RouteHandler {...this.props} />
          /* Embed script element with our serialized data */ 
          {this.state.propStore}
        </body>
      </html>
    );
  }
});
{% endhighlight %}

Where:

 - `PROPS_STORE_ID` some unique id
 - `isBrowser` a function that does environment check, for example:
{% highlight javascript %}
function isBrowser() {
  return typeof window !== 'undefined';
}
{% endhighlight %}

So what we just did:

 - rendered application component on the server side with `props`
 - serialized these `props` and embedded them to the DOM element
 - deserialized element's data on the client side and saved result as default `props` value
 
Now both server and client own the same data and have the same checksum.
