# BEACSS = Block, Element, Attribute CSS


<img src="https://cloud.githubusercontent.com/assets/1434956/16282094/155700ba-387c-11e6-957f-fa5dfe0759f6.gif" />

Before we begin let's talk basics...

# **General Sass Best Practices**

1. **Keep an eye on the CSS**

    * Be aware of what your Sass is creating. Check on the CSS from time to time to ensure there aren’t nested classes (See Rule #1) or any wacky code coming from mixins.

2. **Stay DRY [Don't Repeat Yourself]**

    * Whenever possible, use variables, mixins, and placeholders to avoid repetitious code.

3. **Use semantic placeholders**

    * The idea here is to retain control of your style system by creating a styleguide of possible text, media, and layout type styles, and then extending those styles within various components
    * Use names like %section-headline, %text-input, %strong-paragraph, %default-ol, etc.

4. **Namespacing**

    * If your project uses multiple repositories or frameworks, use class prefixes to avoid collisions.


# **Rule #0 - Never style IDs or HTML elements***

It is advisable to avoid styling IDs (to avoid specificity battles) or HTML elements. If styles are always applied using classes (and modified with attributes), then you can adapt your markup for SEO or accessibility purposes as needed. For example, if we had a headline `<h2 class="ux-section-headline”>` that we wanted to change to an H1 for SEO reasons, we can easily change it without any impact to the styles.

The exception to this rule is when you must relinquish control of the HTML to the content editor in a CMS, usually within a WYSIWYG. In our system, we have one "generic" component for this use case, in which we apply styles to each tag that could appear within a WYSIWYG. This generic component should **never** be nested inside of other components.



	.ux-default--component {
	    p {
	        @extend %default-paragraph;
	    }
	    ul {
	        @extend %default-ul;
	    }
	    ol {
	        @extend %default-ol;
	    }
	}


* * *


# **Rule #1** - Every element should have a single source of truth from which it derives all of its styles.

## The scenario:  I want to use an existing component, but I want to modify one style (i.e. the font size) on a element within that component.

### Existing component:

	// library/component.html:
	
	<div class="ux-component">  <!-- Block -->
	  <div class="ux-component-headline">   <!-- Element -->
	    text
	  </div>
	</div>

	// library/component.scss:

	.ux-component {
	  background: gray;  
	  &-headline {
	    color: red;
	    font-size: 18px;
	  }
	}

### If the styles are reusable across other pages, and the headline is the only thing I need to change, can I add a data-attribute to the headline element? Yes!

	// custom-page.html:

	<div class="ux-component">
	  <div class="ux-component-headline" data-ux-size="xl”>
	    text
	  </div>
	</div>

	// component.scss:

	.ux-component {
	  &-headline{
	    color: red;
	    font-size: 44px
	    &[data-ux-size="xl”]  {
	      font-size: 48px;
	    }
	  }
	}

#### Why?

The ux-component-headline styles all live in the component sass file together, so if changes to the element need to be made, you can see what all the various options are and accommodate accordingly. The data attribute may be applied to HTML as needed. Note that modifiers can use more generic attribute names because the class and attribute must be on the same element.

### If the styles are reusable across other pages, and the context should affect multiple elements within the block, can I add a data-attribute to the block? Yes!

	// custom-page.html:

	<div class="ux-component" data-ux-component-size="xl”>
	  <div class="ux-component-headline">
		   text
     </div>
	  <div class="ux-component-subhead">
        text
     </div>
	</div>

	// component.scss:

    .ux-component {
	   &-headline {
	      color: red;  
	      font-size: 18px;
	      [data-ux-component-size="xl”] & {
	        font-size: 24px;
	      }
	   }
	   &-summary {
	    font-size: 14px;
	    [data-ux-component-size="xl”] & {  //context
	      font-size: 20px;
	    }
	    [data-ux-component-size="xs”] & {  // another context
	      font-size: 11px;
	    }
	  }

#### Why?

The ux-component-headline styles still all live in the component sass file together. The data attribute may be applied to the HTML as needed. Notice that because this data attribute lives on the component (container) and not the element, the name of the data-attribute is scoped with the name of the component to make sure that this same attribute doesn’t exist in a parent block.  Even though the HTML has two piece of information that control this style (the attribute and the class), all the styles live in one place (in the component.scss), so informed updates can be made to the original styles and the various contexts, without fear of breaking something.

### If the headline is the only thing I need to change, and I only want this style to affect my custom page (not-reusable), can I create a new class? Yes!

	// custom-page.html:

	<div class="ux-component">
	  <div class="ux-component-headline-unicorn">…</div>
	</div>

	// custom-page.scss:

	//@todo -- add this style to .ux-component-headline
	.ux-component-headline-unicorn {
	  font-size: 24px;
	  color: red;
	}

#### Why?

The new ux-component-headline-unicorn has 1 source from which it gets its styles, just custom-page.scss. Notice that we have added the color: red to the ux-component-headline-unicorn to complete the styles. You’ll need to copy over any existing styles you want to keep, and change what you want using only that custom class.

### Should I add additional styles to the existing element class? Nope!

	// custom-page.html:

	<div class="ux-component">
	  <div class="ux-component-headline">…</div>
	</div>

	// custom-page.scss:

	.ux-component-headline {
	  font-size: 24px;
	}

#### Why not?

The headline element has 2 sources from which it gets its styles, both the library and custom-page.scss. If the ux-component-headline was modified in the component.scss, the custom page would either not be appropriately affected, or would be adversely affected.

### Should I create a new class, and add both classes to the element? Nope!

	// custom-page.html:

	<div class="ux-component">
	  <div class="ux-component-headline ux-component-headline-unicorn">…</div>
	</div>

	// custom-page.scss:

	.ux-component-headline-unicorn {
	  font-size: 24px;
	}

#### Why not?

The headline element has 2 sources from which it gets its styles, both the library and custom-page.scss. If the ux-component-headline was modified in the component.scss, the custom page would either not be appropriately affected, or would be adversely affected.

### Should I create a data-attribute for the element in the custom-page.scss, and add it to the element? Nope!

	// Custom-page.html:

	<!-- Block -->
	
	<div class="ux-component"> 
	  <!-- Element with attribute -->  
	  <div class="ux-component-headline" data-ux-headline="unicorn” >
	   …
	  </div>
	</div>

	// custom-page.scss:

	.ux-component-headline[data-ux-headline="unicorn”] {
	  font-size: 24px;
	}

#### Why not?

The headline element has 2 sources from which it gets its styles, both the library and custom-page.scss. If the ux-component-headline was modified in the component.scss, the custom page would either not be appropriately affected, or would be adversely affected.

Exceptions to this might include background images only, for cards and/or bands. 

### Should I create a data-attribute for the block in the custom-page.scss, and use it as context for the element? Nope!

	// Custom-page.html:

	<!-- Block with attribute -->
	<div class="ux-component" data-ux-size="unicorn”> 
	
	<!-- Element -->
	  <div class="ux-component-headline">…</div>
	</div>

	// custom-page.scss:

	[data-ux-size="unicorn”] .ux-component-headline {
	  font-size: 24px;
	}

#### Why not?

The headline element has 2 sources from which it gets its styles, both the library and custom-page.scss. If the ux-component-headline was modified in the component.scss, the custom page would either not be appropriately affected, or would be adversely affected.

* * *


# Rule #2 - A selector should only be used to apply styles to elements within that same block

(Meaning the child items should match the namespace of the parent)

### Should I create a generic headline placeholder that can be placed into any component like this? Yes!

#### Components:

	// library/component-A.html:

	<div class="ux-component-A"> <!-- Block -->
	  <div class="ux-component-A-headline"> <!-- Element -->
	   …
	  </div>
	</div>

	// library/component-B.html:
	
	<div class="ux-component-B"> <!-- Block -->
	  <div class="ux-component-B-headline"> <!-- Element -->
	   …
	  </div>
	</div>

#### Styles:

	// library/extends/headlines.scss:

	%fancy-headline {
	  font-size: 24px;
	  line-height: 1.5;
	  color: gray;  
	}
	
	%snazzy-headline {
	  font-size: 22px;
	  line-height: 2;
	  color: gray;  
	}

	// library/component-A.scss:

	.ux-component-A {
	  &-headline {
	    @extend %fancy-headline;
	  }
	}

	// library/component-B.scss:

	.ux-component-B {
	  &-headline {
	    @extend %snazzy-headline;
	  }
	}

### Should I create a generic headline class that can be placed on any component like this? Nope!

#### Components:

	// library/component-A.html:

	<div class="ux-component-A"> <!-- Block -->
	  <div class="ux-headline"> <!-- Element -->
	   …
	  </div>
	</div>

	// library/component-B.html:

	<div class="ux-component-B"> <!-- Block -->
	  <div class="ux-headline"> <!-- Element -->
	   …
	  </div>
	</div>

#### Styles:

	// library/headlines.scss:

	.ux-headline {
	  font-size: 24px;
	  color: gray;  
	}

#### *Why not?*

This breaks our Single Responsibility Principle in that these styles now apply to multiple elements in different blocks. If you are then asked to make the headline in component-B all caps, while keeping component-A with the original styles, you’d have to do the following:

	// library/component-B.scss:
	
	.component-B .ux-headline {
	  text-transform: uppercase;
	}

And this breaks rule #1: all elements must have a single source of truth.


* * *


# Rule #3 - Write CSS selectors by referencing parent selectors whenever possible

### Existing component:

	// library/component.html:

	<div class="ux-component"> <!-- Block -->
	  <div class="ux-component-headline"> <!-- Element -->
	   …
	  </div>
	</div>

### Should I nest the full element class inside the block class like this? Nope!

	// library/component.scss:

	.ux-component {
	  background: gray;  
	  .ux-component-headline {
	    color: red;
	    font-size: 18px;
	  }
	}

#### *Why not?*

If you needed to change the component name, you’d have to do it in multiple places, as well as having to type the name of the component over and over. This violates the principle of DRY (don’t repeat yourself) code. Additionally, this will generate nested selectors, which violates rule #1.

### Should I list the full element class separately like this? Nope!

	// library/component.scss:

	.ux-component {
	  background: gray;  
	}
	
	.ux-component-headline {
	    color: red;
	    font-size: 18px;
	  }
	}

#### *Why not?*

If you needed to change the component name, you’d have to do it in multiple places, as well as having to type the name of the component over and over. This violates the principle of DRY (don’t repeat yourself) code.

### Should I nest the element inside the block, and extend the name using `&-` like this? Yes!

	// library/component.scss:

	.ux-component {
	  background: gray;  
	  &-headline {
	    color: red;
	    font-size: 18px;
	  }
	}

#### *Why?*

There is only one selector for the headline, as opposed to nested selectors. And the component name may be easily changed and all the elements it contains will be updated accordingly

* * *


# Rule #4 - Use extends OR write custom style properties; not both

## The what-if scenario: I want to extend a placeholder because I’m using all the same styles, but I need to override one thing!*

### Existing Styles:

	// library/extends/headlines.scss:

	%fancy-headline {
	  font-size: 24px;
	  line-height: 1.5;
	  color: gray;  
	}

### Should I create a custom selector and add all the styles I need like this? Yes!

	// library/custom-page.scss:
	
	.ux-unicorn-headline {
	    font-size: 24px;
	    line-height: 1.5;
	    color: pink;
	}

#### *Why?*

Rule #1 - single source of truth.

### Should I create a new placeholder, copy styles if indeed they exist elsewhere and add custom styles, then extend that placeholder like this? Yes!

	// library/extends/headlines.scss OR library/custom-page.scss:

	%snazzy-headline {
	    font-size: 24px;
	    line-height: 1.5;
	    color: pink;
	}

	// library/custom-page.scss:

	.ux-unicorn-headline {
	    @extend %snazzy-headline;
	}
	
	.ux-abominable-snowman-title {
	    @extend %snazzy-headline;
	}

#### *Why?*

If this new set of styles is reusable, create a new placeholder, and extend that placeholder. Rule #1 - single source of truth. Now this new placeholder can be updated without fear of breaking something.

### Should I create a new *placeholder* that extends another placeholder and add custom styles like this? Yes! 

Then your class selector can extend the new placeholder.

	// library/extends/headlines.scss:
	
	%lovely-headline {
	  font-size: 24px;
	  line-height: 1.5;
	}
	
	%fancy-headline {
	  @extend lovely-headline;
	  color: gray;  
	}
	
	%snazzy-headline {
	    @extend lovely-headline;
	    color: pink;
	}

	// library/custom-page.scss:

	.ux-unicorn-headline {
	    @extend %fancy-headline;
	}

	.ux-abominable-snowman-title {
	    @extend %snazzy-headline;
	}

#### *Why?*

If this new set of styles is reusable across multiple components, create a new placeholder, and extend that placeholder. Rule #1 - single source of truth. Now this new placeholder can be updated without fear of breaking something. The placeholders all live in the same place, so we can control the inheritance with expected results.

### Should I create a custom selector, and extend a placeholder style and then add custom styles beneath it like this? Nope!

	// library/custom-page.scss:

	.ux-unicorn-headline {
	    @extend %fancy-headline;
	    color: pink;
	  }
	}

#### *Why not?*

Not only does this print a color property twice, when only one is needed, it also violates Rule #1 - single source of truth. It would become very difficult to write theme-based color rules (to accommodate for different background colors) for this headline because overrides live in other files.

