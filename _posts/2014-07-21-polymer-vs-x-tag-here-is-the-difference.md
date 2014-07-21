---
layout: post
title: Polymer vs. X-Tag - Here's the difference
---

Let's face it. Web Components is the next big thing. Sure, technologies have to be adapted and implemented by browsers first, but [we are on the right track](http://jonrimmer.github.io/are-we-componentized-yet/). Turns out that Chrome even comes with full Web Components support in [version 36](https://twitter.com/addyosmani/status/489490560869490688), which is super awesome.

However, some browsers take longer to implement these technologies, for others it's not even sure yet if they'll implement them at all. To make Web Components work in all major browsers, we need polyfills to close the gaps. Luckily, the folks over at [Polymer](http://polymer-project.org) have developed such a set of polyfills to make Web Components work in the browsers **today**.

These polyfills are bundled as [platform.js](https://github.com/Polymer/platform) and you can use them right away to play with Web Components APIs. Taking a deeper look, we can see that platform.js consists of the following polyfills (and prolyfills):

- [CustomElements](https://github.com/Polymer/CustomElements)
- [HTMLImports](https://github.com/Polymer/HTMLImports)
- [ShadowDOM](https://github.com/Polymer/ShadowDOM)
- [NodeBind](https://github.com/Polymer/NodeBind)
- [TemplateBinding](https://github.com/Polymer/TemplateBinding)
- [WeakMap](https://github.com/Polymer/WeakMap)
- [observe-js](https://github.com/Polymer/observe-js)

This is much more than the actual four specifications that form Web Components. The reason for that is that platform.js is used under Polymer's hood and Polymer adds some sugar on top of the actual Web Components APIs that require things like *TemplateBinding*.

However, that doesn't mean you always have to include all of them if you want to build something with Web Components, because not all APIs are necessarily needed depending on your use case. [X-Tag](http://x-tags.org) for example, another project that tries to help us out making the development of web components easier, sits on top of platform technologies too, but it only uses *CustomElement* APIs when it comes to Web Components APIs.

There was an article recently popping up in the interwebs about [Polymer vs. Angular](http://www.binpress.com/blog/2014/06/26/polymer-vs-angular/) and the differences between those two projects. While it's fairly easy to understand the differences between those two, I think it's a bit harder to grasp the different directions and approaches of Polymer in comparison to X-Tag, because both seem to provide a solution for building custom web components.

This article explains the difference between Polymer and X-Tag. There are differences on several levels, I try to get into all of these but we start from a bird's-eye view.

## From a bird's-eye view

Taking a look at both websites, [Polymer's](http://polymer-project.org) and [X-Tag's](http://x-tags.org), it doesn't take long to realise that these projects follow different approaches, while both are built on top of the native technologies though. 

Whereas Polymer makes use of all four Web Components technologies, X-Tag only depends on the Custom Elements technology and gives the developer the right to opt in for ShadowDOM. And that's okay because that just underlines the different philosophies behind these two projects.

Polymer acts as a kind of thin sugar-layer on top of Web Components technologies, to unify the underlying APIs and try to give you a direction on how to use these independent APIs in combination. Polymer really gives you the feeling of packaging things together when creating custom components. Just take a look at the following simple example:

```html
<polymer-element name="foo-element" noscript>
  <template>
    <p>This is a foo-element.</p>
  </template>
</polymer-element>
```

We'll go into technical details in a minute, I just want to make clear that this code snippet already uses HTML Templates, ShadowDOM and Custom Elements technologies. The element is registered as Custom Element behind the scenes. Its template is defined using HTML Template and it gets later injected into the elements ShadowDOM once instantiated. Finally, it can be imported using HTML Imports like this:

```html
<link rel="import" href="path/to/foo-element.html">
```

I think you get the idea. All of these technologies are actually independent and Polymer pushes you into a direction on how to use these together, based on best practices. 

So what about X-Tag? X-Tag has a little different approach than Polymer. Here's what its website says:

> X-Tag is a small JavaScript library, created and supported by Mozilla, that brings Web Components Custom Element capabilities to all modern browsers.

This paragraph pretty much reflects what I've already mentioned. X-Tag only uses Custom Element APIs to build web components. Wait, it only uses one of four technologies? Can that even work? And why should one ever do this when there's more one can use?

Keep cool buddy. Let's get through this one by one. So yea, only using one of four technologies works because they are, as mentioned in the paragraphs above, independent. If you just want to use HTML Templates, that's fine, go for it. If you want to use ShadowDOM for some elements for whatever reason, okay, I'm with you. And if you just want to register some of your custom elements, all you need is Custom Elements APIs. And that's what X-Tag does.

So X-Tag's goal and philosophy is to just use Custom Elements APIs to build web components, because in most cases that's enough. There's no ShadowDOM nor HTML Template required. To have a fair comparison, let's take a look what the element above would look like when implemented with X-Tag:

```js
xtag.register('foo-element', {
  lifecycle: {
    created: function () {
      this.innerHTML = '<p>This is a foo-element.</p>';
    }
  }
});
```
This basically generates the same result, except that there's no ShadowDOM in that example yet. Let's take a deeper look at what's different when creating components with either one or the other library.

## Creating and registering custom elements

Alright, so as we know Polymer provides a declarative way of creating elements using `<polymer-element>` like so:

```html
<polymer-element name="my-element" noscript>
  <template>
    <p>Hello there</p>
  </template>
</polymer-element>
```
You might notice the `noscript` attribute here. This is a pretty smart feature. When creating elements that don't need any further JavaScript logic (lifecycle callbacks, mixins, etc.), we can apply the `noscript` attribute. Polymer is then smart enough to register your custom element implicitly by using the information it gets out of your declarative definition. All it needs to create custom element is a name, which is in our case provided as `name` attribute with the value `my-element`. 

What seems to happen magically here, is just the usage of `document.registerElement()` under the hood, which is the method defined by the Custom Elements specification to register custom elements. Besides an element name (that requires a `-` in it), it expects an object that has an `HTMLElement.prototype` that defines lifecycle callbacks for your custom element. To learn more about Custom Elements in general I recommend the [learn resource](http://www.polymer-project.org/platform/custom-elements.html) by the Polymer folks, or the tutorials on [html5rocks.com](http://www.html5rocks.com/en/tutorials/webcomponents/customelements/?redirect_from_locale=de) written by [Eric](http://twitter.com/ebidel).

It should also be mentioned that once there's a `<template>` defined, Polymer automatically creates a `shadowRoot` and injects its contents into it. In other words, ShadowDOM is used by default and there's no way to opt-out.

If there's logic required for our custom element (and that's mostly the case), Polymer provides an imperative way to register our custom element too. Using that our example element definition looks like this:

```html
<polymer-element name="my-element">
  <template>
    <p>Hello there</p>
  </template>
  <script>
    Polymer('my-element');
  </script>
</polymer>
```
Notice the removed `noscript` attribute. We now explicitly register our element using `Polymer()`. Now, if we want to add lifecycle callbacks, we can do that on the go:

```html
<polymer-element name="my-element">
  <template>
    <p>Hello there</p>
  </template>
  <script>
    Polymer('my-element', {
      created: function () {
        console.log('Hello there.');
      }
    });
  </script>
</polymer>
```
If you want to learn more about Polymer's lifecycle callbacks API, head over to their [docs](http://www.polymer-project.org/docs/polymer/polymer.html#lifecyclemethods).

You can create Polymer elements even entirely imperative with JavaScript if you want like so:

```js
Polymer('my-element');

var element = document.createElement('div');

element.innerHTML = '<polymer-element name="my-element">' +
  '<template>' +
  '<p>Hello there</p>' +
  '</template>' +
  '</polymer-element>';

document.body.appendChild(element);
```

Let's take a look what element creation and registration with X-Tag looks like compared to that:

```js
xtag.register('my-element', {
  lifecycle: {
    created: function () {
      this.innerHTML = '<p>Hello there</p>';
    }
  }
});
```

There's no way to define a custom element declaratively with X-Tag. Even if that feels not so nice compared to Polymer's declarative approach, one shouldn't forget that `<polymer-element>` probably wouldn't be there, if there wasn't an `<element>` element proposal back in the days. You can read about it [here](http://www.w3.org/TR/components-intro/#defining-a-custom-element) but the specification changed and `<element>` has been [removed](http://lists.w3.org/Archives/Public/public-webapps/2013JulSep/0287.html), while Polymer's `<polymer-element>` stays as it is as polyfill in case `<element>` comes back. Just like `Polymer()`, `xtag.register()` uses `document.createElement()` under the hood to register custom elements on the DOM. 

As you can see, X-Tag also doesn't have any default template mechanism. Whereas Polymer **always** looks for a `<template>` inside your element definition and automatically inserts its content into the elements ShadowDOM, X-Tag lets **you** decide if you need a template at all and if so, when and how it gets injected. 

So what looks like a drawback at a first glance, is actually the power of flexibility. You probably don't want to create ShadowDOMs for all your custom elements, because of performance reasons or just because it's not required for your use case. In the example above, we use the `created` lifecycle callback and good old `innerHTML` to assign a template.

However, if you **want** to use HTML Template with X-Tag, all you have to do is to define your template first:

```html
<template id="my-template">
  <p>Hello there</p>
</template>
```
And clone its content later into your elements light DOM:

```js
xtag.register('my-element', {
  lifecycle: {
    created: function () {
      var tpl = document.getElementById('my-template').content;
      this.appendChild(tpl.cloneNode(true));
    }
  }
});
```
In case you want to have encapsulated styles and use the power of ShadowDOM, X-Tag comes with a pretty handy shorthand property `shadow`, that does the job for you:

```js
xtag.register('my-element', {
  shadow: '<p>Hello There</p>'
});
```
Using this property, X-Tag creates a `shadowRoot` behind the scences, creates a document fragment from your applied template string and inserts it into the elements ShadowDOM. And yes, this snippet is even smaller than Polymer's declarative approach.

## Defining attributes

Defining attributes in Polymer is pretty straight forward. When declaring a custom element, all you have to do is to add an `attributes` attribute that defines the names of your public element attributes:

```html
<polymer-element name="my-element" attributes="foo bar">
  <template>
    <p>Hello there</p>
  </template>
  <script>
    Polymer('my-element');
  </script>
</polymer-element>
```

You can now fill these attributes with concrete values like this:

```html
<my-element foo="Hello" bar="there"></my-element>
```

Defining attributes imperatively is as easy as setting up an object literal on the `publish` property:

```html
<polymer-element name="my-element">
  <template>
    <p>Hello there</p>
  </template>
  <script>
    Polymer('my-element', {
      publish: {
        foo: '',
        bar: ''
      }
    });
  </script>
</polymer-element>
```
Basically the same as defining them declaratively just with the fact, that `foo` and `bar` have an empty string default value instead of `null`. You can give them real default values here if you want to.

Attributes in X-Tag are handled via **accessors**. Basically, accessors kind of act like interceptors. They give you the chance to modify an elements value before it's returned or set. You can read about them [here](http://x-tags.org/docs#custom-tag-registration-accessors). Defining an `attribute` object on an accessors makes them available as element attribute. Here's our example written with X-Tag:

```js
xtag.register('my-element', {
  shadow: '<p>Hello There</p>',
  accessors: {
    foo: {
      attribute: {
        name: 'foo'
      }
    },
    bar: {
      attribute: {
        name: 'bar'
      }
    }
  }
});
```
The declarative approach makes that particular case much more pleasant IMO.

## Extending elements

One of the biggest features of web components is the fact that you can extend existing elements. I've already written an article about [inheritance and composition with Polymer](/2014/07/14/inheritance-and-composition-with-polymer/) so I'll only cover X-Tag's APIs here.

Extending elements with X-Tag is very close to the actual custom elements API. Just define an `extends` property and give it the name of the HTML element you want to extend:

```js
xtag.register('my-element', {
  extends: 'div'
});
```

If you want to use composition over inheritance (which is good), or you realise that it's not possible to extend multiple elements with the given API, you have to stick with creating elements, that use other elements in its templates. Just keep in mind, if you want to use custom elements in your template, make sure to use the `innerHTML()` setter, otherwise your custom element is not available.

```js
xtag.register('icon-label', {
  lifecycle: {
    created: {
      xtag.innerHTML(this, '<x-icon></x-icon><x-label>Label</x-label>');
    }
  }
});
```

## Conclusion

Both libraries are **great**!

While Polymer seems to be the perfect answer to how to use Web Components APIs in combination, X-Tag breaks its code base down to what is really needed in all cases (which is Custom Elements) and lets you opt-in for additional technologies like ShadowDOM or HTML Templates if needed. That makes the code smaller and faster. Also, reducing dependencies to only one technology, makes the code usable in more browser environments.

However, Polymer comes with a great set of polyfills (which is also used by X-Tag) to make all Web Components APIs work in all evergreen browsers today. Depending on your use case, you have to decide whether you always need a ShadowDOM for your custom elements and prefer a more isolated, packaged API that's based on best practices, or if you want to be more flexible and just register some custom elements that work with light DOM or use ShadowDOM and HTML Templates only if needed.