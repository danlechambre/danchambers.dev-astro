---
layout: ../../layouts/ArticleLayout.astro
title: "Interactive Non-interactive Elements"
pubDate: 2023-07-01
description: "It's easy to forget the basics"
author: "Dan"
tags: ["a11y"]
---

In my current day job I do a lot of Single Page Apps for SMEs wanting to get to production as _fast_ as possible. This usually means selecting zero-config build tools like Create React App coupled with massive component libraries like Kendo and MUI. It tends to also mean that things like testing and accessibility are simply viewed as "nice-to-haves", and put on the "back burner" 😒. At the end of the day the client gets what the client wants. Fair is fair.

Consequently, it's very rare that I actually build an app using just HTML, CSS, and JavaScript (even though I'm certain this would end up being quicker for certain projects 😂). Nor do I generally get to set up and configure my toolchain from scratch. But worst of all, I rarely have time to consider what code actually ends up getting run in the browser.

There's probably an argument to say this is ok. Less time thinking about these things means more time building the app I guess. And as much as I'd love to hand-craft every app I work on like a DongYang wood carving, its just not viable for most clients.

So, recently, I've been using my personal projects as an opportunity to rediscover the core web technologies and gain a deeper understanding of what all my modern tooling is _actually_ doing for me.

Yesterday, on a new React project, I set up ESLint from scratch including the jsx-a11y plugin. Nothing notable in and of itself. It wasn't until I started to build a simple image gallery component I realised I've forgotten what good HTML should look like 🤦:

```jsx
return (
  <div>
    <img src={images[active]} alt="the thing" />
    <div>
      {images.map((photo, i) => (
        <img
          onClick={handleClick}
          data-index={i}
          key={photo}
          src={photo}
          className={i === active ? "active" : ""}
          alt="thing thumbnail"
        />
      ))}
    </div>
  </div>
);
```

ESLint was quick to yell at me with the following:

<p class="code-title">console-error</p>

```plaintext
Visible, non-interactive elements with click handlers must have at least one keyboard listener.
eslintjsx-a11y/click-events-have-key-events

Non-interactive elements should not be assigned mouse or keyboard event listeners.
eslintjsx-a11y/no-noninteractive-element-interactions
```

As someone who once built a WCAG 2.1 (AA) compliant front-end, I was kinda shocked at what I'd written. Amazingly, the most common tips I found were to add `// eslint-disable-next-line` or set `role="presentation"` on the tag... er, no.

Visiting the ESLint [jsx-a11y plugin docs](https://github.com/jsx-eslint/eslint-plugin-jsx-a11y/blob/main/docs/rules/no-noninteractive-element-to-interactive-role.md) offered some succinct guidance on what I was doing wrong:

> Non-interactive HTML elements indicate content and containers in the user interface. Non-interactive elements include `<main>`, `<area>`, `<h1>` (,`<h2>`, etc), `<img>`, `<li>`, `<ul>` and `<ol>`.
>
> Interactive HTML elements indicate controls in the user interface. Interactive elements include `<a href>`, `<button>`, `<input>`, `<select>`, `<textarea>`.
>
> WAI-ARIA roles should not be used to convert a non-interactive element to an interactive element. Interactive ARIA roles include button, link, checkbox, menuitem, menuitemcheckbox, menuitemradio, option, radio, searchbox, switch and textbox.

I also had a quick dig around on the very dry, but ever informative, [W3C-WAI pages](https://www.w3.org/WAI/design-develop/) to refresh myself on how I should have approached this.

This is what I ended up with:

```jsx
return (
  <section aria-labelledby="gallery-heading">
    <h3 id="gallery-heading" className="visually-hidden">
      Photos of the thing
    </h3>
    <img src={images[active].url} alt={images[active].alt} />
    <ul>
      {images.map((img, i) => (
        <li key={img.id}>
          <button onClick={handleInteraction} onKeyDown={handleInteraction}>
            <img
              data-index={i}
              src={img.url}
              className={i === active ? "active" : ""}
              alt={img.alt}
            />
            <span className="visually-hidden">Select image</span>
          </button>
        </li>
      ))}
    </ul>
  </section>
);
```

Semantically, to me at least, this makes a ton of sense. What are image gallery thumbnails after all, if not an unordered list of buttons, each having the implicit action of "Select image"?

I did scratch my head a bit on the alt text for the thumbnail. At first I changed to an empty alt attribute as you would for an icon button. My thinking was that the alt attribute was redundant with the `visually-hidden` button text, but thinking on, I realised that the image on a thumbnail is definitely not 'presentation-only'.

Without realising, we select certain thumbnails because we are visually drawn to them in some way. To enable a comparable experience for all users, I decided to use the primary alt text on the thumbnail too e.g. _"Large brown labrador jumping over a water sprinkler in a suburban garden"_.

It's easy to imagine a user with a visual-impairment that is not able to easily decipher small images (e.g. thumbnails), but _is_ able to view them well at full size. In that case, the alt text on the thumbnail becomes more important than that of the main image (we should never make assumptions as to why someone may be using a screen reader).

Whether this is the best approach is not really the point. The important takeaway is that when we're working with _so_ many layers of abstraction, in hugely complex applications, it's easy to forget what properly structured, semantic, _accessible_ HTML looks like.

Good reads on the topic:

- [Accessible Icon Buttons](https://www.sarasoueidan.com/blog/accessible-icon-buttons/), _by Sara Soueidan_
- [WebAIM - Screen Reader Survey](https://webaim.org/projects/screenreadersurvey9)
- [W3C WAI - Developing for Web Accessibility](https://www.w3.org/WAI/tips/developing/)
