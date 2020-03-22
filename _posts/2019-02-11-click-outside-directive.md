---
layout: post
title: "Building a ClickOutside directive with Angular"
image: "/images/click-outside/click-outside.jpg"
published: true
---

I often find myself needing to write code to respond to the user having clicked outside a particular Angular Component or DOM element - this is usually when implementing things like modals and menus, where there is a need to remove the component from the DOM if the user clicks anywhere outside of it. 

Today, I am going to explain how we can solve this problem with a simple directive.

## Click Outside directive

We can implement a `ClickOutsideDirective` with just a handful of lines of code:

```ts
import { Directive, Input, Output, EventEmitter, ElementRef, HostListener } from '@angular/core';

@Directive({
  selector: '[clickOutside]',
})
export class ClickOutsideDirective {

  @Output() clickOutside = new EventEmitter<void>();

  constructor(private elementRef: ElementRef) { }

  @HostListener('document:click', ['$event.target'])
  public onClick(target) {
    const clickedInside = this.elementRef.nativeElement.contains(target);
    if (!clickedInside) {
      this.clickOutside.emit();
    }
  }
}
```

In the above, we simply listen for the `document:click` event, which occurs any time the user clicks anywhere in the DOM. We then check if the click was inside of the element which the directive is attached to, using the DOM API `contains` method. If the click was not inside of this element, then we emit.

The directive can then be used in a template as follows:

```html
<div (clickOutside)="someHandler()"></div>
```

Here is a [working StackBlitz demo of this directive](https://stackblitz.com/edit/angular-j7jqq2).

# Helpful resources

I highly recommend the following Udemy course if you are interested in improving your knowledge of Angular (affiliate link):

<figure>
  <a href="https://click.linksynergy.com/link?id=5FPU6FRy5w4&offerid=507388.756150&type=2&murl=https%3A%2F%2Fwww.udemy.com%2Fcourse%2Fthe-complete-guide-to-angular-2%2F">
      <img src="https://i.udemycdn.com/course/480x270/756150_c033_2.jpg"/>
      <figcaption>Angular - The Complete Guide (2020 Edition)</figcaption>
  </a>
</figure>